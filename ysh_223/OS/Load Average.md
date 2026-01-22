> Load Average
[[OS]] [[Linux]]
---

⇒ 얼마나 많은 프로세스가 실행 중 혹은 실행 대기 중인가를 의미하는 수치

- Load Average가 높다면 : 실행 중이거나 실행 가능 상태(R) 또는 I/O 대기 상태(D) 인 프로세스가 많다
- Load Average가 낮다면 : 실행 중이거나 실행 가능 상태(R) 또는 I/O 대기 상태(D) 인 프로세스가 적다
- CPU Core가 많다면 Load Average가 낮다
    - 같은 작업량일 때
        - 코어 수 높음 ⇒ Load Average 적음

⇒ 어떤 값에 대해 절대적이지 않고 CPU의 Core에 기반함

⇒ Load Average는 CPU 사용률이 아니다

**CPU의 코어 수에 따른 해석**

- 4코어에서 Load 4 → 정상
- 4코어에서 Load 8 → 작업 밀림

**프로세스의 종류**

- CPU Bound : CPU자원을 많이 필요로 함
- I/O Bound : 많은 I/O 자원을 필요로 함

**Load Average를 확인하는 방법**

⇒ `uptime`

```cpp
1.20 0.95 0.80 2/312 1845
```

- 앞 3개 1/5/15분 Load
- 2/312 : 실행 프로세스의 수 / 전체 프로세스의 수

⇒ `vmstat`

![[Pasted image 20260122102555.png]]

다음과 같은 출력으로

- `r` : **Runnable** (실행 중 + 실행 가능 상태) → **Load 포함**
    
- `b` : **Uninterruptible Sleep (D state)**
    
    보통 디스크/블록 I/O 대기 → **Load 포함**