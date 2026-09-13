> **프로세싱 기법**
[[OS]]
---

- Asymmetric multiprocessing
    
    ⇒ 하나의 Master 프로세스가 다른 프로세스들을 관리하는 형태
    
- Symmetric multiprocessing
    
    ⇒ 각각 프로세스가 동등한 권한으로 프로세싱 하는 형태
    
    ⇒ 각자 스케줄링을 진행한다
    
    ⇒ 하나의 ready queue에 저장되거나, 각 프로세서의 private queue에 저장된다
    

### 메모리 접근 구분

1. UMA
    - 하나의 공유된 메모리에 여러개의 프로세스가 할당
    - 하나의 메모리 공유 (버스 공유)
    - 어느 메모리를 참조하던 같은 속도를 보임
2. NUMA

> CPU affinity

---

⇒ 특정 Core를 지정하여 task를 실행하도록 설계된 Concept

- Soft Affinity

⇒ 같은 프로세서에서 돌 수 있도록 priority를 높여주는 등 노력은 하지만, 보장은 하지 않음

- hard Anffinity

⇒ 특정 프로세스에서만 돌도록 완전히 보장하는 경우

### Load Balancing

⇒ 특정 Core가 너무 많은 프로세스를 돌리면, 스케줄러를 적절히 분배해주는 행위

- Push migration : Core가 일을 넘기는 것
- Pull migration : Core가 일을 받아오는 것