> **NAPI**
[[Linux & Kernel]] [[Network]]
---

⇒ Interrupt와 Polling 방식이 결합된 구조로 동작하는 모드

**사용 이유**

- 매우 잦은 인터럽트 처린는 비효율적이다
    
    ⇒ Rx Packet마다 ISR을 수행하게 된다면, 다른 HW 기기들이 Interrupt가 올바른 시간에 처리되지 못함
    
- Polling만 쓴다면, 이를 활용하지 않는 시간에는 자원을 낭비하게 된다
    

⇒ Network가 한가할 때 Interrupt로 동작하며 바쁠때는 Polling 방식으로 동작하는 모드이다

- 가장 젓 packet은 Interrupt로 처리함
- 그 후 Packet이 지속적이면, Polling 방식으로 Packet을 담는 Buffer를 확인함 (Polling 방식)
- 그 후 Packet이 느리게 들어오면, Interrupt 모드로 동작

### 상황

Packet이 Rx될 때, net_rx_action() 함수가 호출

### 문제

NIC : 하드웨어와 네트워크를 연결해주는 핵심 부품

RX/TX Ring Buffer가 작으면, 커널 소켓 버퍼가 커도 패킷이 중간에서 드롭됨

- Ring Buffer
    
    ⇒ NIC 안에 있는 하드웨어 큐
    
    ⇒ 패킷이 DMA로 가장 먼저 쌓이는 곳
    
    ⇒ 커널보다 앞단