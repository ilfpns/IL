[[ESP32]]
> ESP32-CAM 보드가 서버나 다른 기기를 위해 HTTP 통신같은 방법을 사용한다 그러므로 HTTP 통신 설정을 해줘야한다

<aside>

구조체 값을 넘길 때 `esp_err_t err =` 에 값을 받아 ESP_OK인지 확인하며 로그를 찍어야 한다

---

1. HTTP 클라이언트 핸들을 만들어서 사용할 수 있게 한다
    
    → esp_http_client_handle_t로 만들어진 핸들을 사용함
    
2. 콘텐츠 길이와 http 상태 코드를 구한다
    
    → esp_http_client_get_status_code(client)
    
    → esp_http_client_get_content_length(client)
    
3. content 길이 만큼 동적으로 메모리 공간 확보
    
    → cJson 형식으로 넘어온 메시지를 읽어들임
    
    → esp_http_client_read_response
    
4. Json 트리 루트 노드 생성
    
5. root로 값 접급해서 데이터 조회 </aside>
    

## 1

---

- `int r = esp_http_client_read(client, buffer + total, content_length - total);` 핸들러를 사용하여 접근
- 값을 읽을 준비를 함

## 2

---

- 데이터의 content_length와 http_status_code를 구한다

```c
int64_t content_length = esp_http_client_get_content_length(client);
int status = esp_http_client_get_status_code(client);
```

## 3

---

- 미리 구해진 Length를 활용하여 미리 선언한 버퍼만큼 동적할당을 한다

```c
char *buffer = malloc(content_length + 1);

n = esp_http_client_read_response(client, buffer + total_read, content_length - total_read);
total_read += n;
```

- `esp_http_client_read_response` 는 읽은 데이터의 길이를 반환한다
- total_read += n 을 하여 읽은 부분은 제외하고 읽는다
- 정보가 담긴 body를 파라미터로 받았다면 필요 없음

## 4

---

```c
cJSON *root = cJSON_Parse(buffer);
```

- 위 코드는 json의 값에 접근하기 위해 최상위 노드의 주소를 포인터에 담는다
- root 는 cJson_Parse의 반환값으로, 파싱된 전체 JSON 문서의 최상의 노드를 가르킴
- root 작업이 끝나면 cJSON_Delete(root)를 해줘야 한다 (자식 노드까지 제거)
- 그 후 buffer를 free 해주면 된다
- 만약 Parse를 할 수 없다면 받은 데이터가 `정확히` Json인지 파악해야한다

## 5

---

```c
cJSON *msg = cJSON_GetObjectItem(root, "message")
if (cJSON_IsString(msg) && msg->valuestring)
        {
            int value = atoi(msg->valuestring);
            ESP_LOGI(cJsonParsingTag, "값 : %d", value);
        }
```

- 다음과 같이 root에서 “message”를 분리하여 가져온다
    
- mag에 값이 정상적으로 들어오면, 구조체 msg에 접근해 값을 추출한다
    
- `cJSON_GetObjectItemCaseSensitive` 는 key이름의 대소문자를 구분한다 `(test ≠ TEST)`
    
- `cJSON_GetObjectItem` 는 key 이름의 대소문자를 구분하지 않는다 `(test == TEST)`
    

> 자주 생기는 함정
> 
> - `else if (root)` 는 위에서 실패 시 `return` 했으니 사실상 불필요한 중복 조건이지만, 동작엔 문제 없다
> - 숫자를 다룰 땐 `atoi`(문자열→정수) 대신 JSON이 숫자 타입이면 `cJSON_IsNumber(node)`+`node->valueint/valuedouble`를 쓰는 편이 더 안전할 때가 있다
> - `root`가 배열일 수도 있는 API라면, `GetObjectItem` 대신 배열 API를 써야 한다