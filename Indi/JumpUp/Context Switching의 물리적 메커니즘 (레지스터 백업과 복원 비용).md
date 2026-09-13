#### Context

⇒ Process가 특정 작업을 수행하기 위해 현재 알고 있어야 하는 정보의 집합

- Process의 경우 현재 Process가 중단 되었을 때, 중단된 시점 부터 다시 Proces를 실행하기 위한 정보를 Context라고 부른다
- Process Context 정보는 PCB 구조체에 저장된다

#### Context Switching

⇒ 작업의 주체가 현재 Context를 중단하고, 다른 Context를 실행하는 것

#### 물리적 레지스터 백업과 복원

컨텍스트 스위칭은 주로 인터럽트나 시스템 콜이 발생하여 OS의 커널이 개입할 떄 일어난다

- Flow
    1. 인터럽트 발생 : 모드 전환
    2. 현재 상태 저장 (PCB or TCB 에 저장) : 현재 Process 저장
    3. 스케줄러 개입 : 다음 실행할 Process 선택
    4. 새로운 상태 복원 : 스케줄러에서 선택된 Process의 PCB에서 이전에 저장해둔 레지스터 값들을 CPU의 실제 레지스터로 불러온다
    5. 실행 재개 : 모드 전화 후 복원된 PC가 가리키는 지점부터 Process의 실행을 재개한다

#### Context Switching 비용

- 직접적 비용
    
    CPU 사이클을 소모하는 순수 오버헤드
    
    - 레지스터 복사 연산
    - 확장된 레지스터 부담
- 간접적 비용
    
    현재 레지스터 문맥을 교체함으로써 발생하는 아키텍쳐 수준의 성능 저하
    
    - TLB 캐시 Flush
    - 캐시 메모리 오염
    - CPU Pipeline Flush