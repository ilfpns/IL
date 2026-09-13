> ESP32-CAM 보드가 화면을 캡쳐하기 위해 필요한 과정이다

<aside>

구조체 값을 넘길 때 `esp_err_t err =` 에 값을 받아 ESP_OK인지 확인하며 로그를 찍어야 한다

---

1. ESP32-CAM 전용 DataSheet에서 PinMap을 확인하여 매크로로 지정해둔다
    
2. 카메라 설정 구조체를 만든다
    
    ⇒ camera_config_t camera_config = { . . . };
    
    ⇒ GitHub이나 GPT에게 기본 설정 구조를 받는다
    
3. 설정된 구조체를 가지고 카메라를 초기화한다
    
    ⇒ esp_camera_init(&camera_config);
    
4. 사진 촬영
    

[Camera config data](https://www.notion.so/Camera-config-data-25bd43ab359a80419769ce25adb63021?pvs=21)

</aside>

### 1

---

⇒ 핀맵은 DataSheet에서 복사해 GPT에게 매크로 제작을 요청한다 (직접 입력하면 너무 비효율임)

### 2

---

⇒ camera_config_t 로 구조체를 선언한다, 이 역시 PinMap을 확인하고 보드와 일치시켜 작성한다

⇒ 그 외 코드는 GitHub나 GPT에게서 참고한다

```c
.xclk_freq_hz = 20000000, // 20 MHz
.ledc_timer = LEDC_TIMER_0,
.ledc_channel = LEDC_CHANNEL_0,
.pixel_format = PIXFORMAT_JPEG, // 웹스트리밍/스냅샷 용
.frame_size = FRAMESIZE_QVGA,   // 800x600 (원하면 QVGA~UXGA 조절)
.jpeg_quality = 12,             // 0(최고)~63(최저)
.fb_count = 1,                  // 더 크게 하면 프레임 안정
.grab_mode = CAMERA_GRAB_WHEN_EMPTY,
.fb_location = CAMERA_FB_IN_PSRAM,
```

### 3

---

⇒ esp_camer_init(&camera_config) 로 구조체를 넘겨 카메라를 초기화한다

### 4

---

⇒ esp_camera_fb_get() 로 프레임 버퍼 구조체를 받는다, 구조체를 포인터 변수에 넣는다 (사진을 찍음)

⇒ pic→ ? 에 접근하여 다양한 정보를 확인할 수 있다

⇒ 실패시 NULL을 성공식 구조체를 반환한다
[[ESP32]]