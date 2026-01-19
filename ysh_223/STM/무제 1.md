> **Input Capture**
[[STM]]
---

⇒ GPIO 입력 엣지 (Rising or falling)가 들어오면 CNT 값을 CCRx에 저장

⇒ 엣지 시점의 타임스캠프 획득

**어디에 사용하나**

- 주기 측청
- 듀티 측정
- 엔코더

**사용 방법**

1. 주기

- 캡처를 상승엣지로 설정
- 캡처값 C1, 다음 캡처값 C2 읽기
- delta = (C2 - C1) mod (ARR + 1)
- T = delta * t_tick, f = 1/T

1. 펄스폭

- 상승엣지 t1
- 하강엣지 t2
- t_high = (t2 - t1) mod (ARR + 1)

### 중요 고려 사함

1. 오버플로 처리가 가장 중요
2. 인터럽트 지연은 측정값에 영향이 없음
3. 입력 노이즈가 있으면 Input Filter 사용

![[Pasted image 20260119090833.png]]

STM32F MCU에는 TIM_X 각각에 capture-campare하는 채널이 있다

- Input capture : 외부에서 signal 발생 시 해당 시점의 timer count값은 CCR에 저장
    
- CCR
    
    타이머에서 이벤트 시점을 저장하거나 (Capture), 이벤트 발생 기준을 정하는 값 (Compare)
    
- Output compare : Timer counter의 값이 미리 설정해둔 CCR 값과 같은 경우에 해당 pin으로 signal을 발생시킴
    

### 그림

위 그림에서 왼쪽이 input, 오른쪽이 output이다