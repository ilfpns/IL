## CUDA 실행 구조 & 스케줄링 [[GPU]]

- Thread → Wrap → Block → SM

|단위|설명|
|---|---|
|Thread|가장 작은 실행 단위|
|Warp|**32 threads 묶음 (실제 실행 단위)**|
|Block|여러 warp|
|SM|warp들을 실행하는 GPU 코어|

GPU는 Wrap 단위로 스케줄링을 진행한다

Wrap Scheduler가 SM안에서 어떤 Wrap을 실행할지 선택한다

**왜 사용할까?**

⇒ GPU의 핵심 문제인 Memory Latency를 해결하기 위해서 사용된다

- 1개의 SM안에는 4Wrap schedulers가 있다
- 각 Scheduler는 ready wrap 중 하나를 골라서 실행

**우선순위 기준**

- Wrap Scheduler는 비선점이며, 우선순위를 기반으로 하지 않고, ready wrap 중 하나를 실행한다

⇒ Ready Wrap?

- 메모리 대기가 없음
- dependency 없음
- pipeline stall 없음