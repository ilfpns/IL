> **Latency**
[[OS]] [[Scheduling]]
---

⇒ 리눅스에서 이벤트가 발생한 시점부터 코드가 실행되기까지 걸리는 지연 시간

1. Interrupt Latency
    
    ⇒ 하드웨어 IRQ (interrupt request)→ 커널이 ISR (interrupt service routine)을 실행하기까지 걸리는 시간
    
    - interrupt masking
    - softirq vs hardirq
    - preempt_disable 영향
    - 커널 락이 IRQ latency에 미치는 영향
    - CPU 캐시/메모리 영향
2. Scheduling Latency
    
    ⇒ 프로세서가 실행 가능 상태가 된 순간부터 CPU 점유를 할 때까지 걸리는 시간
    
    - CFS target latency
    - wakeup path
    - runqueue contention
    - migration
    - cgroup scheduler 영향
3. System call latency
    
    ⇒ 유저 → 커널 전환 비용 + 커널 내부 처리 지연
    
    - syscall 경로
    - context switch cost
    - page fault handling
    - schedule point 등장
4. Timer latency
    
    ⇒ timer가 expire되는 시점
    
    - HRT (High Resolution Timer)
    - tickless kernel
    - NO_HZ_FULL
    - timer migration
    - interrupt throttling