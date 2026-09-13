> **Worst Case Execution Time**
[[OS]]
---

⇒ 가장 긴 코드 실행 시간을 가진다 (=최악 실행시간)

⇒ 실시관 환경에서 타이밍 문제를 일으킬 수 있다

이러한 문제를 해결하기 위해서 WCET를 관리해야 한다.

- WCET를 관리한다는 것은 문제 유발을 야기하는 WCET를 식별하고,
    
- UNIT의 제한 시간 내에서 SW가 실행될 수 있도록 제품을 개발하는 것이다
    
- WCRT
    
    ⇒ 최악 응답시간을 의미한다
    
    ⇒ 해당 태스크가 언제 끝나는지를 본다
    
    ```cpp
    [READY] ──(대기)──▶ [RUN] ──(선점)──▶ [RUN] ─▶ [DONE]
    // 전체가 WCRT
    ```
    

> 코드상의 WCET

---

⇒ 이 함수/태스크가 CPU를 얼마나 오래 사용하는가

```cpp
void control_task(void)
{
    for (int i = 0; i < 100; i++) {  
        calc();
    }
}
```

WCET = 이 루프가 100번 돌 때의 최대 실행 시간

⇒ 최악의 경우에 코드가 걸리도록 WCET가 설명 가능하게 짜야 한다

```cpp
while (sensor_ready() == 0) {}
```

- 루프가 최대 몇 번 돌아가는가? → 모름
- 최악에 몇 ms걸리나? → 모름
- 인터럽트를 막으면 → 무한루프

즉

- WCET 정의 불가
- 정의 불가는 실시간 시스템에서 의미가 없다

```cpp
for (int retry = 0; retry < 50; retry++) {
    if (sensor_ready()) break;
}
```

- 루프가 최대 몇 번 돌아가는가? → 50번
    
- 최악 실행 경로 → 50번 다 돌아가는 경우
    
- WCET →
    

$$ sensor_ready() + 비교 + 분기 * 50 $$