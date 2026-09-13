---

### General

- PetaLinux
    
    Xilinx사의 FPGA 기반의 SoC에서 embedded Linux 개발을 위한 SDK이다.
    
    프로젝트에서 사용하고자 하는 보드는 AMD Zynq7000 제품군이다. 
    
    - ARM Cortex A9 CPU를 기반으로 하는 보드이다.
    
    Zynq 7000 제품군은 FPGA의 유연성과 확장성을 제공하는 동시에 ASIC 및 ASSP 만큼의 성능을 제공한다.
    
    - AISC : 주문형 반도체로 특정 목적이나 App에 맞춰 설계된 반도체이다.
    - ASSP : 특정 용도로 설계했지만, 범용적으로 사용할 수 있도록 판매하는 제품
    
    AMD는 Zynq 7000 라인에서 BSP를 제공하지만 BARN AI는 다음과 같은 이유로 개인 BSP를 개발하게 된다.
    
    1. AMD BSP는 기본 설정만 제공한다 
        - SD, FPIO, SPI, I2C 같은 Peripheral 제어만 제공한다
        - PL IP, AXI, DMA, Sensor interface까지 제공하는 BSP가 필요하다
    2. Vivado
        - Vivado Hardware 구조가 매번 다르다.
        - PL에 어떤 IP를 넣는지, 주소는 어디에서 어느 interrupt를 쓰는지
        - DMA사용 여부 등의 하드웨어 구조 파악
    3. DTS
        - 직접 만든 Pl IP나 Sensor node에 따른 Hardware를 위한 DTS 수정
    4. BSP for BARN AI 
        - Sensor → Zynq PL/PS → PetaLinux → Jetson/ROS2/AI 의 파이프라인 흐름 구성
    
    개발을 위해서 세 IDE를 사용한다.
    
    - Vitis : FW제작 IDE
    - Vivado : FPGA/HW 제작 IDE
    - Petalinux : BSP 제작 IDE
- Design Flow
    1. Vivado에서 Xilinx IP 및 SoC 설계
        1. AXI4-Lite 기반 커스텀 IP, BRAM, GPIO, DMA 등 구성
        2. Zynq PS 설정
        3. XSA
    2. PetaLinux로 Linux system 구성
        1. petalinux-create로 프로젝트 생성
        2. peralinux-config —get-hw-description으로 Vivado의 XSA 정보 가져옴
        3. 자동으로 Device tree, kernel config, U-Boot 설정 생성
        4. Kernel module, app, device driver 추가 가능  
    
    <aside>
    
    !image.png
    
    </aside>
    
    PetaLinux라는 좋은 OS가 있으므로, Vivado에서 대충 구상이 끝나면 바로 BSP 개발을 쉽게 시작할 수 있다.
    
    이를 통하여 Hardware IP (하드웨어 블록 설게도)를 쉽게 사용할 수 있다.
    
    - 파일단위로 자원 사용 가능
    - SSH 접속
    - fs
    - c/c++ 등의 APP 실행
    - network
    - multitasking
    
    ---
    
    | **Vitis (Baremetal or RTOS)** | **Petalinux (Embedded Linux)** |
    | --- | --- |
    | CPU가 부팅하자마자 .elf 펌웨어 실행. | CPU가 부팅하면 Linux 커널이 올라가고, 리눅스 시스템이 시작됨. |
    | 직접 하드웨어 레지스터에 접근해서 제어 속도 빠름, 지연 적음. | 그 위에서 사용자 앱이 실행되며, 하드웨어는 Linux 커널이 제공하는 드라이버 계층을 통해 제어. |
    | 시스템 기능이 제한됨 (파일시스템, 네트워크, 패키지 없음). | 네트워크, 멀티프로세싱, 파일 IO, 디버깅, 패키지 설치 등 복잡한 기능 수행 가능. |
    | MCU 펌웨어 짜듯 사용하는 방식. | Raspberry Pi에서 Python으로 센서를 제어하는 느낌과 비슷. |
    
    ```c
    1. Vivado에서 하드웨어 설계
       Zynq PS 설정 + 필요하면 PL IP 추가
       ↓
    2. .xsa export
       하드웨어 정보 묶음
       ↓
    3. PetaLinux에서 BSP/Linux 구성
       XSA import
       device tree / kernel / U-Boot / rootfs 생성 및 수정
       ↓
    4. petalinux-build
       ↓
    5. BOOT.BIN, image.ub, boot.scr, rootfs 생성
       ↓
    6. SD 카드나 QSPI에 올려서 보드 부팅
    ```
    
    다음 흐름으로 전체 개발을 볼 수 있다.
    
    - 개발 중 .xsa 파일이 바뀌면?
        
        ```c
        // 기본형
        cd <petalinux-project>
        petalinux-config --get-hw-description=/path/to/new_xsa_dir
        petalinux-build
        petalinux-package --boot --fsbl images/linux/zynq_fsbl.elf --u-boot
        ```
        
        ```c
        // PL bitstream이 바뀐 경우
        petalinux-package --boot \
          --fsbl images/linux/zynq_fsbl.elf \
          --fpga images/linux/system.bit \
          --u-boot
        ```
        
        ```c
        // 유지되는 항목
        project-spec/meta-user/recipes-bsp/device-tree/files/system-user.dtsi
        project-spec/meta-user/recipes-kernel/...
        project-spec/meta-user/recipes-modules/...
        project-spec/meta-user/recipes-apps/...
        rootfs config
        kernel config
        u-boot config
        ```
        
        ```c
        // dts 보관 권장 파일 위치
        project-spec/meta-user/recipes-bsp/device-tree/files/system-user.dtsi
        ```
        
        ```c
        // 전체 표
        PS clock, DDR, UART, SD 설정 변경
        → XSA 재import + rebuild 필요
        
        AXI GPIO 같은 PL IP 추가/삭제
        → XSA 재import + DTS 확인 + rebuild + BOOT.BIN에 bitstream 포함
        
        주소맵 변경
        → XSA 재import + driver/DTS 주소 확인 필수
        
        IP 이름 변경
        → device tree node/compatible/label 바뀔 수 있음
        
        단순 C app 수정
        → XSA 재import 필요 없음
        
        커널 모듈 코드 수정
        → XSA 재import 필요 없음
        ```
        
    - PS / PL
        - PS : Hardware IP block
        - PL : Firmware / Software
    
- PetaLinux Install
    
    https://github.com/Digilent/Petalinux-Zybo-Z7-10
    
    Ubuntu 최신 버전에서는 호환성에 문제가 있을 수 있기 떄문에 22.04 버전을 깔아준다.
    
    ```c
    sudo dpkg --add-architecture i386
    sudo apt update
    
    sudo apt install -y \
      gawk xvfb git dos2unix net-tools xterm build-essential \
      libncurses-dev tftpd-hpa zlib1g-dev zlib1g-dev:i386 \
      libssl-dev flex bison chrpath socat autoconf libtool texinfo \
      gcc-multilib libglib2.0-dev screen pax \
      cpio unzip rsync file wget diffstat
    ```
    
    인코딩 언어 설정
    
    ```c
    sudo apt install -y locales
    sudo locale-gen en_US.UTF-8
    sudo update-locale LANG=en_US.UTF-8 LC_ALL=en_US.UTF-8
    ```
    
- Image baking
    
    SD 카드를 굽기 위해서 Image FIle(.iso)를 build해야한다
    
    이에 앞서 전용 보드 설정 및 리소스 묶음인 .bsp 파일이 하나 필요하다
    
    Digilent에서 쉽게 사용할 수 있다
    
    옛 코드이기 떄문에 clone 후 참고만 권장한다
    
    1. settings.sh을 source로 build한다.
    2. sudo ln -sf /bin/bah /bin/sh를 한다 
        
        ⇒ Petalinux는 sh환경을 권장함
        
    3. (개발이 끝나고 마저 작성)
    4. 참고 블로그 : https://salmon1113.tistory.com/211?category=1520755
    

---

### Dev

- **File system**
    1. vivado : PS 설계 → .xsa 파일 생성
    2. Petalinux Project : BSP
        - path
        
        `⇒ ~/work/BARN-AI-BSP-Petalinux/petalinux/project-spec/meta-user/`
        
        - DTS 수정
        
        `⇒ project-spec/meta-user/recipes-bs메p/device-tree/files/system-user.dtsi`
        
        - 커널 모듈/드라이버
        
        `⇒ project-spec/meta-user/recipes-modules/`  
        
        - 유저 앱
        
        `⇒ project-spec/meta-user/recipes-apps/`  
        
        - 커널 패치/설정
        
        `⇒ project-spec/meta-user/recipes-kernel/linux/`  
        
        - rootfs 패키지 옵션
        
        `⇒ project-spec/meta-user/conf/user-rootfsconfig`
        
        - build 위치
        
        ```c
        cd ~/work/BARN-AI-BSP-Petalinux/petalinux
        source ~/xilinx/petalinux/2025.2/settings.sh
        petalinux-build
        
        => petalinux/images/linux/ (산출물 생성)
        ```
        
- **SD boot check**
    
    SD카드에 넣을 BOOT.BIN이 필요하다.
    
    ```c
    // 프로젝트 root에서 
    source ~/xilinx/petalinux/2025.2/settings.sh
    ```
    
    ⇒ Petalinux 명령어는 기본적으로 PATH에 등록되지 않았기에 명령어 적용 Shell Script를 실행해준다
    
    ```c
    petalinux-package 
    	--boot \
      --fsbl images/linux/zynq_fsbl.elf \         // BOOT.BIN에 FSBL을 넣는다 (FSBL = 처음으로 실행되는 부트로더)
      --u-boot \                                  // BOOT.BIN을 U-Boot로 사용
      --force                                     // 덮어쓰기
    ```
    
    ⇒ 이렇게 images/linux/BOOT.BIN이 생긴다
    
    1. zynq는 BOOT.BIN을 찾는다
    2. FSBL이 실행된다
    3. U-Boot가 실행된다
    4. Linux Kernel이 실행된다
    - fpga options
        
        FPGA의 비트스트림을 포함시키는 옵션이다
        
        ⇒ PS와 PL이 한 영역에 존재한다. 이때 전원이 켜져도 PL은 비어있을 수 있다.
        
        즉 `—fpga` 를 설정하면 FSBL이 실행되면서 PL에 bitstream을 올린다
        
    
    이제 다음 세 파일을 SD카드로 옮겨야 한다
    
    ```c
    BOOT.BIN
    image.ub
    boot.scr
    ```
    
    - FAT32 규격
        
        32GB 이상의 USB 메모리 등은 FAT32 포맷으로 윈도우는 이 포맷을 지원하지 않는다
        
        file allocation table을 의미하는 FAT은 다음과 같은 주소를 가진다
        
        !image.png
        
        **특징**
        
        - 28비트만 클러스터 주소로 가진다
        - 파일 할당 테이블을 통해 클라스터 단위로 파일을 관리한다, 이 테이블은 예비용까지 2개를 보유한다
            - 클러스터 주소
                
                특정 클러스터를 가리키는 주소이다
                
                클러스터는 뭘까?
                
                디스크는 섹터 (512byte)단위로 나뉘어지는데, FS는 이 Sector를 몇 개씩 묶어서 클러스터라는 더 큰 단위로 관리한다
                
                ⇒ Sector * 8 = 4KB cluster
                
                OS는 파일을 저장할 때 Sector가 아닌 Cluster 단위로 공간을 할당한다
                
                FAT32의 Cluster addr은 다음 cluster를 가리키고, 이는 파일이 흩어져 있어도 체인처럼 연결하는 역할을 한다
                
        
        **용량**
        
        - 단일 파일 최대 4GB - 1Byte 크기를 가진다
        - 파티션은 최대 2TB까지 사용 가능하다
            - Partition
                
                하나의 물리적 disk를 논리적으로 나눈 영역이다
                
                1TB HDD가 있다고 할 때 물리적으로는 하나지만 파티션을 활용하여 500GB씩 두 개로 나누어 사용할 수 있다
                
        
        **단점**
        
        - 저널링이 없다면 데이터 손상 위험이 있다
            - 저널링
                
                파일을 변경하기 전에, 실행 목록을 미리 기록해두는 것이다
                
                파일 저장 과정은 복잡하다
                
                1. new data를 disk에 쓴다
                2. FAT을 업데이트한다
                3. file size info를 업데이트한다
                4. ….
                
                여기서 중간 과정에서 문제가 생기면 데이터는 그대로 손상된다.
                
                그렇기 떄문에 저널링으로 기록을 통해 문제 발생에 대해 대비한다.
                
                문제가 발생하면 저널링을 보고 다시 이전 작업으로 undo하면 되기 때문이다
                
        - 암호화나 보안이 없다
- **SD card**
    
    ```c
    // SD 카드 드라이브 문자 확인
    powershell.exe -NoProfile -Command "Get-Volume | Select DriveLetter,FileSystemLabel,FileSystem,SizeRemaining,Size"
    
    // 드라이브 문자가 E: 라면
    sudo mkdir -p /mnt/e
    
    // 드라이브 문자 E에 파일을 마운트한다
    sudo mount -t drvfs E: /mnt/e
    ls /mnt/e
    
    // project 내에 폴더를 E에 복사한다
    cp -v images/linux/BOOT.BIN "$SD"/
    cp -v images/linux/image.ub "$SD"/
    cp -v images/linux/boot.scr "$SD"/
    
    sync
    ```
    
    SD카드를 보드에 꼽고 전원을 인가하고 Uart를 통해 로그를 확인할 수 있다
    
    - 사용 프로그램 : Putty
- **Environment  Setting**
    
    Petalinux는 특정 개발 환경에 최적화되어 있어 권장되지 않는 OS나 설정으로 진행할 경우 문제가 매우 많이 발생한다. 이는 디버깅 시간 연장으로 이어진다 
    
    그렇기에 초기 환경 설정이 개발 성공을 좌우할 만큼 중요하다
    
- **BSP dev standard flow**
    
    부팅 체인 → 커널/DT → rootfs → HW access → App/Debuging
    
    현재 프로젝트 개발 Flow
    
    ```c
    1. PS-only 부팅 성공
    2. UART 콘솔 확인
    3. rootfs debug 환경 구성
    4. 기존 장치 확인: /proc, /sys, /dev, dmesg
    5. PL에 AXI GPIO 추가
    6. XSA 재import + device tree 확인
    7. libgpiod/UIO로 GPIO 제어
    8. user app recipe 추가
    9. out-of-tree kernel module 추가
    10. custom AXI IP + driver
    11. release용 rootfs 정리
    ```
    
    - 자주 사용하는 명령어
        
        ```c
        source ~/xilinx/petalinux/2025.2/settings.sh // 환경 로드
        petalinux-create -t project --template zynq -n petalinux // 프로젝트 생성
        
        petalinux-create -t project -s board.bsp // BSP 파일에서 생성
        petalinux-config --get-hw-description /path/to/xsa_dir // .xsa 임포트
        
        petalinux-config -c rootfs // rootfs 설정
        petalinux-config -c kernel // kernel 설정
        petalinux-config -c u-boot // U-Boot 설정
        
        petalinux-build // 전체 빌드
        
        petalinux-build -c device-tree -x clean // 특정 컴포넌트 클리어 후 빌드
        
        petalinux-boot --qemu --kernel // QEMU 테스트
        
        petalinux-build -c device-tree // dts 수정 후
        petalinux-build
        ```
        
    
    ```c
    petalinux-config -c rootfs
    ```
    
    실행으로 앞으로 보드에서 사용할 네크워크, 프로토콜 등을 설정해 줘야 한다
    
- **petalinux-config -c rootfs**
    
    boot를 성공하고 처음에는 initramfs를 사용했다.
    
    이는 image.ub안에서 사용 시 RAM에 잠깐 올라가서 쓰이는 rootfs이기 때문에 빠르지만 휘발된다
    
    그러므로 임시 fs대신에 직접 rootfs를 구성해주어야 한다
    rootfs는 영구적으로 데이터를 저장한다, 또란 SD 카드를 boot partition과 root partition으로 나누어 완전한 리눅스 시스템을 만들 수 있다
    
    - extra info
        - initramfs
            - 부팅 시 RAM에 올라가는 임시 fs
            - 데이터 휘발성
            - 부팅 과정에서 HW init이나 rootfs mount를 위해 사용한다
            
        - SD card partition
            - Boot Partition
                - size : 100MB ~ 500MB
                - fs : FAT32
                - role : bootloader, kernel img, dts
            - Root Partition
                - size : SD 카드 용량 - Boot Partition size
                - fs : ext4
                - role : rootfs
    
    initramfs에서 fs를 config 하지 않고 build를 사용하면 BOOT, image 밖에 생성되지 않았는데
    
    fs를 EXT4로 선택했을 경우
    
    - u-boot.bin: 부트로더
    - image.ub: 커널 이미지
    - boot.scr: u-boot 명령 스크립트
    - rootfs.tar.gz: 루트 파일 시스템 아카이브
    - rootfs.ext4: ext4 포맷의 루트 파일 시스템 이미지
    
    다음과 같은 파일들이 생성된다
