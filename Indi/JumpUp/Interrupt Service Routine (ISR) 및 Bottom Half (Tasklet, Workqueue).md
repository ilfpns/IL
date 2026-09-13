#### ISR

해당하는 interrupt를 처리해주는 Service Routine을 실행함을 의미한다

- interrupt는 block될 수 없다
- 짧게 작성되어야 한다
    - Top half
        - h/w에 관계된, 최대한 빨리 처리해야 하는 작업
    - Bottom half
        - ISR에서 처리할 필요가 없고, 시간이 오래 걸리는 작업
        - deferable work, deferred work로도 불린다
- context save & restore : interrupt가 발생하지 않은 것처럼 이전 프로세스를 재실행 해주어야 한다

#### IVT

interrupt와 해당 interrupt의 ISR의 매핑을 제공

- IVT는 h/w의 특정 주소에 위치한다
    - ARM의 경우 IVT는 0x00 부터 한 워드씩 할당한다
- IVT의 각 entry에 저장되는 값
    - 실행할 ISR의 주소, ISR로 jump하기 위한 명령어

#### Tasklet

Bottom half중 하나의 기법으로, 현재 interrupt에서 처리하지 않아도 되는 부분을 뜻한다

- softirq를 기반으로 동작하며, 다음과 같은 특징이 있다
    1. Atomic하다
        
        ⇒ 한번 시작하면 끝날 때까지 CPU를 사용한다. 중간에 Sleep, Wait으로 전환되지 않는다
        
    2. CPU고정
        
        ⇒ 스케줄링한 해당 CPU에서만 실행된다
        
    3. 직렬화
        
        ⇒ 여러 CPU에서 동시에 실행되지 않기에 Lock 처리를 줄여준다
        

**단점**

⇒ Non-blocking 구조로, 용량이 큰 파일이나 네트워크 응답을 기다릴 때 시스템이 멈출 수 있다

- Workqueue를 대체제로 사용함

#### Workqueue

⇒ 작업(일) 대기열을 관리하는 역할로, 스케줄러에서 실행 할 프로세스를 보관한다

- 일의 작업 단위를 work라고 하며, 이를 Workqueue에 저장한다
- Worker Thread : 나중에 실행해도 되는 작업
- 2개 이상의 Core에서, Core1은 여유롭고, Core2는 바쁠 때 Core1이 Core2의 작업을 가져가는 것을 Work Pulling 이라고 한다