---

### Background concept

- 강화학습
    
    강화는 시행착오를 통해 학습하는 방법 중 하나이다
    이러한 강화를 바탕으로 강화학습은 실수와 보상을 통해 학습을 하여 목표를 찾아가는 알고리즘이다. 기존 신경망들이 라벨이 있는 데이터를 통해 가중치와 편향을 학습하는 것과 비슷하게 보상이라는 개념을 사용하여 가중치와 편향을 학습하는 것이다.
    
    ⇒ 최적의 행동 양식 파악
    
    강화학습은 결정을 순차적으로 내려야하는 문제에 적용할 수 있다. 
    순차적인 문제 정의는 MDP를 사용한다
    
    문제 해결 flow
    
    1. 순차적 행동 문제를 MDP로 전환
    2. 가치함수를 계산
    3. 최적 가치함수와 최적 정책 파악
    
    Markov Process, Chain : MP는 이산 시간이 진행함에따라 상태가 확률적으로 변화하는 과정을 의미한다
    
    - State : 정적인 요소 + 동적인 요소
    - Action : 어떠한 상태에서 취할 수 있는 행동
    - Stochastic transition model : 어떤 state에서 특정 action으로 다음 상테에 도달할 확률
    - Reward : Agent가 학습할 수 있는 유일한 정보를 의미한다, 어떤 state에서 action을 하여 다음 state가 되고 이떄 받는 reward값은 다음 상태가 되는 것에 대한 reward이다
    - Policy : 순차적 action결정 문제(MDP)에서 구해야할 답을 의미한다. 모든 state에 대한 Agent가 어떠한 action을 해야 하는지 정해놓을 것
    
    ⇒ 최적의 정책(Optimal Policy)을 찾는 것
    
    - 가치함수
        
        MDP를 풀기 위해서는 가치함수를 정의해야한다.
        가치함수는 현재 state의 policy를 따라갔을 때 얻는 예측 reward의 총 합을 의미한다. 이때 현재 보상의 추세인 감가율을 고려하여 미래 보상을 예측한다. 
        Agent는 가치함수를 통해서 보장의 합을 최대로 한다는 목표에 얼마나 다가갔는지를 판단한다.
        
        - state
            
            state마다 어떤 action을 선택할지에 대한 policy가 있다.
            
            - policy가 state마다 따로 존재하는 것이 아니다
            - 하나의 policy함수가 모든 state에 대한 action을 지정한다
            
            ```c
            // Q-learning
                      State
                        ↓
                  ┌─────────────┐
                  │ Q(s, action)│
                  └─────────────┘
                   ↙     ↓     ↘
                 left  forward right
                   ↓     ↓     ↓
                  0.2   0.8   0.1
            ```
            
            policy는 Q값을 보고 가장 최적해를 찾는다
            
- PPO
    
    Proximal Policy Optimization
    
    강화학습은 기본적으로
    
    state → policy → action → reward → policy 업데이트 방식이다
    
    문제는 policy를 한 번에 너무 많이 바꾸면 학습이 망가질 수 있다.
    policy 값이 크게 바뀌면 학습에 문제가 생길 수 있음
    
    PPO ⇒ 좋은 방향으로 업데이트하되, 한 번에 너무 많이 바뀌는 것을 방지한다
    다양한 수식을 통해서 좋은 action의 확률은 높이고, 나쁜 action의 확률은 낮추되, policy를 한 번에 너무 크게 바꾸지 않는다
    
    - 자세한 PPO
        
        PPO는 Robotics 학습 분야에서 OpenAI가 제시한 새로운 학습 방법이다 
        근사 정책 최적화 (PPO)에 대해 알아보자
        
        - PPO 등장 이전
            - deep Q-learning : 많은 간단한 문제에서 실패
            - 바닐라 정책 경사 : 낮은 데이터 효율성 및 안정성
            - TPRO : 복잡한 구현
        
        PPO는 샘플 복잡도, 단순성, 실제 소요 시간 사이에서 유리한 균형을 이룬다.
        
        - Sample complexity (샘플 복잡도)
            
            ML 모델이 원하는 수준으로 학습하기 위해 필요한 데이터 샘플의 양
            
        
        - 확률 비율 (Policy의 큰 가중 변화를 막음)
            
            확률을 서로 비교한 비율을 의미한다. 
            
            ⇒ 새 policy가 이전 policy 대비 얼마나 변화했는지를 나타내는 비율이다
            
            ⇒ 1은 변화가 없음을 의미한다
            
        
- **Isaac Sim vs Isaac Lab**
    - Sim : 로봇이 가상 시뮬레이터
    - Lab : 로봇 학습 프레임워크
    
    ```c
                 Isaac Lab
         ┌─────────────────────┐
         │ Task / Environment  │
         │ Observation         │
         │ Reward              │
         │ Domain Randomization│
         │ RL / IL Training    │
         └──────────┬──────────┘
                    ↓
              Isaac Sim
         ┌─────────────────────┐
         │ Physics             │
         │ Rendering           │
         │ Robot / Objects     │
         │ Sensors             │
         │ Simulation World    │
         └─────────────────────┘
    ```
    
    Sim에서 가상 환경을 조성한다, Lab에서 State, Policy, Action 등을 구성한다
    
- RL
    
    
    | 개념 | 이해할 것 |
    | --- | --- |
    | **State / Observation** | 에이전트가 현재 무엇을 보고 있는가 |
    | **Action** | 에이전트가 할 수 있는 행동 |
    | **Reward** | 행동이 얼마나 좋았는가 |
    | **Policy** | State → Action을 결정하는 방법 |
    | **Value / Q-value** | 상태/행동이 앞으로 얼마나 좋을지 |
    | **Episode** | 시작 → 행동 반복 → 종료까지 한 번의 경험 |
    - ~~MDP 같은 내용은 강화학습에서 아주 간소하게 확인할 수 있다…~~

