**CPI stall = CPI_structural hazard + CPI_data hazard + CPI_control hazard**

⇒ SW hazard 해결 말고, 하드웨어가 어떻게 동적으로 instruction을 re-scheduling하여 hazard를 줄일 수 있을까?

### Dynamic Scheduling

data flow와 exception behavior를 유지하는 범위 내에서 하드웨어 적으로 instruction을 re-scheduling하며 stall을 줄이는 방법

⇒ Stall이 발생했을 때에도 지연 없이 CPU 처법

- Data flow : A가 생상한 데이터를 B가 소모함을 알리는 흐름
    
    ⇒ 관계가 없는 명령어 먼저 실행 (ILP)
    
- Exception Behavior : 인터럽트 발생 시 정확한 원인을 파악하는 것
    
    ⇒ 확정된 실행 순서는 다시 스케줄링 되지 않는다
    
- compile time에서 알 수 없는 instruction dependency에 대한 처리를 할 수 있다
    
- 한 파이프라인에서 컴파일된 코드가 다른 파이프라인에서 효율적으로 실행될 수 있게 한다
    
- 컴파일러가 단순해진다
    

### Out-of-order Processor Pipeline이란?

⇒ 특정 작업이 지연됨에 따라 낭비될 수 있는 명령 사이클을 이용하는 방식

- 명령어를 순서대로 처리하지 않는다

→ 명령어 수준에서 병렬성을 찾아서 이를 비순차적으로 해결한다

비순차적 처리를 위해서는 다음과 같은 기술이 필요하다

- 명령어 윈도우
    
- 가짜 의존성 제거
    
- 동적 명령어 스케줄링 (Dynamic Instruction Scheduling)
    
- 순차적 완료
    
- 그렇다면 연관관계가 없는 명령어를 비순차적 (병렬적)으로 처리하는 것이라면 무슨 차이일까?
    
    ILP
    
    - 개념 : “명령어들을 얼마나 동시에 실행할 수 있는가?”라는 잠재력 혹은 상태를 의미한다
    - 특징 : “서료 연관 없는 명령어들이 얼마나 많은가?”가 ILP의 양을 결정
    - 예시 : `A = 1 + 1` 과 `B = 2 + 2` 는 서로 독립적으로, ILP가 높다
    
    OoO
    
    - 개념 : ILP를 실제로 뽑아먹기 위해 CPU가 “순서를 섞는 행위”이다
    - 특징 : 명령어의 순차적 실행보다 비순차적 실행이 더 나을 것 같을 때, 비순차적 진행을 하드웨어적으로 하는 것
    - 핵심 : Dynamic Scheduling이 OoO를 가능하게 만드는 이론
    
    |**구분**|**방법 (수단)**|**누가 하는가?**|**특징**|
    |---|---|---|---|
    |**정적 (Static)**|**VLIW / 컴파일러 최적화**|소프트웨어 (컴파일러)|코드를 짤 때 이미 병렬로 묶어버림. HW는 시키는 대로만 한다|
    |**동적 (Dynamic)**|**OoO (Out-of-Order)**|**하드웨어 (CPU)**|실행 중에 CPU가 실시간으로 순서를 바꾼다|