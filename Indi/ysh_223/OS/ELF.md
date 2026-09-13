### ELF 파일과 4대 핵심 세션
[[OS]]
개발자가 짠 코드는 컴파일을 거치면 .o (object 파일이 된다), 이것들이 모여서 ELF 실행 파일로 변하게 된다

**ELF란?**

⇒ 실행 가능 (Excutable)하고 링크 가능 (Linkable)한 File의 Format을 뜻한다

**ELF file의 format 구조**

- .text (코드): 실행할 기계어 명령어들이다. 수정할 일이 없으니 보통 Flash/ROM에 들어간다
- .rodata (읽기 전용): const로 선언한 변수나 문자열 상수들이다. 역시 Flash에 둔다
- .data (초기화된 데이터): int a = 10; 처럼 초기값이 있는 전역 변수나 static 변수이다
- .bss (초기화 안 된 데이터): int b; 처럼 초기값이 없거나 0으로 초기화된 전역/static 변수이다. 파일 용량을 차지하지 않고, 실행 시 RAM에 자리만 잡아둔다

**하드웨어적 메모리 관점**

- Flash (비휘발성, ROM의 일종) : 전원이 꺼져도 데이터가 날아가지 않는다, 하지만 쓰기가 까다롭다. 바로 실행하는 용도로 사용한다
- SRAM (휘발성, RAM) : 전원이 꺼지면 데이터가 날아간다 대신 읽고 쓰는 속도가 CPU 정도로 빠르다, 연산을 위한 변수가 저장된다

**VMA vs LMA**

- LMA (Load Memory Address) : 전원이 꺼져도 보존되어야 하는 Flash 메모리 주소
    
    ⇒ 출력파일을 로드할때 위치하는 주소
    
- VMA (Virtual Memory Address) : CPU가 실제로 연산할 때 사용하는 RAM 주소
    
    ⇒ 출력파일이 실행될 때 섹션이 위치하는 주소