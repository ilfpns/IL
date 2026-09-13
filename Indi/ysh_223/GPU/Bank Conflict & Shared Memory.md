## Shared Memory
[[GPU]]
⇒ 같은 block에 있는 thread들이 같이 쓰는 GPU on-chip 메모리

```c
Grid
 └ Block
    └ Warp
       └ Thread
```

**특징**

- SRAM
- 빠르다
- block 내에서만 접근이 가능하다

```c
Register        (가장 빠름)
Shared memory
L1 / L2 cache
Global memory   (VRAM, 느림)
```

|메모리|latency|
|---|---|
|Register|~1 cycle|
|Shared memory|~10 cycle|
|Global memory|~400~800 cycle|

### 목표

⇒ Global Memory에 access하는 횟수를 줄이자

방법 → Tiling 방식 사용

### SIMT의 문제

하나의 명령어로 여러 스레드를 관리하면 생기는 문제

⇒ Memory access

- GPU에서는 동시에 여러개의 데이터를 처리해야하기 때문에, 동시에 여러개의 데이터 access를 허용한다
    - GPU는 shared memory를 각 wrap마다 일정 갯수의 memory bank로 나누어 둠
    - 각 bank는 bank단위로 접근할 수 있다

### Bank Conflict

⇒ 프로그래밍 잘못으로 동시에 서로 다른 thread가 특정 bank를 access할 때 발생하는 문제

결과

⇒ 하나의 자원에 대한 순차적 접근이 되므로, 병렬적 처리를 의도대로 수행하지 못함