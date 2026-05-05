---

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
     -initrd rootfs.cpio   
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
