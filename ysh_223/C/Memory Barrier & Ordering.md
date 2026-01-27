> **Memory Barrier**
[[C]]
---

⇒ 컴파일러와 CPU들 모두에게 명령어 재정렬을 금지하는 지점이다

CPU는 속도를 최적화 하기 위해 아래를 마음대로 재배치한다

- Load/Store 순서 변경
- Prefetch
- Store buffer 사용

```c
int data = 1;
int flag = 0;
```

⇒ 두 변수가 순서와 다르게 flag→data 순으로 정렬될 수 있음

```c
// C언어
data = 123;
__sync_synchronize();   
flag = 1;

// ARM, ESP 등
data = 123;
__asm__ volatile("dmb sy" ::: "memory");
flag = 1;
```

⇒ 다음과 같이 메모리 배리어를 직접 넣어 방지할 수 있다

> **Memory Ordering**

---

⇒ 멀티코어/멀티스레드 환경에서 메모리 접근이 다른 스레드에 어떤 순서로 확인 되느냐를 정의한 규칙이다

**필요**

⇒ 컴파일러는 성능 때문에 명령어를 재정렬 함

실제로 코드 순서는 실제 관측 순서와 맞지 않을 수 있다

**차이**

- 배리어는 수단
- 오더링은 규칙

수단에는

```c
atomic_store_explicit(&data, 123, memory_order_relaxed);
atomic_store_explicit(&flag, 1, memory_order_release);

```

다음과 같은 방법도 있다