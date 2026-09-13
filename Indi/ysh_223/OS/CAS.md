### Compare and swap이란
[[OS]]
⇒ 동시성 암고리즘 설계 기술

⇒ 각 Core가 가지는 개인 메모리 값과 RAM의 값이 다른 경우 해결 알고리즘

### 등장 배경

- 멀티 스레드/코어 환경에서 각 CPU는 메인 메모리에서 변수 값을 참조하지 않음
    
    - 메인 메모리 값
    - CPU 캐시 값
    
    두 값이 다를 수 있음 (가시성 문제)
    

### CAS

⇒ 현재 스레드에 저장된 값과 RAM에 값을 비교하여 일치하는 경우 새 값으로 바꾸고, 일치 하지 않는 다면 재시도를 하여 CPU 캐시에서 잘못된 값을 참조하는 가시성 문제를 해결함

- 가시성 vs 원자성
    - 가시성(Visibility): A가 값을 바꿨을 때, B가 그 바뀐 값을 즉시 볼 수 있는가? (캐시 일관성, volatile 등과 관련)
    - 원자성(Atomicity): 내가 값을 읽고(Read), 수정하고(Modify), 쓰는(Write) 이 3단계 과정 사이에 누가 끼어들지 못하게 할 수 있는가?

### CAS 적용

- Mutex를 사용하면 되는 것 아닌가?
    - 실시간성이 깨짐
    - 오버헤드 위험
- CAS는 뭐가 나은가?
    - 값이 변하지 않을 때만 업데이트
    - 실패 시 재시도 (낙관적 동기화)

```c
#include <stdatomic.h>

void safe_increment(atomic_int *ptr) {
    int old_val, new_val;
    do {
        old_val = atomic_load(ptr);   // 현재 값 읽기
        new_val = old_val + 1;        // 새 값 계산
    } while (!atomic_compare_exchange_strong(ptr, &old_val, new_val)); 
}

// new_val과 old_val
// _val에 1을 더하는 로직에서 두 _val은 항상 값이 같음
```