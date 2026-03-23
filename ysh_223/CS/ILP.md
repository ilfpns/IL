### Pipeline
[[CS]]
⇒ Pipeline은 하나의 task에서 일정 부분이 끝났을 때 다음 task의 일정 부분이 연쇄적으로 시작되는 구조이다

- Pipeline에서 ideal pipeline CPI는 (stall이 발생하지 않고, 명령어가 실행되는 최소 클럭) 1이다.
- 하지만 hazard (Pipeline을 방해하는 요소) 때문에, 일반적으로 ideal CPI를 얻기 힘들다. 따라서 각종 hazard를 다 더하여 Pipeline CPI를 얻는다
- stall : pipeline이 멈추는 현상

⇒ Pipeline CPI = Ideal CPI + Structural Stalls + Data Stalls + Control Stalls

### ILP (**Instruction-Level Parallelism)**

보통 명령어는 순차적으로 나열되어 있다, 하지만 명령어를 순서대로 하나씩 실행하면 오버헤드가 커진다

- 서로 independent한 (독립적이라, 순서가 바뀌어도 영향이 없음)한 명령어를 parallle하게 실행시킬 수 있다면 이런 오버헤드를 줄일 수 있을 것이다
- 서로 중첩시키지 않더라도 pipeline과 같이 stage를 약간씩 중첩시켜 실해할 수 있다

⇒ ILP 방식이라고 부른다, 독립적인 명령어들을 최대한 찾아서 오버랩 실행을 하게 된다

### 방법

1 .static한 방법

- 컴파일러에 의존
- Itanium : 하드웨어는 단순하지만 컴파일러가 굉장히 복잡
    - 일반 CPU (dynamic = 프로그램 실행 중에 정해짐)
        
        - 하드웨어가 성능이 좋다
        - CPU가 일을 처리함
        
        ⇒ 실행과 동시에
        
        1. hazard 해결
        2. instruction 재배치
        3. batch prediction
        
        ⇒ Dynamic 방식
        
    - Itanium (static = 컴파일러가 프로그램 실행 전에 결정함)
        
        - 하드웨어가 단순하고, 컴파일러가 다 계산한다
        - CPU가 컴파일러가 지정한 행동만 실행
        
        ⇒ 실행 전에
        
        1. 명령어 동시 실행 선정
        2. dependency 확인
        3. stall 점검
        
        ⇒ 컴파일러가 결정
        
        ⇒ EPIC (Explicitly Parallel Instruction Computing)
        

### Loop-Level Parallelism

**용어 정리**

- Loop-level Parallelism : loop 내부에서 parallel하게 명령어를 돌리는 것
- Basic Block : Branch해서 들어오는 부분, ~ Branch 끝나고 나가는 부분

⇒ 어떻게 하면 Stall을 없애서 CPU가 계속 일할 수 있게 할까?

**해결책 : Loop Unrolling**

```c
// 1. 일반적인 루프 (잡일이 100번 발생)
for (int i = 0; i < 100; i++) {
    A[i] = B[i] + 1;
}

// 2. Loop Unrolling (4개씩 묶어서 펼침 -> 잡일이 25번으로 줄어듦!)
for (int i = 0; i < 100; i += 4) {
    A[i]   = B[i]   + 1;
    A[i+1] = B[i+1] + 1;
    A[i+2] = B[i+2] + 1;
    A[i+3] = B[i+3] + 1;
}
```

⇒ 기존 코드를 풀어서 반복적 연산을 줄임

- 점프(Branch) 횟수가 줄어 속도가 빠르다

**명령어 재배치**

CPU는 Pipeline 구조에서 앞의 계산 결과가 나와야만 뒤의 계산을 할 수 있는 상황에서 (의존성) 결과가 나올 때까지 멈춰서 값을 기다린다. (Stall)

그렇기 떄문에 그동안 상관없는 다른 일을 진행한다

**한계**

- 강제로 복사해서 풀어놓았기 떄문에 코드 크기가 증가한다
- 한번에 너무 많은 계산을 하므로 레지스터가 부족하다