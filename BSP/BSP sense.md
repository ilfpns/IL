- 서론
    
    1기 대마고 김경식 선배님과 대화를 해보니 BSP에 대한 기본적 구조 이해와 그 주변 개념의 이해와 실행 능력이 부족함을 꺠닫고 구조부터 파악하고자 한다.
    

먼저 BSP가 활용되는 Linux는 왜 써야할까?

- 먼저 BSP가 활용되는 Linux는 왜 써야할까?
    - Good scheduler
    - Good network stack
    - many protocol
    - many memory
    - support other many arch, (ARM, MIPS, x86, PowerPC)
    - open source
    - open license (GPL)
    - free
    
    ⇒ 복잡한 프로젝트에 적합하다, 하지만 RTOS같은 실시간성을 haard or soft하게 지키는 OS가 아니다.
    
    리눅스 철학은 다음과 같다.
    
    - Do One Thing and Do It Well
        - 한 가지 일만 완벽하게 해라
    - Everything is file
        - 모든 것은 파일이다
    
    즉 실시간성이나 보안을 위한 RTOS, macOS와는 다른 철학을 가진다.
    

- 먼저 우리는 어떤 방식으로 우리의 입력이 들어가고, 무슨 처리를 하는지 바탕이 있어야 한다.
    1. User : 명령 입력층
    2. Shell : 자연어 → 기계어
    3. Kernel : HW 동작
    4. HW : 물리층

파일 구조를 정리해야 한다. Linux는 결국 여러 파일들의 관계들로 이루어진 OS이기 때문이다.

- file sys
    
    
    ![image.png](attachment:30efb967-f10c-40a6-9be6-9b4eceb0b653:image.png)
    
    - host/                        <-- 개발 PC에서 쓸 크로스 컴파일러(Toolchain)가 들어있는 곳
    - images/                   <-- 최종 산출물 (보드에 구워야 할 파일들)
    - u-boot.bin               <-- 부트로더
    - zImage                      <-- 커널 이미지
    - board.dtb                <-- 하드웨어 설계도 (DTS가 컴파일된 것)
    - rootfs.ext4              <-- 루트 파일시스템 /bin, /etc, /usr 등이 들어있는 방대한 폴더 집합
    - sdcard.img             <-- 통합 이미지, 위의 모든 것을 하나로 합쳐둔 굽기용 파일
    
    - **U-Boot ──(조회)──> Kernel & DTB**
    전원이 켜지면 가장 먼저 `u-boot.bin`이 작동한다.
    U-Boot는 같은 파티션(또는 지정된 주소)에 있는 `zImage(커널)`와 `am335x-bbb.dtb(하드웨어 정보)`를 찾아서 램(RAM) 메모리로 복사해 올린 뒤, 리눅스 커널에게 지휘권을 넘긴다.
        - 임베디드 리눅스 프로그래밍 완전정복 3/e 발췌
            
            bootloader는 kernel에게 다음 3가지를 전달한다.
            
            1. PowerPc or ARM SoC를 비교하는 기계 번호
                - PowerPc와 ARM은 RISC ARCH로 만들어진 SoC이다.
                - arch/ARCH/boot/dts ← dts file path
            2. dtb (HW info file)
            3. kernel 명령
    
    - **Kernel ──(마운트)──> Rootfs**
    지휘권을 받은 `zImage(커널)`는 전해 받은 **`dtb`** 파일을 보고 보드의 HW를 파악하고 주변 장치들을 켠다.
    그 후 다른 파티션에 있는 `rootfs(루트 파일시스템)`를 마운트한다.
        
        파일과 (드라이버) HW를 연결하는 것을 마운트라 한다.
        
    
    - **Rootfs ──(실행)──> 최종 부팅 완료**
    이제 리눅스 커널이 `rootfs` 내부에 있는 `/sbin/init` 프로그램을 실행한다.
    이렇게 우리가 아는 리눅스 로그인 쉘(`root@board:~#`)다.
    

file sys에 대해 이해했으면 위에 폴더들의 개념부터 학습하면 된다.

먼저 CrossCompiler과 toolchain 개념이다.

- CrossCompiler & toolchain
    
    툴체인은 모든 코드를 컴파일하는 도구이다. 이는 프로젝트 초기 단계에 정해지며, 진행 도중에 절대로 바뀔 수 없다. 
    
    - toolchain : 소스 코드를 타깃 장치에서 실행할 수 있는 실행 파일로 바꿔주는, 컴파일러, 링커, 런타임 라이브러리를 포함하는 도구의 집함
    처음에는  bootloader, kernel, rootfs를 위해 toolcahin이 필요하다.
        - 흔한 리눅스용 툴체인은 GNU가 있다
            - + LLVM
            - + Clang (좀 더 빠르다는 장점이 있지만, GCC GNU는 광범위한 아키텍쳐를 지원하는 컴파일러라는 너무 큰 장점이 있다.
        
    - 툴체인은 여러 컴파일러, 링커, 라이브러리 등을 모아둔 도구를 지칭하고, 이는 GNU가 유명하다고 소개했다. 그렇다면 어떤 도구들이 GNU를 이룰까?
        1. Binutils : ASM과 Linker를 포함하는 유틸리티이다.
        2. GCC : C와 여러 언어를 위한 Compiler이다.
        3. C library : POSIX 규격에 기반을 둔 표준 API (APP ↔ OS kernel로 연결되는 규격) library가 있다.
    
    툴체인은 두 가지 종류가 있다.
    
    - native : toolcahin이 만들어내는 프로그램과 같은 종류의 시스템이다.
    - cross : toolchain이 타킷 기계와 다른 종류의 시스템에서 실행된다.
    
    부족한 HW 문제를 위해서 호스트에서 타깃과 환경을 분리한다.
    
    우리는 local PC에서 board를 위한 실행 파일을 만드는 cross compiler에 대해 집중하면 좋다. 
    
    - 이러한 compiler를 포함하는 toolchain은 CPU arch에 맞게 명령어가 입력되어야 한다.
        - CPU arch : ARM, MIPS, x86_64
        - big-endian, little-endian
            - 설명
                - big : 주소가 들어온대로 받는다.
                    
                    ⇒ 1 → 2 → 3 → 4
                    
                - little : 들어온 주소를 거꾸로 받는다.
                    
                    ⇒ 4 ← 3 ← 2 ← 1 ← 
                    
        - 부동소수점 지원
            - FPU (소수점 연산 장치)가 있는가?
                - FPU가 있다면 : hard float (hf)
                - FPU가 없다면 : soft float (sf)
        - ABI (Application Binary Interface)
            - 함수 호출 간에 인자를 넘기는 호출 규칙 (Interface를 규격으로 이해하면 편하다.)
            - 이후 ARM arch는 EABI를 개발한다.
                - EABI : 각 컴포넌트 간 데이터를 개발/주고받을 때 정해진 규칙
                - 예시
                    - 레지스터 사용 규칙 (Calling Convention)
                    - 데이터 타입 크기와 정렬
                    - 하드웨어 부동소수점(Floating Point) 지원 여부
                    
                
                부동소수점 인자 전달 방식에따라 두 가지 방식으로 나뉜다.
                
                - EABI (범용)
                - EABIHF ⇒ HF이므로 FPU가 없다면, 호환될 수 없다.
    
    ![image.png](attachment:52be4ce9-9e70-46d8-a4ff-acc73510c6ee:image.png)
    
    ```c
    gcc -dumpmachine
    ```
    
    다음 명령어를 통해 현재 local에서 사용하는 toolchain 정보를 볼 수 있다.
    
    - x86_64 arch
    - linux kernel
    - gnu user space
    

현재 있는 toolchain은 

```c
apt install gcc-arm-linux-gnueabihf
export CROSS_COMPILE=arm-linux-gnueabihf-
```

이런 방식으로 사용했다. 이 방법은 분명 편하지만 너무 제한적이다. 그러므로 개인 toolchain 구성을 해보려 한다.

- Crosstool-NG (빌드 시스템인 buildroot나 yocto를 사용하면, 더 빠르고 쉽게 할 수 있다.)
