[[CS]]
_**coherence하다**_

⇒ RAM에 있는 data를 읽을 떄, data가 유효 또는 가장 최신인 data임을 보장한다

_**consistency하다**_

⇒ RAM에 존재하는 data를 가져오거나 실행할 떄, 정해진 순서로 가져오거나 실행함을 보장

### Wrtie Policy

쓰기 정책을 크게 2가지로 나뉜다

- Write through
    
    ⇒ 한 캐시에서 일어난 변경점을 바로 다른 계층의 메모리에 전달
    
- Write back
    
    ⇒ 변경점을 바로 저장하지 않고(캐시에만 저장), 이후 변경이 일어난 line이 evicted(방출)될 떄 RAM에 반영
    
    - line
        
        캐시가 데이터를 관리하는 최소 단위
        
        - cache line = 32B
            
        - 주소 `0x20001004` 읽음
            
            → `0x20001000 ~ 0x2000101F` 통째로 캐시에 로드
            
    - evicted
        
        캐시는 크기가 작기에 새 line이 필요하면 기존 line을 버린다
        

### 예제

![[Pasted image 20260203172253.png]]

- Main memory에는 100이 있다
- Processor 1은 이 값을 읽어옴 (캐시에 100 저장)
- Processor 2는 이 값을 읽어와 200으로 바꿔 저장

⇒ Main memory, Processor 1, Processor 2 어느 곳에 있는 값이 유효한지 알 수 없음

⇒ 캐시 일관성 문제

⇒ 캐시와 RAM에 있는 값이 달라서 CPU, DMA가 보는 값이 다른 문제 발생

### 해결법

**Cache maintenance**

⇒ 캐시와 RAM을 강제로 동기화

```c
// 캐시에만 있는 내용을 RAM에 반영 (Write-back)
SCB_CleanDCache_by_Addr(buf, len);

// CPU 캐시에 있는 해당 주소의 데이터들을 무효화
SCB_InvalidateDCache_by_Addr(buf, len);

// 메모리 관련 명령이 끝날 때까지 CPU를 멈춤
__DSB();
```

1. RAM <-> Cache 동기화
2. Cache값 무효 (-> RAM값을 기준으로 사용)
3. 작업이 끝날 때까지 기다림 (꼬이지 않게 하기 위해)