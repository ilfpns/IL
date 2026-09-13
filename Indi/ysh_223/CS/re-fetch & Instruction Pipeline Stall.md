> **Re-fetch**
[[CS]] 
---

⇒ 이미 가져왔던 명령어을 버리고 다시 가져오는 것

**상황**

- 분기 예측 실패
    
- 예측한 경로의 명령어를 미리 fetch했는데 틀림
    
    ⇒ 올바른 PC 경로로 re-fetch
    

⇒ 잘못 가져온 것을 다시 두고, 올바른 것을 가져옴

> **Pipelining Hazard**

---

Hazard : 특정한 이유로 다음 clock cycle에 명령어가 실행되지 않는 상황

**3가지 원인**

1. Structural Hazard
2. Data Hazard
3. Control Hazard

⇒ 모든 Hazard는 waiting으로 해결된다

## 1. Structural Hazard

⇒ 자원은 하나인데 여러 명령이 동시에 수행되려고 할 때 발생

- EX) 한 메모리에 여러 자원이 접근

⇒ hardward resource를 추가

- 각 용도에 맞는 메모리만큼 n개의 datapath추가

## 2. Data Hazard

⇒ pipeline 명령어가 끝나지 않은 register에 접근하여 명령어를 실행하려고 할 때

## 3. Control Hazard

⇒ deq혹은 J 명령어 등 분기 명령어를 통해 명령어를 건너뛰면, PC 값이 변하고 중간에 있는 명령어를 실행하지 않음

⇒ 예상 밖의 일로 못 가서 멈춤