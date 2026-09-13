STM32-MX에서 timer, clock 등을 설정할 때 clock configure을 설정해주어야 한다
[[ARM]]
![[Pasted image 20260325172205.png]]
**클럭이 뭘 의미할까?**

⇒ 컴퓨터 CPU가 연산 작업을 초리하는 속도 단위이다, 초당 펄스 수인 헤르츠다

⇒ nHz 클럭은 1초에 n번 신호가 반복됨을 의미하다

**클럭 관련 용어**

- HCLK : Core clock으로 실제 소스 코드를 동작시키는 clock이다
- SYSCLK : Sys clock으로 power on reset 직후에는 무조건 내부 clock으로 먼저 동작한다

---

- H : High
- S : Speed
- E : External
- L : Low

→ HSE : 외부 고속 clock, 하드웨어 부품인 크리스털이 필요하다

→ HSI : 내부 고속 clock, 보드 내부의 RC회로에서 동작하는 clock이다

→ LSE : 외부 저속 clock, RTC를 맞출 때 사용한다

→ LSI : 내부 저속 clock, RTC 다

- CSS : HSE clock에 문제가 있을 때, 인터럽트 발생 및 clock source를 HSI로 변경해주는 기능이다

---

Timer clock은 어떻게 계산할까?

2가지의 계산 장치로 클럭을 조정함

- PLL : 곱셈 (증폭)
- Prescaler : 나누기 (감소)
    - Counter Period : 목표 값까지의 카운팅 범위

STM32 보드에서 명세된 클럭 수 = 72MHz ( = 72,000,000Hz)

![[Pasted image 20260325172219.png]]
다음은 헤르츠 공식이다

![[Pasted image 20260325172234.png]]
$$ 72,000,000/7,199 * 99 $$

라는 공식이 도출된다

- Prescaler : 7,199 ( + 1 = 7200)
- CP : 99 ( + 1 = 100)
- Timer의 목표 클럭수를 100Hz로 설정함