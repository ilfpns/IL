> **Before…**
[[Embedded]]
---

### FSM

⇒ 시스템의 상태를 유한한 상태 집합과 전이 규칙으로 모델링해 제어 로직을 명확하게 표현하는 기법

### Cache Coherence

⇒ 멀티코어에서 각 코어의 캐시가 같은 메모리 값에 대해 일관성을 유지하도록 보장하는 매커니즘

> **MSI**

---

### FSM

- Modified
- Shared
- Invalid

라는 세 가지 State로 구성된다

### Case

1. Shared : Processor의 Cache가 다른 Processor의 Cache와 서로 공유된 상태
2. Modified : 원본 데이터와 값이 바뀌었기 때문에 Cache line을 바꿔야 함
3. Invalid : Processor의 Cache line이 변경되기 위하여, 같은 address를 가진 Cache line을 사용하지 못하게 함 (읽지 못하도록 무효화)

각 MSI는 서로 상호적 관계를 가진다

- 읽을 때 : S
- 수정할 때 : M
- 수정을 기다리고, 읽기를 종료할 떄 : I

### Problem

- Write하여 수정한 Cache Line을 다른 Processor가 access할 때
    
- Memory를 꼭 거쳐야 한다
    
- Why?
    
    Processor A는 새로운 데이터를 RAM에 적어야 한다 (+ 비용)
    
    이 후 바뀐 값을 Processor B가 RAM까지 가서 봐야한다 (+ 비용)
    
- Memory에서 data를 가져오는 것은 비용이 많이 든다
    
- 하여 불필요한 Memory Access를 하지 말아야 한다