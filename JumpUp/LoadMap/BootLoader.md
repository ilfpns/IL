

## BootLoader?

컴퓨터가 부팅될 때 가장 먼저 실행되는 소프트웨어로, OS를 로드 및 실행하며, 커널을 로드하는 역할을 한다.

동시에 하드웨어를 초기화하고, 메모리 맵을 커널에 전달한다

### 대표적인 BootLoader

**BIOS: 가장 일반적으로 많이 사용되는 부트로더**

- 16bits real mode
    - CPU 내부에서 16bits단위로 동작하는 레지스터이다
    - 운영체제는 부트 시 메모리에 대한 가상 메모리 개념이 없기 때문에 프로그램이 명령한 주소와 메모리에 위치한 주소가 같다
    - 다른 종류의 mode
        - **Protected Mode: 메모리를 보호하는 기능과 가상 메모리 기능을 사용함**
        - **Long Mode: 모든 기능이 활성화 되며 64bits로 작동한다**
        - Real mode → Protected mode → Long mode 순으로 로드된다
        
    - Real Mode는 독특한 메모리 주소 계산법을 가진다
        - 16bits는 2^16 = 64KB 만큼의 공간만 사용 할 수 있는데, 더 많은 공간을 사용하기 위해서 다음 기능을 사용했다
        - 세그먼트 (Segment)와 오프셋 (Offset) 이라는 두 개의 16bits 레지스터를 조합하는 방법을 사용함
        - 예시
            
            BSP는 RTOS에 Periperel 등의 레지스터를 제어해야 한다
            
            - UART base addr: 0x40011000
            - TX offset:              0x04
            - STATUS:                0x08
            
            ```c
            #define UART_BASE         0x40011000
            #define UART_TX_REG       (*(volatile uint32_t *)(UART_BASE + 0x04))
            #define UART_STATUS_REG   (*(volatile uint32_t *)(UART_BASE + 0x08))
            ```
            
- MBR 기반
    - Master Boot Record는 하드디스크나 SSD같은 메모리의 가장 맨 앞에 위치한 512 바이트 크기의 작은 공간이다
    - BIOS는 OS의 위치를 디스크에서 스스로 찾을 수 없기에, MBR안에 Raw Sector 주소만을 복사하고 실행 권한을 부여한다
        - MBR→Raw_Sector 안에는 개발자가 직접 하드코딩한 다음 코드 실행 주소가 있다
- 파일 시스템 자원이 없으며, 디스크가 직접 읽는다
- 디스크에서 커널로 직접 로드한다
    - OS없이 파일 시스템을 사용할 수 없기에, 개발 진행 시 MBR에 Raw Sector를 지정한다.
    Raw Sector안에 있는 코드는 실행되며 부트가 진행된다.
    - 이 방식을 파일 구조를 거치지 않고 디스크에게 물리적 주소를 직접적으로 지시하는 LBA라고 한다

**UEFI: 부팅만을 위해 존재하는 작은 OS와 같다**

- 64bits 실행 가능
- GPT기반
    - Guid Partition Table은 MBR의 업그레이드 버전이다
        - Disk 인식: 제타바이트까지 인식 가능
        - Partition: 128개 이상 사용 가능
            - **SecTor: 데이터를 저장하는 가장 작은 단위**
                
                ![image.png](attachment:0633a763-1a71-464b-a96e-39a0943b59d7:image.png)
                
                HDD나 SSD와 같은 저장 매체에서 데이터를 읽고 쓰는 물리적/논리적 최소 단위이다. 
                
                1개의 SecTor는 보통 512byte의 크기를 가진다 하지만 최근에 대용량 드라이브나 (HDD) SSD에서는 효율성을 위해 4096byte (4KB)를 하나의 Sector로 사용한다.
                
                OS가 disk에 데이터를 기록할 때 1~10 SecTor까지 데이터를 저장 하듯이 주소를 지정하여 데이터를 관리하는 절대저인 기준점이 된다.
                
                주소: LBA (Logical Block Addrees
                
                HDD나 SSD 등 저장 장치의 데이터 위치를 0번부터 순차적인 일련번호로 지정하는 방식
                
            - Partition: 디스크의 논리적으로 나뉜 구역
                
                파티션은 하나의 물리적인 디스크를 여러 개의 독립적인 논리적 구역으로 분할하는 것을 말한다. 수많은 SecTor를 묶어서 용도별로 구역을 나눈다.
                
                Window OS는 디스크를 C, D 등으로 나누는 것과 같은 원리지만, 리눅스에서는 이를 다루는 방식의 차이가 있다.
                
                **특징**
                
                1. Mount: 구역별로 나뉜 파티션을 리눅스 특정 폴더에 연결하여 사용한다. 이런 방식을 마운트라 한다.
                ⇒ EX: 1번 파티션은 /에 연결, 2번 파티션은 /home에 연결하는 식이다
                2. 안정성과 독립성: 파티션을 나눠서 데이터를 관리하면 특정 파티션에 문제가 생겼을 때 파티션을 복구 할 수 있다.
                3. 디바이스: 리눅스에서는 하드웨어도 파일로 취급하여 /dev 디렉토리에 표시한다.
- FAT32 파일 시스템 지원
- 파일 시스템에서 읽어서 로드

## 작동 흐름

1. ROM code 
    - 역할 : 전원 인가 / 리셋 직후 가장 먼저 실행, SoC 내부에 영구 저장
    - 특징 : DRAM을 초기화 하지 않음, SRAM만 사용
    - 핵심 : 다음 부트 코드를 SRAM에 올려줌

1. SPL
    - 역할 : DRAM 초기화,  커널 또는 TPL을 DRAM에 로드
    - 제약 : 크기 제한이 큼
    - 핵심 : DRAM을 쓸 수 있게 만드는 단계

1. TPL
    - 역할 : 완전한 부트로드, 커널 + DTB + initransfs 로드
    - 특징 : 다양한 드라이버 포함, 사용자 개입 가능
    - 핵심 : 커널을 메모리에 올리고 실행시키는 관리자

1. 커널
    - 커널 entry로 점프
    - 부트로더는 메모리에서 제거
    - 이후 시스템 제어는 커널 + userspace

| 단계 | 메모리 | 하는 일 |
| --- | --- | --- |
| ROM Code | SRAM | 다음 로더 로드 |
| SPL | SRAM → DRAM | DRAM 초기화 |
| TPL(U-Boot) | DRAM | 커널 로드/부팅 |
| Kernel | DRAM | 시스템 운영- |

---

## 부트로더 구성 요소

부트로더는 기본적으로 여러 기술들의 집약체지만 중요한 4개는 다음과 같다

- **링크 스크립트**
    - 컴파일 후 코드 조각을 `.text` (실행코드), `.data` (초기화 변수), `.bss` (초기화 안 된 변수) 영역을 나누고, 이것들을 물리적 메모리 주소에 매핑 규칙을 명시한다
    - 부트로더 파일인 `.ld`안에는 물리적 메모리 주소 매핑 규칙서인 링크 스크립트가 들어있다
    
- **스타트업 코드**
    - `main.c` 를 호출할 OS 코드이다
    - 스택을 초기화한다

- **크로스 컴파일 툴체인**
    - OS 환경은 보통 x86/AMD 환경이다. 하지만 우리는 ARM기반의 보드에서 실행되는 코드가 필요하다
    - gcc와 이를 연결하는 Makefile을 연동하여 사용해야한다
        - 컴파일: 코드를 한번에 기계어로 바꾼다
        - 인터프리터: 코드를 한줄씩 기계어로 바꾼다
        - 툴체인: 여러 컴파일, 빌드 도구를 포함하는 툴
        
- **하드웨어 디버깅 및 테스트 환경 QEMU & UART**
    - **QEMU (에뮬레이터):** 실제 보드에 굽기 전에, PC 안에서 ARM 보드나 x86 시스템을 가상으로 띄워놓고 부트로더를 테스트할 수 있는 필수 에뮬레이터이다

---

## 부트로더 뜯어보기

- LoadMap
    
    **[0] 환경 세팅 ✅ 완료**
    
    `목표: 크로스컴파일 환경 구성
    만진 것: WSL, 환경변수
    
    export ARCH=arm
    export CROSS_COMPILE=arm-linux-gnueabihf-
    
    - WSL 설치
    - arm-linux-gnueabihf- 툴체인 설치
    - ARCH, CROSS_COMPILE 환경변수 설정`
    
    **[1] 커널 빌드 ✅ 완료**
    
    `목표: ARM용 커널 빌드
    만진 파일: kernel/linux/
    
    - Linux 커널 소스 clone
    - vexpress_defconfig로 설정
    - 빌드 → zImage, vexpress-v2p-ca9.dtb 생성
    - QEMU로 커널 단독 부팅 확인`
    
    **[2] rootfs 구성 ✅ 완료**
    
    `목표: 커널이 올릴 최소 파일시스템 구성
    만진 파일: rootfs/
    
    rootfs/
    ├── init          ← 커널이 PID 1로 실행
    ├── bin/sh        ← busybox 복사
    ├── dev/console   ← mknod으로 생성
    ├── proc/         ← 마운트 포인트
    └── sys/          ← 마운트 포인트
    
    - BusyBox 소스 clone
    - static 링크로 빌드
    - rootfs 디렉토리 구조 생성
    - cpio로 묶어서 rootfs.cpio 생성
    - QEMU로 부팅 확인 → shell 프롬프트 확인`
    
    **[3] U-Boot 빌드 ✅ 완료**
    
    `목표: U-Boot 빌드 및 쉘 확인
    만진 파일: u-boot/
    
    - U-Boot 소스 clone
    - vexpress_ca9x4_defconfig로 설정
    - 빌드 → u-boot 실행파일 생성
    - QEMU로 U-Boot 쉘 (=>) 확인`
    
    **[4] U-Boot에서 커널 부팅 ← 지금 여기**
    
    `목표: U-Boot 쉘에서 bootz로 커널 + rootfs 부팅
    만지는 파일: run.sh, U-Boot 쉘 명령어
    
    - run.sh 스크립트 작성
    - QEMU 실행 시 커널/DTB/rootfs 메모리에 올리기
    - U-Boot 쉘에서 bootz 명령어 실행
    - 커널 → rootfs → shell 뜨는 것 확인
    
    부팅 체인:
    QEMU → U-Boot → 커널 → rootfs → shell`
    
    **[5] U-Boot 포팅**
    
    `목표: 존재하지 않는 새 보드를 U-Boot에 등록
    만지는 파일:
      board/mycompany/myboard/myboard.c
      include/configs/myboard.h
      configs/myboard_defconfig
    
    - board/mycompany/myboard/ 디렉토리 생성
    - myboard.c 작성 (보드 초기화 코드)
    - myboard.h 작성
        - 메모리 맵 (DRAM 주소, 크기)
        - UART 설정 (콘솔 출력)
        - 부팅 명령어 (CONFIG_BOOTCOMMAND)
    - myboard_defconfig 작성
    - 빌드 후 QEMU로 부팅 확인`
    
    **[6] Device Tree 작성**
    
    `목표: 새 보드의 하드웨어를 DTB로 표현
    만지는 파일: arch/arm/dts/myboard.dts
    
    - .dts 파일 직접 작성
        - CPU 정의
        - 메모리 정의
        - UART 정의
        - 버스 (I2C, SPI) 정의
    - dtc로 컴파일해서 .dtb 생성
    - 커널에 붙여서 부팅 확인`
    
    **[7] 커널 포팅**
    
    `목표: 새 보드에서 커널이 정상 동작
    만지는 파일:
      arch/arm/configs/myboard_defconfig
      drivers/
    
    - 새 보드용 defconfig 작성
    - DTB 연결
    - 드라이버 확인
        - UART 드라이버 (콘솔 출력)
        - 메모리 드라이버
        - GPIO 드라이버
    - 커널 부팅 로그 분석`
    
    **[8] rootfs 자동화 (Buildroot)**
    
    `목표: Buildroot로 rootfs 자동 생성
    만지는 파일: buildroot/.config
    
    - Buildroot 설정
    - 패키지 추가 (busybox, 네트워크 툴 등)
    - 자동으로 rootfs.cpio 생성
    - 수동으로 만든 것과 비교`
    
    **[9] 디버깅**
    
    `목표: 부팅 안 될 때 혼자 디버깅
    만지는 파일: 로그 분석
    
    - UART 로그 분석
    - 커널 패닉 메시지 해석
    - U-Boot 환경변수로 디버깅
    - JTAG 디버거 개념 (실제 보드)`
    
    **[10] 실제 보드 (선택)**
    
    `목표: QEMU 말고 실제 하드웨어에서 동작
    대상: Raspberry Pi 4 or BeagleBone Black
    
    - SD카드/eMMC에 이미지 굽기
    - 실제 UART 연결해서 로그 확인
    - 실제 부팅 테스트`
    
    ---
    
    ### 실무 BSP 엔지니어가 실제로 하는 것
    
    `1. 칩 벤더한테 새 SoC + 데이터시트 받음
    2. 레퍼런스 보드 BSP 분석
    3. 우리 보드에 맞게 U-Boot 포팅
    4. DTB 작성
    5. 커널 드라이버 붙이기
    6. 디버깅 (이게 제일 오래 걸림)
    7. QA 팀에 넘기기`
    
    지금 연습이 3, 4, 5, 6번이에요.
    
    ---
    
    ### 지금 연습 vs 실무 비교
    
    | 지금 연습 | 실무 |
    | --- | --- |
    | QEMU 가상 보드 | 실제 보드 |
    | 수동 rootfs | Yocto / Buildroot |
    | mainline U-Boot / 커널 | 칩 벤더 포크 |
    | vexpress 포팅 | 신규 보드 포팅 |
    | 혼자 | 팀으로 |
- WSL 설정 및 크로스 컴파일러 설정 → QEMU 기본 실행
    
    이제부터 실전으로 들어가서 부트로더를 뜯어볼 것이다
    
    - WSL download
        
        ~~이다연이 쌩고생을 하면서 WSL을 깔던 기억이 나서 정리한다~~
        
        ```c
        wsl --install
        // Power Sheell 관리자 권한
        ```
        
    - BSP dev loadmap
        
        ```c
        [1] 크로스 컴파일러 준비
              ↓
        [2] U-Boot 빌드 및 포팅
              ↓
        [3] 커널 빌드 및 포팅
              ↓
        [4] 디바이스 트리 수정
              ↓
        [5] Root Filesystem 구성
              ↓
        [6] 부팅 테스트 & 디버깅
              ↓
        [7] 애플리케이션 개발
        ```
        
    
    **WSL 이란**
    
    ⇒ WSL(Windows Subsystem for Linux)은 Windows 10/11 이상에서 별도의 가상 머신 없이 리눅스 환경(Ubuntu, Debian 등)을 네이티브에 가깝게 실행하는 기능
    
    이후 어떤 파일을 실행하기 위해서는 윈도우 환경변수라는 것을 알아야 한다.
    
    **환경변수**
    
    ⇒ 운영체제(OS)에서 프로세스가 동작하는 방식에 영향을 미치는 동적인 키-값(Key-Value) 형태의 설정값
    
    ```c
    export ARCH=arm
    export CROSS_COMPILE=arm-linux-gnueabihf-
    ```
    
    - 아키텍쳐를 ARM으로 지정한다
    - 또한 Cross compile을 다음과 같이 지정한다
        - arm: 타켓 CPU 아키텍쳐를 의미
        - linux: 타겟 보드에서 돌아갈 OS
        - gnu: c언어 표준 컴파일 라이브러리
        - eabi: Embedded AI의 약자로, 레지스터 사용 방식이나 데이터 크기, 함수 호출 등의 방식 규격
        - hf: SW단에서 처리할지, 소수처리 장치 FPU를 쓸 것인지 결정함 (현재는 FPU 사용)
        - -: 빌드 진행 접두사
        
        ⇒ `ARM32`비트 CPU위에서 돌아가는 `Linux`용 프로그램을 하드웨어 `소수점` 연산을 지원하는 표준 `GNU`규격으로 코드를 번역
        
    
    ```c
    make vexpress_defconfig
    ```
    
    - make는 설계도를 선택하는 명령어이다
    - vexpress_defconfig라는 파일을 해당 빌드 설계도로 설정한다
    - 이후 .config 파일이 만들어진다
    
    **QEMU 실행**
    
    ```c
    qemu-system-arm \
      -M vexpress-a9 \
      -m 256M \
      -kernel arch/arm/boot/zImage \
      -dtb arch/arm/boot/dts/vexpress-v2p-ca9.dtb \
      -nographic \
      -append "console=ttyAMA0"
      
      // 나가려면 ctrl + a 눌렀다가 떼고 x
    ```
    
    - M vexpress-a9: 가상 보드 설정
    - m: 메모리 설정
    - kernel: & dtb: 커널과 하드웨어 명세서 설정
    - nographic: UART로 화면만 모니터링 (별도의 창 x)
    - append: 콘솔 부팅 로그 설정
    
    ```c
     qemu-system-arm   
     -M vexpress-a9   
     -m 256M   
     -kernel kernel/linux/arch/arm/boot/zImage   
     -dtb kernel/linux/arch/arm/boot/dts/arm/vexpress-v2p-ca9.dtb   
     -initrd oldrootfs.cpio   
     -nographic   
     -append "console=ttyAMA0 rdinit=/init"   
     -audio none
    ```
    
    - initrd: 임시 루트 파일 시스템 이미지 지정
    - audio: 오디오 에뮬을 사용하지 않게해서 리소스 최적화
    
    ---
    
    현재는 수동적으로 RootFS를 만들었다
    
    ![image.png](attachment:e275b935-87bf-4f1c-8653-3d2ad1941b9b:image.png)
    
    하지만 이는 사용할때마다 만들어야 하는 불편함이 있다. 
    그렇기 때문에 자동 RootFS를 만들어 주어야한다.
    
    [RootFS](https://www.notion.so/RootFS-356d43ab359a8087bbf8c934e1baee2c?pvs=21)
    
- U-Boot 제작
    
    ```c
     qemu-system-arm   
     -M vexpress-a9   
     -m 256M   
     -kernel kernel/linux/arch/arm/boot/zImage   
     -dtb kernel/linux/arch/arm/boot/dts/arm/vexpress-v2p-ca9.dtb   
     -initrd rootfs.cpio   
     -nographic   
     -append "console=ttyAMA0 rdinit=/init"   
     -audio none
    ```
    
    현재 단계에서는 `initrd` 에 연결되는 rootfs를 만들어 볼 것이다.
    
    RootFS는 이전에도 설명했듯 단순히 root밑에 있는 파일 시스템이다.
    기본 구성 요소는 다음과 같다
    
    ```c
    rootfs/
    	init
    	bin
    	dev
    	proc 
    ```
    
    - init 파일
        
        ```c
        #!/bin/sh // /bin/sh에서 실행하도록 설정
        echo "=== Hello from my rootfs! ==="
        mount -t proc proc /proc // procfs를 proc에 올리기
        mount -t sysfs sysfs /sys // sysfs를 sys에 올리가
        exec /bin/sh
        ```
        
    - bin 폴더
        
        bin/sh 라는 경로로 busybox를 빌드 후 만들어진 busybox 파일을 cp로 bin/sh에 넣는다
        
        ![image.png](attachment:ca68b947-b999-4c70-8b16-2effb86c42cc:image.png)
        
        - BusyBox: 하나의 실행파일에 모인 명령어 + 심볼릭 링크
            
            BusyBox: 명령어 모음집으로 여러 명령어가 하나의 실행 파일에 들어있다
            
            Symbolic link: busybox하나를 가리키는 심볼릭 링크이다. 실행 할 때 입력을 보고 어느 명령어인지 판단한다
            
        
        왜 우리는 bin과 bin/sh에 폴더를 cp했을까?
        
        1. 리눅스 파일시스템 규약(FHS)에서 필수 실행 파일은 bin에 위치해야 한다
            - FHS
                
                ```c
                /bin    → 필수 실행파일 (sh, ls, mount 등)
                /dev    → 디바이스 파일 (console, tty 등)
                /proc   → 커널/프로세스 정보 (가상 파일시스템)
                /sys    → 하드웨어/드라이버 정보 (가상 파일시스템)
                /etc    → 설정 파일
                /lib    → 라이브러리
                ```
                
                - make
                    - 빌드 자동화 툴으로, Makefile에 적힌 규칙으로 컴파일 한다.
                    
                    ```c
                    make defconfig      # 기본 설정 파일 생성
                    make menuconfig     # GUI로 옵션 수정
                    make -j$(nproc)     # CPU 코어 수만큼 병렬 빌드
                    ```
                    
                    $j(nproc) CPU코어 수를 자동으로 설정, 4코어 기준 -j4와 같다
                    
                    mount: File system을 특정 dir에 연결한다
                    
                    ```c
                    mount -t proc proc /proc
                            ^타입  ^장치  ^연결할 디렉토리
                    ```
                    
                    - mknod
                        
                        ```c
                        mknod /dev/console c 5 1
                        #                  ^ ^ ^
                        #                  | | minor number
                        #                  | major number (5 = tty 계열)
                        #                  character device
                        ```
                        
        2. init코드 마지막에 bin/sh로 이동하므로 초기화 이후 사용자가 명령어를 입력 할 수 있는 shell을 뛰운다
    - dev, proc
        
        init에서 mount용 포인트 역할으로 디렉토리만 존재하면 된다
        
    
    이제 이 roofs파일을 cpio로 만들어야 한다
    
    cpio: tar과 비슷하게, 여러 파일을 하나의 아카이브로 묶는 툴이다. 리눅스 커널이 initramfs 포맷으로 cpio를 사용한다
    
    ```c
    find . | cpio -H newc -o > ../rootfs.cpio
    ```
    
    - find . → 현재 디렉토리 파일 출력
    - cpio -H newc → newc 포맷으로 묶기 (커널이 정한 포맷)
    - -o → output 모드로 묶기
    - > ../rootfs.cpio → 상위 디렉토리에 저장
    
    이제 이전에 실행했듯 QEMU 실행하면 된다
    
    ```c
    -initrd rootfs.cpio
    
    // 지금까지 한 것들의 순서는 다음과 같다
    Cross compile -> kernel build -> Busybox build -> create rootfs -> QEMU 부팅 
    ```
    
- U-boot 빌드 및 포팅
    - 현재 순서:  `QEMU → kernel → rootfs`
    - 바꾼 순서: `QEMU → U-Boot → kernel → rootfs`
    
    차이점
    
    1. 기존 방식은 QEMU가 커널을 직접 메모리에 올려준다 → QEMU가 BootLoader 역할을 한다 (실제 하드웨어는 할 수 없음)
        
        ⇒ QEMU가 U-Boot를 올리고, U-Boot가 커널을 올리는 방식이다.
        
        - QEMU: U-Boot 올리기
        - U-Boot: HW초기화, kernel load
    
    U-Boot Shell 에서 가능한 것
    
    - 네트워크로 커널 받앗서 부팅 (tftp)
    - 부팅 파라미터 수정
    - 플래시 메모리에 이미지 굽기
    - 디버깅
    - USB로 이미지 받아서 부팅
    - 환경변수 저장/불러오기
    - 스크립트로 자동 부팅 시퀀스 구성
    - HW test
        
        **QEMU clone** 
        
        ```c
        git clone https://source.denx.de/u-boot/u-boot.git
        ```
        
        **실행**
        
        ```c
        make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- vexpress_ca9x4_defconfig
        sudo apt install libgnutls28-dev
        make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- -j$(nproc)
        
        qemu-system-arm \
          -M vexpress-a9 \
          -m 256M \
          -kernel u-boot \
          -nographic \
          -audio none
        ```
        
        [Addr](https://www.notion.so/Addr-358d43ab359a800f845bdd6ab9775949?pvs=21)
        
        **흐름**
        
        ```c
        QEMU 실행
            ↓
        U-Boot 시작 (하드웨어 초기화)
            ↓
        U-Boot 쉘 (=> 프롬프트)
            ↓
        bootz 명령어 실행
            ↓
        커널 압축 해제 & 실행
            ↓
        /init 실행 (PID 1)
            ↓
        /bin/sh 실행
            ↓
        ~ # 프롬프트
        ```
        
        딱봐도 뭔가 복잡해졌다.
        
        QEMU는 -device loader로 kernel/DT/rootfs를 지정한 메모리 주소에 직접 가져다 둔다. 또한 U-Boot는 하드웨어 초기화 이후 그 주소해서 커널을 찾아서 실행한다
        
        실제로는 다음과 같은 흐름이다.
        
        즉 QEMU가 ROM code + SPL의 역할을 해주는 것이다
        
        ```c
        전원 => ROME code -> SPL -> U-Boot -> kernel
        ```
        
        ROM code는 알겠는데, SPL은 뭘 의미할까?
        
        Secondary Program Loader의 약자이다.
        
        ROM code는 SoC안에 영구적으로 박혀있다. SRAM은 전원이 들어오면 바로 쓸 수 있지만, DRAM을 그렇지 않다.
        
        그렇다고 SRAM에 U-Boot를 올리기엔 (KB인 SRAM은 MB인 U-Boot를 올릴 수 없음) 너무 작다
        
        그렇기 때문에 STAM에 올릴 수 있는 작은 로더가 필요했고, 이를 SPL이라 한다
        
    
    **하는 일**
    
    ```c
    1. DRAM 컨트롤러 초기화
       (클럭, 타이밍, 전압 설정)
            ↓
    2. DRAM 테스트
            ↓
    3. DRAM에 U-Boot 전체 복사
            ↓
    4. U-Boot로 점프
    ```
    
    - 메모리 관점
        
        ```c
        전원 ON
            ↓
        ROM Code (SRAM) → SPL 올리기
            ↓
        SPL (SRAM) → DRAM 초기화 → U-Boot 올리기
            ↓
        U-Boot (DRAM) → 커널 올리기
            ↓
        커널 (DRAM) → 시스템 운영
        ```
        
    - BSP가 다루는 MEM
        
        [SoC 내부]                    [SoC 외부]
        ┌─────────────┐              ┌─────────────┐
        │ ROM (SRAM)      │              │  DDR RAM        │ ← 실행 메모리
        │ IRAM            │              │  eMMC/UFS       │ ← 저장장치
        │ Cache           │              │  NAND Flash     │ ← 저장장치
        │ 레지스터           │              │  NOR Flash      │ ← 부트용
        └─────────────┘              └─────────────┘
        
        SRAM (on-chip)
        SoC 칩 안에 내장된 RAM이다
        용량은 당연히 작을 것이다. 대신 빠르겠지
        
        이는 DDR 초기화 전에 유일하게 쓸 수 있는 메모리이다.
        그렇다면 DRAM은? 못 쓴다, 초기화 전에 사용하지 못 한다.
        
        DDR을 올리기 전까지 SRAM을 사용한다
        (사실 Secondary Program Loader라는 게 있긴 하다, SRAM안에서 돌아가는 프로그램으로 DRAM을 init한다)
        
        ---
        
        Double Data Rate SDRAM
        DDR은 쉽게 말하면 그냥 RAM이다.
        
        개념에 들어가기 앞서 SDRAM에 대해 알아보고자 한다.
        
        DRAM (Dynamic RAM)
        
        - 커패시터 전하 저장
        - 느리지만 큰 용량 보유
        - 저렴하다
        - 주기적 리프레시가 필요하다
        
        S는 SRAM을 의미하는 것 같지만, 사실은 Synchronous이다.
        즉 클럭을 맞추는 DRAM이다.
        
        CPU/Mem ctrl의 clock signal과 동기화되어 동작한다.
        => 몇 클럭 뒤에 데이터 출력 <= 같은 기능이 가능하다.
        
        SDR (Single Data Rate) - 클럭 올라갈 때만 데이터 전송
        DDR (Double Data Rate) - 클럭 올라갈 때 + 내려갈 때 둘 다 전송
        → 같은 클럭으로 2배 대역폭
        즉 SDR은 Rising Edge 시 데이터를 전송하고, DDR은 Falling Edge와 Rising Edge 전부 데이터를 전송한다는 의미이다.
        저전력을 요구하는 Embedded System에서는 LPDDR을 사용한다. 이는 Low Power를 의미한다.
        
        ---
        
        DDR은 SRAM이 아니다, 이전에 공부했듯 초기화 이전에 사용 가능한 RAM은 SRAM 밖에 없다.
        즉 BSP는 DDR을 초기화하는 BootLoader를 만드는 것이 좋은 방향임을 알 수 있다.
        
        제조사마다 DDR 설정이 다르기 떄문에, 물리적 조건에 따라 타이밍 파라미터를 전부 다 맞춰줘야 한다
        
        - tCL : 명령을 보내고 데이터가 올 때까지 대기
        - tRCD : 행을 열고, 열을 접근하기까지 대기
        - tRP : 행을 닫는 데 걸리는 시간
        - tRAS : 행을 열어두는 최소 시간
        
        ```
        # 메모리 맵 정의 (Device Tree)
        memory@0 {
            reg = <0x0 0x00000000 0x0 0x80000000>; // 2GB
        };
        
        # 영역 분할
        0x00000000 ~ 0x00008000  → 커널 시작
        0x00000000 ~ 0x08000000  → 커널 영역
        0x08000000 ~ 0x10000000  → 유저스페이스
        0x10000000 ~ 0x18000000  → GPU 전용
        0x18000000 ~ 0x20000000  → ISP/카메라 전용
        ```
        
        CMA (Contiguous Memory Allocator)
        카메라, 비디오 인코더같은 HW는 물리적으로 연속된 큰 메모리가 필요하다
        이걸 미리 예약하는 역할이 BSP이다
        
        ```
        # Device Tree에서
        reserved-memory {
            cma_region: cma {
                size = <0x20000000>; // 512MB 예약
                linux,cma;
            };
        };
        ```
        
        ---
        
        NAND Flash
        not and flash임을 그 누구도 쉽게 알 수 있다.
        특징은 다음과 같다
        
        - 큰 용량
        - 읽기/쓰기가 비교적 빠르다
        - Bad block이 발생한다
        - XIP 안 된다 (RAM에 올려서 실행해야 한다)
            - eXecute In Place의 약자로, 메모리에 복사하지 않고 저장장치에서 직접 실행
        
        Bad Block관리는 BSP의 핵심이다.
        
        NAND는 쓰다보면 특정 블록이 죽는다.
        (데이터를 Electron 형태로 저장하는데, 이를 쓰기/지우기를 반복할수록 마모된다)
        죽은 블록은 건너뛰고 써야 한다.
        
        BBT (Bad Block Table)
        
        - 어느 블록이 죽었는지 기록
        - BootLoader가 init될 때 스캔한다
        - dead block을 자동으로 skip한다
        
        ECC (Error Correction Code)
        
        - 비트 에러를 자동으로 수정한다
        - HW ECC 쓰는지 or SW적으로 CPU를 사용해야 하는지에 따라 다름
        - ECC 강도 설정 (4bit, 8bit, 24bit, 몇 비트까지 오류 검출이 가능한가?)
        
        eMMC
        
        - NAND Flash + 컨트롤러가 하나의 패키지
        - Bad Block 관리를 내부에서 알아서 함
        - BSP 입장에선 그냥 블록 디바이스로 보임
        
        UFS
        
        - eMMC 후속
        - 속도 훨씬 빠름
        - rk3588 같은 고급 SoC에서 씀
