### CPU Scheduler
[[Scheduling]]

- 연성 (Soft)
- 경성 (Hard)

두가지 방법으로 나뉜다

---

### 연성

- 스케줄링에 대해 아무런 보장을 하지 않는다
- 중요 프로세스가 우선권을 가진다는 것만 보장

### 경성

- 스케줄링에 대해 엄격한 요구 조건을 가짐
- 정해진 시간에만 스케줄링 서비스를 처리한다

---

### Real-Time Scheduling

⇒ 실시간 시스템에 사용되는 Deadline에 따른 task를 이루는 스케줄러

⇒ 각 Task는 Deadline을 지키며 과제를 수행해야 함

**Deadline을 지킬 수 있는 스케줄링 기법에는 뭐가 있을까?**

- Rate Monotonic Scheduling (RMS)
- Earliest Deadline First (EDF)

---

## RMS

⇒ 각 프로세스 중 rate가 가장 높은 프로세스부터 스케줄링을 하는 기법

- 처음 rate를 사용하여 priority를 계산하여 이를 가지고 스케줄링을 진행함
- Flow
    1. 두개의 Task가 존재함
        1. P1 (주기 : 50), P2 (주기 : 100)
    2. 짧은 주기 = 높은 rate
    3. P1 → P2 순으로 사용됨

특징

- Static한 Priority를 부여함

## EDF

⇒ Deadline이 가까워져 올 때, 가장 급한 task먼저 스케줄링 하는 기법

특징

- task들이 들어오는 순간마다 priority를 부여함
- 오직 deadline만을 이용함으로, CPU burst time을 몰라도 사용할 수 있다
    - CPU burst time
        
        ⇒ 프로세스가 CPU를 직접 점유해서 계산 작업을 수행하는 시간
        
        - CPU Bound process : CPU burst time이 긴 프로세스
        - I/O Bound process : CPU burst time은 짧기만, 자주 I/O를 요청하는 프로세스