## Bank0
[[ESP32]]
MCU는 보통 32비트이다

이 때 MCU의 핀이 40개가 넘어간다면? 32비트 레지스터 하나로 모든 핀을 통제할 수 없다

그렇기 떄문에 핀들을 32개씩 묶어서 나는 구역을 Bank라고 한다

- 32bit
    
    데이버 버스에서 bit가 한번에 32개를 움직인다
    
    ESP32, STM32 보드 32bit 데이터 버스를 가진다
    
- bank 0 : 0 ~ 31 pin
    
- bank1 : 32 ~ 63 pin
    

## W1TS (Write 1 To set) / W1TC (Write to 1 clear)

- pin에 대해 전기를 high할지 low할지 결정하는 전용 스위치 레지스터이다
- W1TS : high를 쏘면 전기가 들어온다
- W1TC : high를 쏘면 전기가 끊긴다

### 왜 사용할까?

레지스터 제어는 속도와 안정성 제어를 장점으로 가진다

- 원자성 : 레지스터를 제어하기에 1클럭에 실행되므로 atomic하다
- 속도 : 레지스터를 직접 제어하므로 빠른 속도를 보여준다

```c
// 예시: ESP32 기준
// Bank 0 (0~31번 핀) 제어 레지스터
#define GPIO_OUT_W1TS_REG  (*(volatile uint32_t *)(0x3FF44008)) // 켜기
#define GPIO_OUT_W1TC_REG  (*(volatile uint32_t *)(0x3FF4400C)) // 끄기

// Bank 1 (32~39번 핀) 제어 레지스터 
#define GPIO_OUT1_W1TS_REG (*(volatile uint32_t *)(0x3FF44014)) 
#define GPIO_OUT1_W1TC_REG (*(volatile uint32_t *)(0x3FF44018)) 

// --------------------------------------------------------

// 1. 4번 핀 켜기 (4번 핀은 31 이하이므로 Bank 0 사용)
GPIO_OUT_W1TS_REG = (1 << 4); 

// 2. 33번 핀 켜기 (33번 핀은 32 이상이므로 Bank 1 사용!)
// 주의: 33번 핀을 제어할 땐, 33칸을 미는 게 아니라 (33 - 32 = 1)칸만 민다
GPIO_OUT1_W1TS_REG = (1 << (33 - 32));
```