> **Preemption Model**
[[Scheduling]]
---

⇒ 최악의 지연시간이 상한을 가지는가

⇒ CPU가 커널 코드를 실행할 때, 언제 task switch가 허용되는지를 정의한 규칙 집합

- IRQ 차단 시간
- lock을 잡고 있는 시간
- 선점 불가 구간 길이

세 개를 줄이는 문제임

### Linux ≠ RT

- 리눅스는
    - Throughput
    - Fairness
    - SMP 확장성
    - 을 목표로 한다

⇒ 긴 Critical Section 허용, spinlock기반 IRQ disable 구간이 존재한다

⇒ 지연 시간 상한을 보장하지 않음

---

### No preempt

- 중간 교체 불가
- 인터럽트는 처리되지만, task교체는 syscall 종료 전까지 불가
- scheduling latency = 커널 실행 시간
- 우선순위가 높아도 기다림

### Preempt (일반 리눅스)

- 규칙하에 Task 교체 가능
- Spinlock보유, IRQ off 때 선점 불가
- 평균 latency는 낮지만 최악 latency는 여전히 큼

### Preempt RT

- spinlock 제거, spinlock → rtmutex
- mutex는 sleep 가능
- hardirq는 최소 처리만 수행

### RTOS와의 관계

- RTOS: 처음부터 전 구간 선점 설계
- Linux: 비선점 설계 → RT 패치로 구조 변경

### 결론

⇒ Preemption Model은 커널 실행 중 task switch를 어디까지 허용할 것인가를 정한 정책