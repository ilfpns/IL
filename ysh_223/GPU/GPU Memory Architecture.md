![[Pasted image 20260308204916.png]]

- GPU SM은 CPU Core와 같은 단위이다. SM은 Streaming Multiprocessor의 약자이다. 1개의 GPU에는 여러 개의 SM으로 구성되어 있다
- SM 내부에는 L1 Cache가 있고, SM 들은 L2 Cache를 통해 RAM (global memory)에 access 하는 구조이다

### 프로그램 실행 데이터 흐름

- Grid : 병렬 실행되는 명령의 모음, Kernel(=함수)당 1개의 grid를 실행, 여러 개의 block으로 구성
    
- Block : 병렬로 독립적으로 수행되는 단위, 여러 개의 thread로 구성
    
- Thraed : 명령을 실행하는 동작 단위이다, Cudar core 1개가 1개의 thraed를 수행한다
    
- Register : Thread마다 사용되는 메모리
    
- Shared Memory (=SMEM) : 데이터나 값을 미리 복사해 놓는 빠른 속도의 임시 저장소
    

### 병렬 단위

- Kernel : GPU에서 병렬로 실행하게 구현한 함수
    
    ⇒ 코드를 작성하는 함수 단위
    
    ⇒ Kernel은 SW 형태, Grid는 HW 형태로 구분한다
    
- Wrap : 동시에 동일 명령으로 실행되는 Thread의 묶음
    
    ⇒ 스케쥴링되는 최소 단위
    
    ⇒ 32개의 thread로 구성됨
    
    ⇒ 여러 개의 Wrap으로 1개의 thread block을 구성한다
    

[GPU (=CUDA) 관련 용어를 잘 정리해둔 블로그](https://xoft.tistory.com/75)