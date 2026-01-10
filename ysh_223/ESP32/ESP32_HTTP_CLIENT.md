> ESP32-CAM 보드가 서버나 다른 기기를 위해 HTTP 통신같은 방법을 사용한다 그러므로 HTTP 통신 설정을 해줘야한다

<aside>

구조체 값을 넘길 때 `esp_err_t err =` 에 값을 받아 ESP_OK인지 확인하며 로그를 찍어야 한다

---

1. 서버 통신 매크로 설정
    
    ⇒ Server IP, Port, route 매크로 설정
    
    ⇒ 127.0.0.1, 5000, /upload/
    
2. 멀티파트 구성
    
    ⇒ 구분자 문자열, header, body 를 지정한다
    
3. 콘텐츠 총 크기 구하기 → 총 크기를 구하고 설정 구조체 제작
    
4. esp_http_client_handler 제작
    
5. 헤더 제작 → 서버에게 이 데이터는 어떻게 해석하는지 알려줌
    
6. 실패 상황 대비 함수 만들기 → goto문을 활용
    
7. 본문 스트리밍
    
    ⇒ Write라는 단어를 사용하며, 지금까지 설정한 len들을 사용함
    
8. 사진 전송
    
9. 응답 상태, 헤더, 바디 읽기
    
10. 종료 </aside>
    

### 1

---

⇒ 구동 가능한 서버에서 올바른 IP, Port를 가지고 온다

⇒ 서버에서로 라우팅을 처리해야한다

### 2

---

⇒ 사진의 용량이 커질 수 있기에 나눠서 보내야한다 (멀티파트 방식)

⇒ `const char *boundary = “——ESP32CamBoundary”` 를 기준으로 파트를 구분한다

<aside>

1. header
    
    ```c
    char head[256];
    int head_len = snprintf(head, sizeof(head),
                            "--%s\\r\\n"
                            "Content-Disposition: form-data; name=\\"image\\"; filename=\\"esp32-cam.jpg\\"\\r\\n"
                            "Content-Type: image/jpeg\\r\\n\\r\\n",
                            boundary); 
    ```
    
    ⇒ head 버퍼에 sizeof(head) 만큼의 문자열을 넣는다
    
    ⇒ 문자열 서식은 3번째 인자를 따른다
    
2. tail
    
    ```c
    char tail[64];
    int tail_len = snprintf(tail, sizeof(tail), "\\r\\n--%s--\\r\\n", boundary);
    ```
    
    ⇒ tail도 header와 같은 구조이다, 하지만 더 전송항 데이터가 없기에 header보다 짧다
    
3. 상태 점검
    
    ⇒ snprintf는 사용한 버퍼의 len을 반환한다
    
    ⇒ 두 버퍼의 len이 0과 같거나 0보다 작다면 실패이다
    

</aside>

### 3

---

⇒ `size_t len = (size_t)head_len + pic→len + (size_t)tail_len`

⇒ 각각 멀티파트 서식 header 크기, 사진 크기, 멀티파트 종료 tail 크기로 이들을 모두 더하면 총 크기이다

```c
esp_http_client_config_t cfg = {
    .url = url,
    .method = HTTP_METHOD_POST, // 전송 방식 (method에 POST)
    .timeout_ms = 10000,        // waiting 시간
};
```

⇒ esp_http_client_config_t 자료형으로 설정 구조체를 제작한다

⇒ 서버 URL을 지정하고, 서버가 기다리는 waitting시간을 지정한다

⇒ HTTP 전송 방식은 GET과 POST 중에 POST를 선택한다

### 4

---

⇒ `esp_http_client_init(&cfg);` 를 통하여 기본 설정 조작 후 핸들러 제작

### 5

---

⇒ Content-Type 필드에 넣을 데이터 가공

⇒ Content-Type에 넣을 데이터를 cType에 넣어서 보냄

⇒ 넘어간 cType은 서버가 데이터를 어떻게 읽을지 알려줌

```c
char ctype[128]; 
snprintf(ctype, sizeof(ctype), "multipart/form-data; boundary=%s", boundary);

esp_http_client_set_header(client, "Content-Type", ctype);
esp_http_client_set_header(client, "Content-Length", NULL);
```

⇒ esp_http_client_set_header 로 멀티파트를 사용함을 선언하고 ctype을 넘김

⇒ 다음 코드는 NULL을 부여함 → 이 헤더를 사용하지 않음을 선언

⇒ client가 들어가는 이유는 구조체를 넘겨 핸들로 관리하기 위해서이다

핸들 설정까지 끝났으면 서버와 TCP 연결을 맺고 데이터를 보낼 준비를 한다

⇒ `esp_err_t err = esp_http_client_open(client, content_len);`

### 6

---

⇒ 만약 ESP_OK가 나오지 않거나 esp_err_t err의 값이 NULL일 떄

```c
WRITE_FAIL:
	esp_http_client_close(client);
	esp_http_client_clean(client);
	esp_camera_fb_return(pic);
```

⇒ 열렸던 길을 다시 닫고, 클라이언트 핸들을 비우며 (메모리 관리) 찍은 사진 메모리를 반환한다

### 7

---

⇒ `esp_http_client_write(client, head, head_len)` 을 통해서 지정한 버퍼를 len만큼 body로 보낸다

### 8

---

⇒ 사진 전체 크기를 저장하는 포인터 변수 선언 (buf에는 jpg, jepg 같은 데이터가 들어있음

⇒ 사진을 보내고 남은 배열 크기를 저장하는 변수 선언 (초기값은 당연히 총 길이)

```c
const uint8_t *p = pic->buf;
size_t remain = pic->len;
```

```c
while (remain > 0)
{
    size_t chunk = (remain > 1024) ? 1024 : remain;
    w = esp_http_client_write(client, (const char *)p, chunk);
    if (w != (int)chunk)
    {
        ESP_LOGE(sendPhotoTag, "write jpeg 실패");
        goto WRITE_FAIL;
    }
    p += chunk;      
    remain -= chunk; 
}
```

⇒ 최대 1024바이트씩 보낸다

⇒ 1024를 성공적으로 보냈다면 w에는 1024가 들어간다

⇒ `w != chunk` 가 true일 경우 전송 실패

⇒ 아니라면 P의 주소에 이미 보낸 len을 더해 새로운 부분부터 보내기

⇒ 총길이에서 현재 보낸 길이를 빼서 연산

```c
w = esp_http_client_write(client, tail, tail_len);
if (w != tail_len)
{
    ESP_LOGE(sendPhotoTag, "write tail 실패");
    goto WRITE_FAIL;
}
```

⇒ 헤더, 사진을 모두 보냈다면 tail을 보내 종료

- 다음과 같이 서버에 요구사항을 header에 담아 보낼 수 있다

```c
esp_http_client_set_header(client, "Content-Type", ctype);
esp_http_client_set_header(client, "Accept", "application/json");
esp_http_client_set_header(client, "Accept-Encoding", "identity");
esp_http_client_set_header(client, "Connection", "close");
```

### 9

---

⇒ 전송을 마쳤다면 성공적으로 보내졌는지 확인해야함

```c
int status = esp_http_client_fetch_headers(client);   
int http_status = esp_http_client_get_status_code(client); 
```

⇒ status에 header만 읽고 성공시 헤더 길이를 반환

⇒ HTTP 상태 코드를 반환함 (ex : 404, 200, 503)

```c
if (resp_buf && resp_buf_sz > 0)
    {
        int total = 0;
        while (1)
        {
            int r = esp_http_client_read(client, resp_buf + total, (int)resp_buf_sz - 1 - total);

            if (r <= 0)
            {
                break;
            }
            total += r;                  
            if ((size_t)total >= resp_buf_sz - 1) 
                break;
        }
        resp_buf[total] = '\\0';
        ESP_LOGI(sendPhotoTag, "응답(%d) : %s", http_status, resp_buf);
    }
```

⇒ 저장할 공간이 충분하다면 서버 응답을 읽어와서 변수에 저장

⇒ 0이 더 크다면 전체 실패 → break

⇒ 그게 아니라면 total에 읽어론 len을 더해서 새로운 부분부터 조회

⇒ 다 읽었다면 종료

⇒ 마지막 널문자 넣고 로그 출력

⇒ 다음에 cJsonParsing에 필요할 수 있으므로

⇒ Json이 담긴 body, body의 length, 클라이언트 핸들을 확보한다

### 10

---

⇒ 모든 코드가 종료되면 메모리 확보를 위해 핸들러, 사진을 반남 후 서버 연결 종료

⇒ `return status;

[ESP32]]