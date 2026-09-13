> **전통적 tick**
[[Linux]]
---

- 커널은 기본적으로 HZ주기로 타이머 인터럽트를 발생시켜 시간 관리, 스케줄링, 통계 등을 처리한다
- 이 주기를 tick이라 하며, jiffies는 tick 발생 횟수를 누적한 전역 카운터다

### 문제점

- Idle 상태에서도 주기적인 tick이 발생함
    
    ⇒ 불필요한 인터럽트 / 전력 낭비
    

> **Tickless (Dynamic Tick)**

---

⇒ 주기적인 tick 대신 필요한 경우에만 타이머 인터럽트를 발생시키는 구조

### NO_HZ 옵션

CONFIG_NO_HZ_IDLE / CONFIG_NO_HZ_FULL 등 설정으로 tick 인터럽트를 줄인다.

- dle 상태에서 tick을 멈춤 → dyntick-idle
- 시스템 전체 혹은 커널 상태에 따라 더 폭넓게 tick을 줄이는 옵션도 존재.

> **High-Resolution Timers**

---

- x86/ARM 등에서 one-shot 타이머 하드웨어를 기반으로 타이머 인터럽트 시점을 정확히 제어하는 메커니즘.
- HSRT(High-Resolution Timers)는 주기 tick을 대체하거나 보완한다

|모드|tick 발생 방식|특성|
|---|---|---|
|**Periodic tick**|모든 CPU가 HZ 주기마다 tick|구현이 단순하지만 항상 인터럽트가 발생|
|**dyntick-idle**|Idle CPU는 tick 멈춤|인터럽트 감소, 전력 절감|
|**full tickless**|대부분 CPU에 tick 없음|최대 절전, 스케줄러 tick 없음|

> **Points**

---

1. Tick이 단순히 제거되는 것이 아니다
    - 다른 이벤트로 대체되거나, 동적 스케줄링으로 계속 유지한다
2. Idle vs active 범위
    - idle (저전력)에서만 tickless인 경우와, active에서도 tick을 주는 경우가 다르다
3. jiffies 유지
    - 주기 tick이 없어도, sleep 후 wake-up에서 보정하여 jiffies를 유지한다
4. trade-off 존재
    - tick을 줄이면 전력/Idle residency는 좋아지지만, 미세 타이밍/RT 성능이나 TSR에 영향을 줄 수 있다