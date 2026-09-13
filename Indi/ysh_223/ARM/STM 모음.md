![[Pasted image 20260110223439.png]]

4번째 칸이 중요함

3번 타이머

클럭에 중요성

클럭수가 높다면

- 연산 빨라짐, 폴링이 자주 일어나겠지?
- 발열/전력 소모 이빠이

---

EXTI ⇒ External Interrupt

외부 핀이다 (버튼, 센서 등)

인터럽트를 발생하는 하드웨어

### **핀**

EXTI0 → GPIO핀 0번과 연결

P (A, B, C ..) (1, 2, 3 ..)

Port는 독립된 GPIO 레지스터 세트라 본다

---

system core / MCU기본 기능/ RCC, GPIO, NVIC, SYS

Analog / 아날로그 입출력 / ADC, DAC, 온도센서

Timers / 타이머, 카운터 / PWM 출력, 주기적 인터럽트

Connectivy / 통신 관련 / UART, SPI, I2C, USB, CAN

Multimedia / 오디오, 영상 / LCD, DCMI, SAI

Security / 보안 관련 / CRS, HASH, RNG 등

Computing / 연산, 처리 유닛 / DMA, FPU

Middleware and software Packages / 상위 소프트웨어 구성요소 / FreeRTOS, Stack

---

![[Pasted image 20260110223454.png]]

크리스털 : 진동자라고 불린다 (아주 정확한 주파수로 전기 신호를 진동시키는 물리 부품) MCU가 이 신호를 받아 정확한 시간 기준(클록)으로 사용한다

⇒ MCU 가 시간을 재는 기준 (PLL X 크리스털 = CPU 클럭 속도)

HSE : 보드에 달린 크리스털 클록

HSI : 내부 기본 클록

PLL : 클록 배수기 (CPU 속도, 보통 168MHz)

SYSCLK : MCU 메인 클록 (CPU가 쓰는 속도)

---

### HSI HSE

HSE 가 ㅈㅉ 정확

HSI : MCU안에 내장 / 항상 켜져 있음 / 바로 사용 가능

⇒ 단순 GPIO, LED, 기본 로직, 임시 테스트

HSE : 보드에 크리스털 부품 필요 / 보드에 따라 켜야함 / HSE enable 해야함

⇒ 통신 UART, USV, I2C 등, 정밀 타이밍이 필요

⇒ 하드웨어 필요, 코드 더 복잡

HSE가 멈췄는지 자동 감지하고, 멈추면 HSI로 전환

![[Pasted image 20260110223510.png]]

원하는 모드 선택, HSI를 입력 소스로 씀

![[Pasted image 20260110223520.png]]

크리스털

---

![image.png](attachment:6ba05577-7be8-49d3-88e9-dd31864dbdc3:image.png)

project → properties → c/c++ build → setting → convert to bin file

빌드할 때 이진수 파일로 변경해서 파일 용량 저하 및 경량화

---

![[Pasted image 20260110223529.png]]

![[Pasted image 20260110223540.png]]

점퍼선이라 함 JP1

멀티미터기를 꼽아서 전류 정상적으로 실행되는지 확인하는 JP핀

⇒ MCU 소모 전력을 측정하기 위해 사용한다

sleep, stop 등 정확한 전력소모를 측정하기 위해 사용

|예시 JP 이름|역할|설명|
|---|---|---|
|**JP1 (IDD)**|전류 측정용|MCU 전원선(VDD)을 분리해서 전류 측정|
|**JP2 / JP3**|ST-LINK 디버거 연결|ST-LINK ↔ MCU 연결을 끊거나 연결|
|**JP4 (BOOT0)**|부팅 모드 설정|플래시 부팅 / 시스템 부트로더 선택|
|**JP5 (USB_PWR)**|USB 전원 분리|USB 전원과 외부 전원 선택|
|**JP6, JP7 ...**|확장 핀 선택|주변 회로나 테스트 포인트용|

⇒ 어떤건 전류 측정

⇒ 어떤건 전원, 통신, 부트 경로 선택용

---

### Src폴더 내 파일들

- stm32F4xx_hal_msp
    
    - msp = MCU support Package
    - 각 주변장치 저수준 초기화
    - 핀/클럭/IRQ 설정 등을 한 곳에 모아둔 파일
    - IRQ : 인터럽트 처리 요청을 보냄
- stm32f4xx_it.c
    
    - IT = Interrupt
    - 모든 인터럽트 핸들러 함수가 들어가는 곳
- syscalls.c
    
    - 표준 c라이브러리가 필요로 하는 시스템 콜 (_whrite, _close, _sbrk) 등
    - 원래 OS가 하는 일을, MCU환경에 맞게 연결하는 것
    - printf, malloc 같은 함수를 내부적으로 시스템 콜을 호출하기 때문에
- sysmem.c
    
    - 주로 _sbrk() 구현
    - _sbrk : malloc 같은 힘 메모리를 늘릴 때, 실제 메모리를 할당해주는 함수
    - malloc/free 사용시 힙을 어디까지 쓸 수 있는지 알려주는 함수

---

![[Pasted image 20260110223553.png]]

기본설정 시!!발

generate per`~ 어쩌고 이거 켜야 라이브러리 호출하거나 MCU설정에 따라 자동 파일 생성해줌ㅇㅋ??

---

### Vector Table

⇒ MCU가 리셋되자마자 가장 먼저 참조하는 테이블 (플래시 맨 앞에 위치한다)

- 인터럽트/예외의 진입 주소 들이 순서대로 들어있다

```c
.word _estack          ; 0번: 초기 스택 포인터
.word Reset_Handler     ; 1번: 리셋 핸들러
.word NMI_Handler       ; 2번: NMI
.word HardFault_Handler ; 3번: 하드폴트
```

MCU가 부팅될 때 estack 값을 Stack Pointer로 로드

Reset_Handler 주소로 Program Counter 점프

Reset_Handler가 Startup.s에 정의됨 (어셈블리 언어로 작성됨)

### startup.s 란?

⇒ 벡터 테이블과 초기화 루틴을 담은 파일

- 스택 초기화
- . data, .bss 영역 초기화 (RAM에 값 복사)
- SystemInit() 호출 (클럭 세팅)
- main() 진입

---

클럭 트리

⇒ 어떤 신호를 기준으로 CPU와 주변 장치가 얼마나 빠르게 동작할지 결정함

- PLL (Phase-Locked Loop)
    
    - 주파수를 배수로 증폭시키는 회로
    - HSE(8 MHz X PLL = 168MHz → CPU클럭 생성
        - PLL Lock : 입력 클럭을 기준으로 정확한 고속 클럭을 생성하는 회로
        - ⇒ 입력 클럭과 위상을 맞출 때까지 시간이 걸림
        - ⇒ 입력 주파수와 위상이 일치하며 안정된 상태가 되면 PLL이 LOCK됨
- SYSCLK
    
    - MCU 메인 클럭
    - CPU/AHB/APB 버스의 기준이 됨
- RCC
    
    - MCU 내부의 시스템 클럭과 주변 장치 클럭을 제어하는 하드웨어 코드를 짤 때 사용하는 라이브러리
    - RCC : 실제 하드웨어 레벨의 클럭 제어 레지스터
    - HALL_RCC_* : RCC를 다루기 위한 HAL 라이브러리 함수

---

---

리눅스 커널 계열 개발자 (FW or Linux kernnel DP 누가 더 high s)