### TLB

⇒ VM의 주소를 PM의 주소로 바꾸는 속도를 높히기 위해 사용됨

- 최근 일어난 VM addr과 PM addr의 변환을 테이블에 기록한다
- 일종의 주소 변환 캐시이다
- MMU 안에 존재하는 고속 캐시이다

**기존 방식**

- 모든 메모리 참조는 최소 2번 일어났다

⇒ TLB에서 Page fault가 발생하지 않으면 1번의 참조로 주소 획득 가능

### Page fault

⇒ TLB Table에 접근했지만, 원하는 주소값이 없어서 Memory에 접근했지만, Memory에 주소가 존재하지 않아서 탐색을 실패한 경우

전체 흐름

```c
1. CPU가 VA 접근
2. TLB Miss
3. Page Table 조회
4. Page 없음 → Page Fault 발생
5. OS 개입 (Interrupt)
6. 디스크에서 페이지 읽기
7. 빈 Frame 확보 (or 교체)
8. Page Table 갱신
9. TLB 갱신
10. 명령 재시작
```

구조적 흐름

```c
CPU
 ↓
MMU
 ↓
TLB
 ↓ (Miss)
Page Table
 ↓
[Invalid] → Trap 발생
 ↓
OS (Page Fault Handler)
 ↓
Disk (Swap 영역)
 ↓
Memory 적재
 ↓
Page Table & TLB 업데이트
 ↓
CPU 재실행
```