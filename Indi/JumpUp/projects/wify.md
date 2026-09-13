### 프로토콜 설명

발행-구독 기반의 송수신 프로토콜이다.

TCP/IP 프로토콜 위에서 동작하지만 동시에 굉장히 가볍다

MQTT는 Bluetooth나 Zigbee처럼 별도의 모듈로 대역폭을 갖는 통신 규약이 아닌, Wi-Fi와 같이 인터넷을 통해 TCP/IP 기반의 메시지 송수신 방식이다.
하지만 어느 기술이던 Trade-off가 있는 법. 이는 QoS (quality of service)에 제약이 있다.

### 특징

- 연결지향적
    - 한 번 연결 후 client는 socket을 유지하여 TCP/IP를 명시적으로 끊기 전까지 연결 상태 유지
    - Live라는 하트비트와 Topic에 발행되는 메시지를 통해 연결 유지 및 송수신
    - 연결이 끊어져도 재접속 가능
- 브로커 통신
    - 발행-구독 패턴은 오로지 브로커를 통해서만 통신할 수 있게 개발됨
    - Topic에 메시지를 발행하면 Topic을 구독하는 client에게 메시지 발행 가능
    - 1:N 통신 방식
- QoS
    - 0 : 메시지 1회만 전송 (수신 보장 x)
    - 1 : 메세지 N회 전송 (수신 실패 시 재송신, handshake과정을 엄밀히 살피지 않으므로 중복 전송 위험)
    - 2 : 정확히 한 번 수신 보장
- 메시지 유형
    - 연결
    - 끊기
    - 방행

### 3키워드

1. 발행/구독
    1. 발행자와 구독자가 서로 모르는 상황에서 Topic만으로 연결됨
        1. 발행 : 송신 (클라)
        2. 구독 : 수신 (서버)
2. 경량
    1. 헤더가 매우 작기 때문에 저전력
        
        ⇒ 헤더를 2바이트로 줄이자 (개발 배경에 따른 철학임)
        
        - 첫 바이트 : 메지지 종료 (구독 or 발행), Flasg (QoS, etc)
        - 둘쨰 바이트 : 길이
        
        ⇒ 이진 포맷 
        
        - HTTP처럼 텍스트 헤더가 아닌 비트/바이트 이진 인코딩을 통해 감량
        
        ⇒ Broker 
        
        - 경로/메타데이터 없이 Topic으로 브로커가 라우팅을 대신 함
3. 브로커 기반
    1. 모든 통신에 대해 브로커를 거침
        - 연결 관리 : 모든 기기에 대한 관리
        - 구독 관리 : 누가 어떤 토픽을 구독하는지 테이블 보유
        - 라우팅 : 토픽에 따라 구독자 명단에 구독자에게 전달
        - QoS 보장 : QoS/ACK 등을 브로커가 담당
        - Retained : Topic의 마지막 메지시를 저장했다가 새로운 구독자에게 최신 정보 제공
        - LWT : 기기 비정상 종료에 대해 브로커가 연결 비정상 종료 메시지 발행

### 흐름

Topic이 존재한다 송신자는 Topic에 data를 송신하고 (발행), Topic을 구독하는 수신자 (구독)는 data를 받는다.

### Topic 구조

메시지의 주소를 의미한다.

⇒ 메시지 발행/구독은 채널 단위로 일어난다. 이 채널 단위를 Topic이라고 하며 Topic은 ‘/’로 구분되는 계층 구조이다.

`home/livingroom/temperature` 처럼 계층 구조이다. 

와일드카드로 묶어 구독 가능가능하다

- `+`(한 단계)
- `#`(그 아래 전부)

⇒  예: `home/+/temperature`는 모든 방 온도, `home/#`는 home 아래 전부

**Retained Me**ssage & LWT

- Retained: 브로커가 토픽의 마지막 메시지를 저장 → 새로 구독한 기기가 즉시 최신 상태를 받음
    - Broker : 구독자/발행자 사이에서 구독 명단 관리 + 메시지 전달 등의 역할을 한다
    - 발행자 → Broker (전달) → 구독자
- LWT (Last Will and Testament): 기기가 비정상적으로 끊기면 브로커가 대신 기기 문제 발생 메시지를 발행

### Point

페이로드는 Json으로 짜여짐, 이때 토픽 구조 설계가 정확해야 확장성이 뛰어남
⇒ BLE에서 GATT 트리를 설계하는 것과 비슷하다

### Version

- MQTT
- MQTT5
    - MQTT 업그레이드 버전
        - 에러 코드 이유 제공
        - 메타데이터 추가 확장 가능
        - 메시지 만료 기능 설정 가능
        - 공유 구독 기능
        - 세션 말료
        - 토픽 병칭 (긴 토픽을 짥게)

- Background
    - ble : 서버를 써야해서 변경
    - WS : 확장성을 위해서 mqtt로 변경
- **Dev**
    
    https://github.com/espressif/esp-mqtt
    
    ESP ↔ 서버 서로 행동을 제어하기 때문에 두 단 모두 구독/발행을 구현해야 한다
    
    - MQTT connect Broker addr : "mqtt://192.168.0.10:1883”
    - wifi sta, mqtt 모두 channel 0으로 설정
    - channel 개념
        
        무선 통신이 사용하는 주파수 대역의 구역
        
        - 2.4GHz 주파수는 대한민국 기준 13개로 쪼개어진 channel에 나눠짐
            - 통신을 위해서 같은 channel에 위치해야 한다
            - AP, STA는 한 번에 한 channel만 사용
            - STA는 AP의 channel을 따라감 ( → STA에서는 0으로 설정)
    
    ---
    
    - ESP32 → Broker → Server
        
        #### Status
        
        | Topic | Func | Data | Flow |
        | --- | --- | --- | --- |
        | "wify/device01/status”         | 현재 기기 상태 | “online” / “offline” | Client → Server |
        | "wify/device01/heartbeat” | 기기 작동 heartbeat 전송  | payload | Client → Server |
        
        #### Restroom
        
        | Topic | Func | Data | Flow |
        | --- | --- | --- | --- |
        | "wify/device01/restroom” | 화장실 사람 감지 (PIR 깨어남) | ACT | Client → Server |
        | "wify/device01/restroom” | 화장실 사용 없음 (sleep) | DEACT | Client → Server |
        | "wify/device01/restroom” | PIR 첫 연결 (init) | LOAD | Client → Server |
        
        #### Network
        
        | Topic | Func | Data | Flow |
        | --- | --- | --- | --- |
        | "wify/device01/nownetwork” | 현재 네트워크 id, passw | “online” / “offline” | Client → Server |
        | "wify/device01/nownetwork/status” | 현재 네트워크 상태 | “connect” / “disconnect” | Client → Server |
        | "wify/device01/nownetwork/again” | 잘못된 id/passwd로 다시 재 입력 필요 메시지 | AGAIN | client → server |
        
        ### AI
        
        | Topic | Func | Data | Flow |
        | --- | --- | --- | --- |
        | "wify/device01/AI” | AI 현재 판단  값 : 위험 | DAN | Client → Server |
        | "wify/device01/AI” | AI 현재 판단  값 : 평범 | NOR | Client → Server |
        | "wify/device01/AI” | AI 현재 판단  값 : 주의 | WARN | Client → Server |
        
        ### Baseline
        
        | Topic | Func | Data | Flow |
        | --- | --- | --- | --- |
        | “wify/device01/baseline/status” | Baseline 상태 알림 | BASELINEDONE | Client → Server |
        
        ESP ↔ APP
        
    - Server → Broker → ESP32
        
        #### Baseline
        
        | Topic | Func | Data | Flow |
        | --- | --- | --- | --- |
        | “wifi/device01/baseline/cmd” | Baseline 재구축 명령 | BASELINEREBUILD | Server → Client |
        
        #### Network
        
        | Topic | Func | Data | Flow |
        | --- | --- | --- | --- |
        | "wify/device01/edit/nownetwork” | 새로운 id/passwd 입력 | “id:%s/passwd:%s“ | Server → Client |
        | "wify/device01/edit/editnetwork” | 새로운 id/passwd 적 | NEWNETWORKEDIT | Server → Client |
    - Problem
        - 사담
            
            방금 아영쌤이랑 상담하고 왔는데,,, 기술 블로그를 쓰라고 하신다… 쓸만한 주제가 없던 참에 주제가 생겼다.
            
            기존에는 ble → ws → mqtt 스택 변경 이유를 쓰려 했는데, 하나 더 생긴 거 같다
            
        - Critical Section 문제
            
            문제 : Ciritical section
            
            프로젝트에 대해 claude가 이렇게 분석해줬다.
            
            !image.png
            
            서로 다른 파일에 위치한 두 함수와 안전하지 않은 bool 방식 안전 점검만으로 g_baseline에 대한 공유 접근 문제를 해결할 수 없다는 문제가 터졌다.
            
            여기서 대응책은 크게 2가지이다.
            
            1. task 꼬임이 없도록 하나를 삭제 (개소리)
            2. 상호배제 기법 사용
            
            이 상황에서는 2번이 바람직하다. 상호배제 기법은 spin lock, TAS, Mutex, Semaphore 등이 있지만.
            
            여러 task가 하나의 자원에 대해 접근하는 것을 원치 않으므로 mutex 사용이 옳아보인다.
            
            !image.png
            
            이 부분이 문제다. 현재는 ready를 통해서 아주 간단하게 상호 배제를 사용하고 있다. 하지만 ready를 체크하는 함수가 2, 3 번에서 g_baseline에 접근한다면? 아직 4번 라인 실행되지 않기 때문에 baseline_ready → apply 식으로 flow가 바뀌며 문제가 발생한다.
            
            (Mutex 선정 이유 더 정리)
            
            즉 g_baseline은 상호배제 처리가 필요함 Critical section이다.
            
            !image.png
            
            이렇게 단순하게 mutex를 걸 수도 있겠다.
            
            나도 이런 방식을 생각했다. 대중적인 상호배제 기법이니까.. 
            
            하지만 실제로는 
            
            !image.png
            
            그저 flag를 하나 더 만들어서 관리하는 방법이 훨씬 효율적이었다.
            
            mqtt만 받아두고 flag를 true로 맞춘 후 다음 csi를 받을 때 baseline을 초기화 하면 되는 문제였다.
            
            괜히 동시에 자원에 접근할 필요가 근본적으로 없어졌다.
            
        - CSI 흐름
            
            #### CSI
            
            ⇒ wifi 패킷이 도착하면 hw가 이 신호의 각 부반송파에서 얼마나 변형되었는지를 측정함
            
            ⇒ 부반송파 (subcarrier)
            
            - wifi는 한 주파수로 통신하지 않음, 채널 대역폭 안에서 수십개의 대역으로 쪼깨어저 통신함. 이 쪼갠 조각 하나가 subcarrier이다. (OFDM 방식)
            - 의문
                
                왜 전처리기에 52개의 channel로 설정하였나
                
                ⇒ 20MHz에서 채널의 데이터용 subcarrier가 52개이기 떄문
                
            - 복소수 subcarrier
                - 위상
                - 크기
                
                두 데이터가 있음, 우리는 크기만 사용한다.
                
                위상은 송수신 기기의 발진기가 안 맞아서 생기는 노이즈가 너무 많음 → 버림
                
            1. csi_rx_cb(info)
                1. csi 정보
                2. len
                
                두 정보를 포함하는 구조체를 가짐
                
                → Callback 으로 등록
                
                → queue에 담아서 전달
                
            2. g_csi_queue
                
                queueHandle로써 espnowAP → espAI 파일까지 csi info 전달
                
            3. esp_ai_task
                
                전처리 과정
                
            4. amp
                
                subcarrier 별 신호 세기
