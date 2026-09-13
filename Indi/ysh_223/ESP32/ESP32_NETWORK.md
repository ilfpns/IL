> ESP32-CAM 보드가 서버나 다른 기기를 위해 HTTP 통신같은 방법을 사용한다
> 
> 이는 같은 네트워크를 사용해야 한다

<aside>

구조체 값을 넘길 때 `esp_err_t err =` 에 값을 받아 ESP_OK인지 확인하며 로그를 찍어야 한다

---

1. 현재 사용하는 Wi-Fi의 Id와 password를 매크로로 작성한다
    
    ⇒ const char *ssid = “id”
    
    ⇒ consrt char *password = “password”
    
2. 와이파이 핸들러를 작성한다 → 현재 와이파이 상태 변화를 처리하기 위해 작성한다
    
    ⇒ static void wifi_event_handler(void *handler_arg,
    
    ```
                                                       esp_event_base_t base,
    
                                                       int32_t event_id, 
    
                                                 void *event_data)
    ```
    
3. 와이파이 초기화 → 앞으로 사용할 와이파이를 초기화한다
    
    ⇒ esp_netif_create_default_wifi_sta()
    
    ⇒ wifi_init_config_t wifi_init_config = WIFI_INIT_CONFIG_DEFAULT();
    
    ⇒ esp_wifi_init(&wifi_init_config)
    
4. Id, Password, mode가 담긴 설정 만들기
    
    ⇒ wifi_config_t witi_config = { . . .};
    
5. 와이파이에 연결한다 → 초기화 후 입력된 새 와이파이 정보들로 연결을 시작한다
    
    ⇒ esp_wifi_set_mode(WIFI_MODE_STA)
    
    ⇒ esp_wifi_set_config(WIFI_IF_STA, &wifi_conig)
    
    ⇒ esp_wifi_start()
    

</aside>

- STA → Wi-Fi 공유기에 접속하는 클라이언트 역할 (연결 요청 / 허락 처리) → 수신
- AP → Wi-Fi 공유기 역할 (접속 허용 / 관리) → 송신

### 1

---

⇒ 현재 사용하는 와이파이 Id, Password이여야 한다

### 2

---

⇒ 핸들러는 이벤트를 감지하는 콜백 함수로 작동한다

⇒

[Network_handler_t](https://www.notion.so/Network_handler_t-259d43ab359a802d95dcd65a99a87869?pvs=21)

```c
static void wifi_event_handler(void *handler_arg,  
                                     esp_event_base_t base,
                                     int32_t event_id, 
                                     void *event_data)
// 사용자 정의 인자, 이벤트 종류 인자, 이벤트 세부 id, 이벤트에 대한 추가 구조체
```

⇒ 핸들러로 인자들을 받았다면 `base == WIFI_EVENT` 일 때 상태를 검사한다

⇒ 세부 id (event_id)에서 각 상태들에 맞는 처리를 해준다

- 성공 → 로그 출력
- 실패 → 로그 출력 후 재시도 (esp_wifi_connect())
- 시도중 → 로그 출력

⇒ 해당 이벤트 구조체를 열어보려면

```c
// IP = WIFI_EVENT_AP_STACONNECTED
wifi_event_ap_staconnected_t *event = (wifi_event_ap_staconnected_t *)event_data;
// IP = IP_EVENT_STA_GOT_IP
ip_event_got_ip_t *event = (ip_event_got_ip_t *)event_data;
```

⇒ IP이름 자료형으로 포인터 변수를 만든다

⇒ 핸들러의 event_data는 void 포인터이므로 캐스팅이 필요하다

### 3

---

```c
ESP_ERROR_CHECK(nvs_flash_init()); // nvs(플래시 메모리 사용)를 초기화하여 값을 넣을 준비
ESP_ERROR_CHECK(esp_netif_init()); // NewtWork InterFace 초기회(IP 할당 관리 역할)
ESP_ERROR_CHECK(esp_event_loop_create_default()); // 기본 이벤트 루프 생성
```

⇒ 기본적으로 사용할 공간이나 설정을 초기화 하고

⇒ `esp_netif_create_default_wifi_sta();` 로 Wi-Fi sta 생성

⇒ 생성했으므로 handler 호출이 필요함 (상태를 관리해야하기 떄문에)

```c
esp_handler_instance_register(WIFI_EVENT, ESP_EVENT_ANT_ID, &wifi_event_handler, NULL, NULL);
```

⇒ 핸들러에 각각 값을 넘긴다

⇒ (base, id(ANY → 해당 그룹의 모든 이벤트를 다 받음), 핸들러 구조체, 옵션, 옵션)

### 4

---

```c
 wifi_config_t wifi_config = {
        .sta = {
            .ssid = "ORBI96",
            .password = "moderncurtain551",
            .threshold.authmode = WIFI_AUTH_WPA2_PSK,
        },
    };
```

⇒ wifi_config에서

1. ID를 넘김
2. PASSWORD를 넘김
3. 연결할 AP의 최소 인증 방식 지정

### 5

---

```c
ESP_ERROR_CHECK(esp_wifi_set_mode(WIFI_MODE_STA));               // STA 모드로 설정
ESP_ERROR_CHECK(esp_wifi_set_config(WIFI_IF_STA, &wifi_config)); // SSID/PASS 설정 적용
ESP_ERROR_CHECK(esp_wifi_start());                               // Wi-Fi 연결

ESP_LOGI(WifiConfigTag, "wifi_init finished. SSID:%s password:%s", wifi_config.sta.ssid, wifi_config.sta.password);
```

⇒ esp_wifi_set_mode를 STA모드로

⇒ esp_wifi_set_config에 wifi interface를 STA(수신 모드)로 설정, 구조체 전달

⇒ 연결 시작

⇒ 구조체 안에 구조체 안에 값들이 있기 때문에 `.` 2개로 접근

[[ESP32]]