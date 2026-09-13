## SIMD / SIMT
[[GPU]]
### Single Instruction Multiple Data

⇒ 하나의 명령어로 여러 데이터를 동시에 처리

- CPU 벡터 연산
    - SSE
    - AVX
    - NEON

⇒ 데이터가 완전히 동일한 연산일 때 사용함

|분야|예|
|---|---|
|영상 처리|pixel 연산|
|신호 처리|FIR filter|
|과학 계산|matrix|
|게임 엔진|physics|
|AI inference|vector math|

즉, 대량의 데이터이면서 동일한 연산을 할 때 사용

**왜 사용할까?**

CPU는

- Core의 개수가 적음
    
    ⇒ 한 Core가 여러 데이터를 처리
    

**장점**

- 높은 연산 효율
- 하드웨어 효율
- 메모리 지역성이 좋음

**단점**

- branch에 약함
    
- irregular data 처리 어려움
    
    ⇒ 데이터 접근 패턴이 규칙적이지 않은 데이터는 처리가 어렵다
    
    ⇒ 연속된 값에 더 빠른 연산이 따름 (Memory coalescing)
    
- 프로그래밍 어려움
    

### Single Instruction Multiple Thread

⇒ 하나의 명령을 여러 Thread가 동시에 실행

GPU

- CUDA
- OpenCL
- Metal GPU

|분야|예|
|---|---|
|AI|CNN|
|graphics|shader|
|HPC|simulation|
|data processing|big data|
|physics|particle|

⇒ 데이터가 병렬적이며, thread가 병렬적일 때 사용됨

**왜 사용할까?**

GPU는

- Core의 갯수가 많다
    
    ⇒ 수천 thread 실행 가능
    
- 실행 단위 : thread
    
- wrap : 32Thread
    
- 명령 : 동일
    
- control Flow : thread마다 가능
    

**장점**

- 병렬성
- Latency hiding
- thread abstraction

**단점**

- **w**arp divergence 문제
    
    ⇒ wrap내부 thread가 다른 코드 경로를 실행하는 상황
    
- memory access 패턴 민감
    
    ⇒ bandwidth 낭비로 메모리 접근 패턴에 민감하다
    
- occupancy 관리 필요
    
    ⇒ 활성된 wrap 비율 관리
    
    ⇒ Latency hiding 문제로 발생함