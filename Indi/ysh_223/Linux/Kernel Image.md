## Kernel Image

⇒ 부트로더가 메모리에 로드하여 실행하는 커널 실행 파일

```c
// 부팅 Flow 위치
ROM → Bootloader → Kernel Image → init → userspace
```

1. Bootloader 준비
2. Kernel Image를 RAM에 올림
3. 엔트리 포인트로 점프

### Image

- 압축되지 않은 순수 커널 바이너리
- ARM64에서 많이 사용
    - boot protocol이 단순함
    - 합축 해제를 지원하는 부트로더가 많음
    - 메모리적 여유가 있음
- 빠르지만 용량이 크다

### zImage

- 압축된 커널 이미지
    
- self-decompress 기능을 포함한다
    
    ⇒ 커널 이미지 안에 압축 해제 코드를 포함한다
    
    - 메모리에 올라가면
    - 스스로 압출을 풀고, 플린 커널로 점프한다
    - Bootload가 압축을 풀어줄 필요가 없다
- ARM 32bit에서 흔하다
    

### bzImage

- x86 계열에서 사용
- gzip + boot protocol
- big + zImage

### uImage

- U-Boot용 포맷
    
    ⇒ U-Boot가 이해할 수 있는 부트 이미지 형식
    
    - U-Boot는
    
    ⇒ 임베디드 리눅스에서 가장 많이 쓰이는 부트로더 프로그램으로, 다음과 같은 일을 한다
    
    - 로드 주소
    - 엔트리 주소
    - 아키텍쳐
    - CRC
    
    같은 정보를 헤더에서 읽음
    
- 헤더를 포함한다 (load Address, entry point 등)