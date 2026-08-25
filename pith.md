STM보드를 사용할 때 실제로 어떤 프로젝트를 생성할 때에도 다음과 같은 프로세스를 거친다.

`폴더 생성 -> 보드 선택 -> 클럭 설정 -> 보드 설정 -> 개발`

PITH의  경우에 앞 두 단계를 건너뛰므로, 바로 Clock -> Pin 설정을 해줘야 한다.
Nucleo 보드를 기준으로 PA5는 LED에 배선 되어있다.

필요에 따라
1. RCC APB2 초기화 (CLK)
2. GPIO 초기화       (PIN)

![[Pasted image 20260825231748.png]]
Reset and Clock Control (RCC) -> IOPAEN을 초기화 하려면 2번 시프팅을 해야한다.

```c
RCC_APB2ENR_IOPAEN (1UL << 2)
```
RCC의 APB2ENR을 IOPAEN으로 설정하는 코드는 이런 식으로 작성해야 한다. 
이렇게 클럭을 리셋한다.

이제 사용할 Pin을 reset해야한다.

GPIO의 Configuration Register Low (CRL) 중 PA5를 설정하므로 
Low는 0~7번 핀 담당이다 HIgh는 8 ~ 15 핀 담당이다.
- CNF5 (22)
- MODE5 (20)
을 사용해야한다. Input Mode이므로 

- << 22
- << 20
으로 진행한다.


![[Pasted image 20260825232211.png]]![[Pasted image 20260825232511.png]]

```c
    RCC->APB2ENR |= RCC_APB2ENR_IOPAEN;

    GPIOA->CRL &= ~(GPIO_CRL_CNF5 | GPIO_CRL_MODE5);
    GPIOA->CRL |= GPIO_CRL_MODE5_0;

```
설정해준다.

---
이후 어떻게 LED 점멸을 해야 할까 
Bit Set/Reset Register핀 설정을 만져줘야 한다. 

GPIO 핀은 원자적으로 High/Low를 제어할 때 사용한다.
- 해당 핀을 Set : 0 ~ 15
- 해당 핀을 Low : 16 ~ 31

BRR은 Reset 핀으로 low용도로만 사용한다.
같은 원리로 BS는 Set high, BR은 Set Low와 같다.

```c
    RCC->APB2ENR |= RCC_APB2ENR_IOPAEN;

    GPIOA->CRL &= ~(GPIO_CRL_CNF5 | GPIO_CRL_MODE5);
    GPIOA->CRL |= GPIO_CRL_MODE5_0;
    
	GPIOA->BSRR = GPIO_BSRR_BS5;
    for (volatile uint32_t i = 0; i < 500000; i++) {}

    GPIOA->BSRR = GPIO_BSRR_BR5;
    for (volatile uint32_t i = 0; i < 500000; i++) {}

```
