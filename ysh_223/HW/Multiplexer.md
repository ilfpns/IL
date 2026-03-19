## Multiplexer
[[HW]]
⇒ 2^n개의 입력 신호 중 n개의 선택 신호를 입력하여 입력 신호 중 하나를 선택하여 출력함

⇒ 여러개의 입력 중 하나의 선택을 한다

- Example
    
    1. TV는 플스, 셋톱박스, 노트북 중 하나만 선택해서 화면에 띄움
    2. 4 x 1 멀티플렉서란 4(2^2)개읩 입력 신호 중, 2개의 선택 신호를 이용하여 입력 신호 중 하나를 선택하여 출력하게 된다
- 많은 입력 중 하나를 선택하므로, 데이터 선택기 라고도 불린다
    

![[Pasted image 20260319165533.png]]
### 왜 사용할까?

이런 어려운 개념을 가진 멀티플렉서를 어디에 사용할까?

- MCU에 구현된 I2C, SPI, UART에 전부 선을 사용한다면, pinhead가 너무 많아진다
    
    ⇒ MUX를 사용하여 현재 사용하지 않는 기능은 끊어두고, 필요한 기능만 스위치로 선택하여 사용할 수 있도록 한다
    

### 어디에 사용할까?

Pinheader와 칩 내부의 기능 사이에서 MUX가 이를 이어준다

- 입력 : 칩 내부에 위치한 UART, PWM, GPIO
- 출력 : 보드에 하드웨어적으로 존재하는 물리 Pinheader
- 선택 신호 : 코드에서 어떤 기능을 사용할지 register로 선택

### Datasheet에서 볼 수 있는 I/O MUX는 뭘까?

- 상황
    
    <aside>
    
    ESP32에서 UART, SPI, I2C 등 여러 선이 존재하지만, 실제로는 물리적인 핀 40개에 모두 연결되어 있다
    
    </aside>
    
- 해결책
    
    ⇒ 모든 기능마다 핀헤더를 만들 수 없으므로, 물리적인 핀 1개 앞에 MUX스위치를 사용하여 개발자가 조정할 수 있도록 만든다
    
- 실제 상황
    
    Pin 4 : GPIO4 / UART1_TX / ADC2_CH0 / PWM_OUT (핀 하나에 열러 기능이 몰림)
    
    - 선택 0: 그냥 전원만 사용 (GPIO 모드)
    - 선택 1: 센서랑 시리얼 통신하는 선으로 사용 (UART1_TX 모드)
    - 선택 2: 아날로그 전압값 읽는 선으로 사용 (ADC 모드)
    - 선택 3: 모터 속도 조절하는 선으로 사용 (PWM 모드)
    
    ```c
    #define IO_MUX_BASE_ADDR     0x3FF49000 
    #define IO_MUX_GPIO4_REG     (*(volatile uint32_t *)(IO_MUX_BASE_ADDR + 0x0048)) 
    
    // MUX 스위치를 조작해서 4번 핀을 '일반 GPIO' 기능으로 연결하는 매크로
    // (ESP32의 경우 MCU_SEL 비트를 조작하여 기능을 선택한다)
    #define MUX_SET_AS_GPIO(pin_reg)  (pin_reg = (pin_reg & ~(7UL << 12)) | (2UL << 12)) 
    
    #ifndef GPIO_BASE_ADDR
    #define GPIO_BASE_ADDR 0x3FF44000 
    #endif
    
    #define PIR_SENSOR_PIN 4
    
    // bank0
    #define GPIO_ENABLE_W1TC_REG    (*(volatile uint32_t *)(GPIO_BASE_ADDR + 0x0028)) // 0 ~ 31 출력 비활성화
    #define GPIO_IN_REG             (*(volatile uint32_t *)(GPIO_BASE_ADDR + 0x003C)) // 0 ~ 31 입력 읽기
    
    #define FAST_GPIO_INPUT_EN(pin) (GPIO_ENABLE_W1TC_REG = (1UL << (pin)))
    #define FAST_GPIO_READ(pin)     ((GPIO_IN_REG >> (pin)) & 1UL)
    
    #endif
    
    void gpio_pin_init(void) {
    		// MUX를 이용해 내가 원하는 기능에 연결
        MUX_SET_AS_GPIO(IO_MUX_GPIO4_REG);    
        FAST_GPIO_INPUT_EN(PIR_SENSOR_PIN);
    }
    ```
    

### Demultiplexer (DeMUX)

MUX와 달리 한 선으로 입력을 받아서 2^n개의 출력을 하는 회로이다