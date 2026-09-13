### ISA
[[OS]]
Instruction Set Architecture의 약자로, SW와 HW 사이의 약속을 의미한다

low-level language로 번역된 후 HW에 명령을 내릴 때, SW와 HW를 연결하는 것이 ISA이다

### RISC

⇒ CPU 설계 아키텍쳐

이런 ISA에는 RISC방식이 포함된다

Reduced Instruction Set Computer의 약자로, 단어의 뜻만 보아도 명령의 수가 CISC에 비해 줄어듦을 알 수 있다

CISC의 명령어 중에서 20%만큼만 상요되지만, 비슷한 일 처리를 보여준다

**장점**

- 적은 명령어
- 빠른 속도
- 단순
- 적은 전력 소모
- 저렴한 가격

**단점**

- 하드웨어가 간단하지만, 소프트웨어가 복잡함

**용도**

RISC 구조는 파이프라인 중첩이 가능해서 같은 수의 명령어에 대해 적은 clock으로 처리가 가능하다

따라서 임베디드 시스템에서 많이 사용된다

### MIPS란?

- RISC ISA에 한 종류이다
- 고정된 길이 (32bits)와 형식(R/I/J)를 가진다. 32bits 기준으로 0~31까지 32개의 register를 가진다
- MIPS는 모든 명령어들에 대해 3개의 operands를 가진다

⇒ register-register arithmetic instruction의 약자로,

메모리에 액세스하기 위해서 무조건 register로 데이터를 가져온다.

연산 후에는 결과를 메모리에 저장하는 구조로 작동한다

**MIPS Pipeline**

일반 Pipeline과 같이 여러 task를 일이 끝남에 따라 잇따라 실행되게 하는 구조가 같다

Pipeline은 병렬처리보다 느리지만 비슷하고, 순차처리보다는 빠른 속도를 보여준다