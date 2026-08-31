Polling과 Interrupt 방식의 차이점은 
- Polling : CPU가 주기적으로 특정 장치의 변화된 값이나 상태를 체크하는 것이다.
  만약 Polling 주기가 길다면 원하는 시간 안에 제대로 된 이벤트 처리를 지원할 수 없다.
- Interrupt : 외부 장치인 HW나 SW적으로 값의 변화나 상태 변화를 직접 CPU에게 신호를 보내 알리는 방식이다. Interrupt 발생 시 ISR을 처리하며 오버헤드가 발생하지만, 해당 Interrupt는 사용자가 의도한 루틴대로 처리할 수 있다.


응답성이 중요한 button의 경우 Interrupt Pin 설정을 해주어야 한다.
외부 인터럽트 핀을 의미하는 EXTI를 설정하면 NVIC가 인터럽트 처리를 도우며 Interrupt 기능을 사용할 수 있다.

---

- GPIO : 범용 입출력 핀으로 센서나 외부 값을 받아오는 핀으로 사용할 수 있다.
- AFIO : UART, I2C, SPI, TIMER, 주변장치 등을 사용거나 EXTI 핀으로 사용할 수 있다.

Interrupt는 신호가 EXTI로 들어와서 NVIC에서 처리되고, 보드 코어에서 연산을 진행한다. 기초인 EXTI핀을 사용하려면 다양한 레지스터에 접근하여 컨트롤을 해줘야 한다.


- EXTI_IMR : Interrupt Mask -> 인터럽트 사용 시 
- EXTI_EMR : Event Msak -> 이벤트 사용 시
- EXTI_RTSR : Rising Trigger Selection -> Rising 엣지에서 발생
- EXTI_FTSR : Falling Trigger Selection -> Falling 엣지에서 발생

Event는 Interrupt가 아니기에 ISR에 들어가지 않고, 직접 CPU에게 보고하지 않는 방식이다.
IMR에서 버튼을 예시로 누름 (Rising) vs 뗌 (Falling) 에 따라 엣지 설정이 다를 것이다.

---

Datasheet에서 주소 체계 표시일 경우 -> 0x3UL
2진수 기반 표시일 경우 -> 0x2UL
십진수 -> 1UL 이다

여기서 또 비트 필드에 따라 나뉘는데, CRH/CRL 등은 핀 하나당 4비트를 사용한다.
하지만 APB2ENR은 0/1로 정할 수 있는 레지스터이므로 둘의 차이가 드러난다.

![[Pasted image 20260828012047.png]]
위와 같은 방식으로 Px13을 사용하면 EXTI13으로 모두 입력이 받아진다.

```c
#define EXTI_IMR_MR13       (0x3UL << 13)
#define EXTI_EMR_MR13       (0x3UL << 13)
#define EXTI_RTSR_TR13      (0x3UL << 13)
#define EXTI_FTSR_TR13      (0x3UL << 13)

// 위 코드와 아래 코드는 정확히 같은 역할을 수행한다.
#define EXTI_LINE13         (1UL << 13)
```

-----

![[Pasted image 20260831195041.png]]

```c
struct flags {
	int a : 1 
	int b : 3
	int c : 7
};
```
위와 같은 상황에서 a는 1 bit만 사용할 수 있다. 이는 C언어 정통 비트필드를 의미한다

![[Pasted image 20260831200343.png]]

실제로 우리는 이런 표를 참조하고
```c
#define GPIO_CRHCNF13 (0x3UL << 20)
```
같은 느낌으로 사용한다. 이는 하드웨어 레지스터 안에서 특정 비트 범위가 독립적으로 필드를 구성함을 나타낸다.

그러므로 우리는 레지스터 전체에서 논리적으로 독립된 각 비트들에 접근하여 값을 조정하는 것이다.

한 레지스터 전체에서 논리적으로 분리된 독립 비트를 접근하려면 다양한 방법이 있지만, HAL 드라이버 없이 개발하는 입장에서는 bit 레지스터 접근이 가장 편하다.
- & : 특정 핀 지우기 가능
- | : 특정 핀 세우기 가능 (지우는 게 불가하므로, clear 필수)
- ~ : Not

---

Interrupt 는 CPU가 하던 일을 멈추고, 특정 Interrupt에 따른 Interrupt Service routine을 처리하고 다시 기존 분기로 복기하는 방식이다.
이 상황에 3개의 Layer가 개입한다.

1. 주변장치 : EXTI의 RTSR, FTSR에 따라 pending bit인 pr bit를 표시한다.
2. NVIC : ARM Cortex-M 내부에 인터럽트 처리 담당 하드웨어이다. 인터럽트들의 활성화, 우선순위를 관리한다.
3. CPU core : NVIC가 Interrupt 처리 시작 핀에 신호를 주면 CPU는 벡터 테이블에서 Interrupt 번호에 매핑된 함수에 맞춰 ISR을 처리한다

pending 상태는 Interrupt는 발생했지만, 핸들러에 맞게 처리되지 않은 상태를 의미한다. EXTI가 Interrupt 신호를 보내고 NVCI는 Interrupt를 처리하므로, 이것들을 조율하여 CPU core로 이전한다.

NVIC는 하드웨어이므로, HW적으로 pending을 활성화/초기화한다. 하지만 EXTI는 SW적으로 직접 clear 해줘야 한다. (pr bit는 EXTI 비트이다.)
