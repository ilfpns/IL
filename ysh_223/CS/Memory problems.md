Memory (,bus)와 관련되어 생길 수 있는 문재들에 대해 정리한다.

---

#### Bus contention

2개 이상의 Master가 동시에 하나의 공유 버스를 이용하려 할 떄 발생하는 충돌이다.

- DMA와 CPU가 동시에 한 버스를 사용하려 시도 한다면? → 문제 발생

이는 

- 지연 시간 증가
- 데이터 손실 및 충돌
- 전력 소모

등의 문제를 발생시킨다.

이는 Bus Arbiter라는 방법으로 해결 할 수 있다.

| **방식** | **설명** |
| --- | --- |
| **Daisy Chain** | 장치들을 직렬로 연결하여 우선순위가 높은 장치부터 차례대로 사용권을 부여한다. |
| **Polling** | 중재기가 각 장치에게 "사용할 계획이 있느냐"고 순서대로 물어보는 방식이다. |
| **Independent Request** | 각 장치가 중재기에게 개별적인 요청 선을 가진다. 가장 빠르지만 회로가 복잡하다. |

```c
#include <stdio.h>
#include <pthread.h>

int shared_bus = 0; // 공유 자원 (Bus 역할)

void* access_bus(void* arg) {
    for (int i = 0; i < 1000000; i++) {
        shared_bus++; 
    }
    return NULL;
}

int main() {
    pthread_t master1, master2;
	
		// 동시에 한 값에 접근해 증감 연산 실행
    pthread_create(&master1, NULL, access_bus, NULL);
    pthread_create(&master2, NULL, access_bus, NULL);
	
		// 끝날때까지 main은 기다리기
    pthread_join(master1, NULL);
    pthread_join(master2, NULL);

    printf("최종 버스 데이터: %d\n", shared_bus);
    return 0;
}
```

#### Race contention

Bus contention의 소프트웨어 느낌으로, 스레드가 공유 메모리를 동시에 접근하는 문제이다.

- 두 개 이상의 프로세스나 스레드가 공유 자원에 동시에 접근하여, 실행 순서에 따라 결과값이 달라진다.

이는 다음과 같은 특징이 있다

- 비결정적
    - 실행할 때마다 결과가 달라진다
- 디버깅이 어렵다
    - 문제가 발생했다가 안 했다가 하기에, Heisenbug라고도 부른다
- 보안 취약
    - 경쟁 타이밍에 권한을 뺏길 수 있다

⇒ Mutex를 사용하여 해결 가능하다

```c
pthraed_mutex_lock(&lock);
shared_bus++;
pthread_mutex_unlock(&lock);
```

- Buffer Overflow
- Stack Overflow
- Null Pointer Dereference
- Dangling Pointer
- Memory Corruption
- Alignment Fault
- Hard Fault (Bus/MemManage 포함)
- Cache Coherency Issue
