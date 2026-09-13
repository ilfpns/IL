## Memory hierarchy
[[CS]]
![[Pasted image 20260307223415.png]]
1. Register : PC, IR
2. Cache : L1, L2
3. RAM : SRAM, DRAM
4. Disk : HDD, SSD
5. Optical Disk : 광디스크
6. Magnetic Tape : 자기 테이프

### cache

⇒ 두 장치 간의 속도차가 큰 장치의 속도차에 따른 병목 현상을 줄이기 위해 만들어진 고속 메모리

⇒ Cache와 RAM 사이에서 정보를 옮기는 것을 Mapping이라 한다

- **Mapping**
    1. 직접 매핑 Direct Mapping
        
        ⇒ RAM의 블록들이 지정된 한 개의 캐시 라인으로만 매핑될 수 있음
        
        ⇒ RAM의 한 블록마다 지정된 Cache line이 존재한다
        
    2. 연관 매핑 Associate Mapping
        
        ⇒ Cache line에 정해진 RAM 블록을 확인하는 Tag가 존재함
        
        ⇒ 모든 태그를 병렬로 검사하여 매핑
        
    3. 집합 연관 매핑 Set Associate Mapping
        
        ⇒ 정해진 블록의 집합내 어디서든 매핑이 가능하다
        
        ⇒ Cache Line을 Set으로 묶고, RAM 블록은 지정된 Set내의 아무 라인에 위치할 수 있다
        

**Cache Locality 캐시의 지역성**

⇒ CPU의 cache hit율을 높히기 위해 사용됨

⇒ 지역성의 전제조건으로 프로그램은 모든 코드나 데이터를 균등하게 Access하지 않는다는 특성을 기본으로 한다

- Locality : 기억 장치 내의 정보를 균일하게 Access하는 것이 아닌 어느 한 순간에 특정 부분을 집중적으로 참조하는 특성
    
    ⇒ 균등하다면, Cache값을 예측할 수 없이 모든 값을 Access하게 됨
    
    ⇒ 다음 두 지역성에 맞춰 Cache data가 준비됨
    
    - 시간 지역성
    - 공간 지역성