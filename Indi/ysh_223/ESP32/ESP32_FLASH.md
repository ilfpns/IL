> 임베디드에서 디지털은 0과 1밖에 나타내지 못한다.
> 
> 그렇기에 디지털을 아날로그 같이 출력하는 것을 PWM이라 한다

<aside>

구조체 값을 넘길 때 `esp_err_t err =` 에 값을 받아 ESP_OK인지 확인하며 로그를 찍어야 한다

---

1. 타이머 / 채널 설정 → 앞으로 사용할 채널과 타이머를 설정함
    
    ⇒ ledc_timer_config_t / ledc_channel_config_t
    
2. 범용입출력핀(GPIO) 설정 → 사용할 핀을 설정한다
    
    ⇒ gpio_pad_select_gpio(pinNumber)
    
    ⇒ gpio_set_direction(pinNumber, mode)
    
    ⇒ LED는 GPIO_OUTPUT_MODE
    
3. 타이머 / 채널 설정 → 구조체에서 설정한 값을 HW로 넘긴다
    
    ⇒ ledc_timer_config(&structureName);
    
    ⇒ ledc_channel_config(&structureName);
    
4. 전원 입력 → Flash 코드를 실행한다
    
    ⇒ ledc_set_duty(LEDC_LOW_SPEED_MODE, LEDC_CHANNEL_0, 100)
    
    ⇒ ledc_update_duty(LEDC_LOW_SPEED_MODE, LEDC_CHANNEL_0)
    
    ⇒ vTaskDelay(100 / portTICK_PERIOD_MS)
    

</aside>

### 1

---

```c
/* 1 */ ledc_timer_config_t ledc_timer = {~};
/* 2 */ ledc_timer_config(&ledc_timer);
```

⇒ 코드 1은 구조체를 선언하고 설정할 때 사용한다

⇒ 코드 2는 선언한 구조체를 하드웨어에 적용한다

⇒ t는 보통 `type` 의 약자이다

⇒ 선언을 성공하면 `ESP_OK` 를 반환한다

---
### 2

---

→ 보통 전처리기로 pinNumver을 지정한다
### 3

---

```c
esp_err_t ret = ledc_channel_config(&ledc_channel);
ESP_LOGE("FLASH LIGHT", "ERROR");
ESP_LOGE("FLASH LIGHT", "ERROR : %d", ret);
```

⇒ ESP_LOG_ERROR이다

⇒ 첫 번째 인자는 로그를 구분할 태그 이름이 들어간다

⇒ 두 번째 인자는 출력할 문자열을 적는다

⇒ `esp_err_t` 는 int형 32비트 자료형이므로 %d를 쓴다

### 4

---

→ PWM이기 때문에 파라미터 인자에 각각 (속도, 사용할 채널, duty비) 를 넣는다
[[ESP32]]