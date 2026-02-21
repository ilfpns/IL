> **Symmetric multiprocessing**
[[OS]]
---

⇒ 모든 CPU 코어가 평등하게 동작하는 다중 처리 시스템

### 핵심

- 완전한 평등
    - 같은 권한
- 하나의 메모리 공유
- 하나의 운영체제

⇒ 모두가 평등하고 같은 메모리를 공유함

### 문제

⇒ 코어 4개가 동시에 런큐에 접근

다음과 같은 상황 발생

- 하나의 코어는 프로세스를 찾음
- 다른 코어 병목
- DeadLock & spin lock

### 해결 (percpu)

⇒ 각 Core 들의 개수만큼 Runqueue를 쪼개어 가짐

1. Runqueue를 쪼개어 가짐
2. 한 Core의 Runqueue에 프로세스 포화
3. 다른 Core가 이를 점검
4. Runqueue가 여유롭다면, 포화 Runqueue에서 프로세스를 가져옴
5. 작업량의 평등성음 맞춤

⇒ Load Balancing