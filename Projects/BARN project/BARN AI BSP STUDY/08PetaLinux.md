### Session ID/PASSWD

---

- ID : petalinux
- PASSWD : dsm2026

### General

---

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
    
    사실 rootfs는 petalinux-build를 하면서 자동으로 만들어진다 
    
    ~~이건 모르는 사실이었담~~
    
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
    
    SD 첫 번째 파티션은 FAT으로 500MB이고, 두 번째 파티션은 3GB를 권장하며 EXT4여야한다
    
    - EXT2 ~ 4
        
        EXT2
        
        Linux는 파일 위치 cache를 가지는 inode 하나당 하나의 파일을 저장한다, 가장 중요한 특징은 파일을 저장할 때 Block Mapping 방식을 사용한다
        
        inode 1번부터 12번 가지는 하나의 파일에 매핑을 한다. 하지만 파일은 수천개가 넘으므로 13번 inode부터는 Block mapping 식으로 하나의 inode가 여러개의 블록을 가리키도록 하여 많은 파일들을 효율적으로 저장할 수 있도록 한다
        
        - 의문
            
            1번 inode부터 Block Mapping을 하면 안 되는 것인가?
            
            ⇒ 1 ~ 12번은 많이 접근되는 파일의 inode가 위치한다
            
        
        EXT3는 EXT2에서 저널링  기능을 더한 것이다.
        
        저널링은 간단히 복구가 빠르도록 파일의 변경 사항을 기록하는 것이다.
        
        ETX4는 EXT3와 비슷하지만 block mapping이 아닌 exxtents 트리를 이용하여 파일을 관리한다
        
        B+ tree는 간단하게 한 노드당 정련된 k개의 값을 가질 수 있고, 자신 노드는 k + 1개의 노드 개수를 가진다.
        
        Extents 특징
        
        - 실제 데이터는 오직 leaf node에만 위치한다 (단말 노드)
        - leaf node가 서로 연결된다
        - 항상 탐색 시간을 최소화 하기 위해서 balance를 맞춘다
        
         
        
        - 대소비교 방식으로 빠르게 값을 탐색할 수 있다
        - Log N의 속도를 보인다
        
    
    ```c
    // linux
    lsblk 
    
    // window
    winget install --interactive --exact dorssel.usbipd-win
    usbipd list
    ```
    
    현재 SD의 용량을 찾아서 NAME을 확인한다
    
    !image.png
    
    ```c
    usbpid list // 현재 usb 확인
    usbpid bind --busid 2-2 // list에서 나온 id로 등록
    
    usbipd attach --wsl --busid 2-2 // wsl에 등록
    usbipd detach --busid 2-2 // wsl에서 해제
    ```
    
- PL AXI gpio .xsa file
    
    이제 driver, dts부분을 만져야한다. 그렇다면 이를 자동으로 만들어줄 새 PL의 .xsa가 필요하다
    
    그러므로 Vivado에서 AXI gpio를 PL로 하나 구현해서 .xsa로 가져올 예정이다. 
    
    그렇다면 driver를 왜 구현해야할까?
    OS가 없는 baremetal환경에서는 레지스터에 직접 접근한다. 그렇다면 메모리 주소를 직접 읽는다는 것을 의미한다. 이는 아무 값이나 HW에 마음대로 접근할 수 있다는 것이다.
    
    그건 에바지.
    
    그렇기 때문에 driver(응용프로그램)는 gpio에서 아무 값이 HW 레지스터를 직접 못 건드리도록 한다.
    
    User space와 kernel space의 분할로 생각할 수 있다
    
    |  | 베어메탈 펌웨어 | BSP 드라이버 |
    | --- | --- | --- |
    | 접근 | 레지스터 직접 | 드라이버 통해서 |
    | 보호 | 없음 | MMU가 앱 격리 |
    | 공유 | 불가 | 여러 앱이 안전하게 공유 |
    | 이식성 | 주소 하드코딩 | 표준 API(gpiod)로 보드 바뀌어도 앱 그대로 |
    | 리눅스 기능 | 없음 | 인터럽트, sleep, 전원관리 등 커널 서브시스템 연동 |
    
    그러므로 새 .xsa를 만들 것이다.
    
    - New xsa
        
        일단 AXI와 GPIO에 대해 알아야 할 필요가 있다
        
        - GPIO
            - 범용 입출력 핀이다
        - AXI
            - 다중 채널 버스로 읽기/쓰기에 최적화 되어 있는 버스이다
            
            AHB 같은 고속 버스의 경우 채널이 버스로 구성되어 독립적 작동이 불가하지만, AXI는 채널이 도입되어 독립적으로 작동이 가능하다
            
            AHB는 앞의 데이터가 수신될 때 까지 기다려야 하기 때문에 저속 디바이스가 큰 문제였지만, AXI는 각자 독립된 채널이 동작하므로 속도 문제를 해결했다.
            
            1. valid : 유효 data와 control info 사용 가능 여부
            2. ready : data가 수락 가능한지 여부
            3. last : transaction에서 마지막 데이터 아이템의 전송을 의미
            4. BRESP : transaction이 정상적으로 완료됨
        
        !image.png
        
        Open block design → ADD IP (+) → AXI_GPIO로 새 IP (하드웨어 블럭)을 추가할 수 있다
        
        그 다음에 자동으로 auto connection을 이용하면 IP들이 자동으로 연결된다
        
    
    새로운 xsa를 가지고 xsa교체 → petalinux-build→ 새 BOOT.bin 생성을 하고나면 dts가 생길 것이다.
    
    ```c
    grep -A8 axi_gpio components/plnx_workspace/device-tree/device-tree/pl.dtsi
    
    /*
    components/plnx_workspace/device-tree/device-tree/pl.dtsi 이 주소 밑에서 
    axi_gpio를 찾아서 밑에 8줄까지 보면 편하다
    */
    ```
    
    이제 dts도 있고 dtsi에서 확인도 할 수 있다. 그렇다면 driver를 짤 차례이다. 원래는 file_operations같은 거 보면서 귀찮게 하나하나 짰어야 했는데…
    
    킹갓 petalinux는 이런 귀찮은 기능들에 대해 많은 것을 지원한다. 
    
    ```c
    cd ~/work/BARN-AI-BSP-Petalinux/petalinux
    petalinux-create -t modules --name axi-gpio-drv --enable
    ```
    
    이후 
    
    ```c
    project-spec/meta-user/recipes-modules/axi-gpio-drv/
    // 이 경로에서 files밑에 c에다가 driver를 짜면 된다.
    // .bb 파일은 bitbake 빌드 규칙 레시피이다.
    ```
    
    !image.png
    
    자동 생성 dtsi이다.
    
    현재 GPIO가 주소 41200000에 연결되어 있음을 볼 수 있다
    
- TPG
    
    Video Test Pattern Generator이다. CAM의 실질적 값을 받는 게 아니고, 외부 입력이 아니라 TPG가 알아서 컬러바 픽셀을 만들어서 보낸다.
    
    이는 V4L2에 HW 카메라처럼 잡히고 device에서도 접근 가능하며, v4l2-ctl로 캡쳐까지 가능하다 (실제 사진은 아니겠지만) 
    TPG는 즉 카메라 제작 전에 파이프라인을 구성하고 뒷 단을 개발하기 위해 임시로 끼워넣는 IP 모듈이다.
    
    새로운 .xsa가 왔기 때문에 교체해준다
    
    ```jsx
    source ~/xilinx/petalinux/2025.2/settings.sh
    petalinux-config --get-hw-description=~/work/xsa/design_TPG.xsa
    ```
    
    몰랐는데 지금 받은 TPG에 제어 레지스터가 하나도 없다 .xsa를 열었는데 DTS가 수정된 부분이 없는 것이다.
    
    1. 레지스터 없음
    2. 파라미터 고정
    3. 핀 하드와이어링
    
    그렇다면 VDMA를 쓰면 된다. Video DMA는 비디오 프레임 전용 DMA 엔진이다.
    일반적인 AXI DMA가 임의 크기 버퍼를 한 번씩 옮긴다면 VDMA는 가로 * 세로 크기의 프레임을 계속 반복해서 옮긴다.
    
    즉 설계부터 S2MM (Stream to memory) 채널만 사용한다 ⇒ 받기만 하고 보내지 않는다
    
    .hwh파일을 열어보면 (HW가 어떻게 구성되는지 알려주는 파일)
    
    - H_ACTIVE = 64
    - V_ACTIVE =48
    - TDATA_NUM_BYTES = 3
    
    으로 64*48에 초당 3바이트 ( = 24bits)르 전송하는 것을 알게되었다.
    이제 이 안에 채널 순서가 중요하다. R-G-B 순서인지 B-G-R 순서인지 알기위해 채널 순서가 중요하다
    
    ```jsx
    petalinux-config -c kernel // DMA test for client 설정
    
    petalinux-build
    petalinux-package --boot --fsbl --fpga --u-boot --force 
    
    petalinux-boot --qemu --kernel
    dmesg | grep -iE 'vdma|xilinx'
    ```
    
    여러 설정을 했고,,, 특정 모듈을 빌드했다.
    
    ```jsx
    petalinux-build -c vdma-capture 
    // vdma-capture 빌드
    ```
    
    bitbake는 레시피 (.bb파일)들을 전부 스캔하여 PN (package name)이 우리가 요청한 이름과 같은 레시피부터 찾는다.
    
    - gpio : axi-gpio-drv.bb ← 레시피 이름
    - 아래 사진을 보면 vdma-capture가 이름인 레시피를 찾지 못한 것이다.
    
    !image.png
    
    레시피들을 확인해보자.
    
    ```jsx
    ls project-specs/meta-user/recipes-modules/
    
    // module이 없으므로 생성
    petalinux-create -t modules --name vdma-capture --enable 
    project-specs/meta-user/recipes-modules/ <- 이 경로에 하나의 모듈이 추가됐을 것이다
    ```
    
    VDMA는 실제로 커널에 올라가는 모듈이 되어야 한다 →
    
    `petalinux-create -t modules —name <이름> —enable` 로 모듈을 생성해야한다
    
    ⇒ .bb 레시피 파일 + Makefile 생성
    
    ```jsx
    static const struct of_device_id vdma_capture_of_match[] = {
            { .compatible = "barn,vdma-capture", },
            { },
    };
    ```
    
    DTS와 compatible을 맞추기 위해 이를 수정한다
    
    DTS는 `project-spec/meta-user/recipes-bsp/device-tree/files/system-user.dtsi` 이 경로에 있다
    
    - DTS 해석
        
        ```jsx
        &amba_pl { // 이미 정의된 amba_pl에 내용 추가
        
            vdma_capture: vdma_capture@0 { 
        				// vdma_capture <- 다른 파일에서 지정 가능한 이름
        				// vdma_capture@0 <- 이름@주소
        				// TPG처럼 레지스터가 없는 게 아니라 가상 소비 노드이므로 reg가 없다 
        				
                compatible = "barn,vdma-capture"; 
        	      // probe을 위한 of_device_table과 맞춘 매칭 문자열
                
                dmas = <&axi_vdma_0 1>;
        	      // axi_vdma_0 <- DMA 제공자가 이 노드임
        	      // 1 <- dma-cells에서 요구한 인자 
                
                dma-names = "s2mm";
                // 이름 지정, stream to memory -> s2mm
            };
        };
        ```
        
    
    DTS오류를 찾기 위해서 
    
    1. `petalinux-build -c device-tree`
        1. 중간에 오류가 터졌는데 
        
        ```jsx
        /home/dsm/work/BARN-AI-BSP-Petalinux/petalinux/build/tmp/work/zynq_generic_7z020-amd-linux-gnueabi/device-tree/2025.1+git/system-user.dtsi 
        ```
        
        여기서 오류가 남; 내 DTS랑 비교해 보니 
        
        ```jsx
        dmas = <&axi_vdma_0, 1>;
        // 이렇게 작성됨
        // dts는 쉼표가 아니라 공백으로 문자를 구분하므로 쉼표가 문제
        ```
        
    2. `petalinux-build -c vdma-capture`
    
    순으로 진행한다
    
    - 회고
        
        **`petalinux-config --get-hw-description` 으로 .xsa를 받아와도 새로고침이 적용되지 않는 오류가 생김**
        
        ```jsx
        petalinux-build -c device-tree -x cleansstate
        petalinux-build -c device-tree
        ```
        
        DTS는 콤마로 구분하지 않음, git 사본과 실제 meta-user 값이 같아야 함
        
    
    빌드가 성공적으로 끝나면 된 것이다
    
    나중에 받은 RGB 순서를 이용해 3bytes씩 잘라서 python으로 순서에 맞게 띄우면 된다
    
    - 새로운 TPG
    FPGA 개발 담당 친구에게 새 .xsa를 받았다. 이제부터 부족했던 개발을 마저 진행한다.
        
        ```jsx
        1. git pull로 새 버전 받기
        2. grep으로 필요한 파라미터 보기
        3. axil_regfile_0의 BUSINTERFACES/baseaddr 확인해서 주소 파악
        4. 레지스터가 뭘 제어하는 파악
        5. axis_vid_mux_0의 select핀 배선 확인
        6. 확인
        ```
        
        새 .hwh 안에 grep해서 INSTANCE를 axil_regfile_0으로 찾아보면 MODULE을 찾을 수 있다. 이 MODULE하나는 IP 하나를 의미한다. 
        
        !image.png
        
        - MODULE ( = IP) : axil_regfile_0
        - BASEADDR : 0x43C00000 ← CPU접근 주소 (BASEADDR ~ HIGHADDR)
        - SIGNAME : 데이터 통로 이름
        
        ```jsx
        <PORT DIR="O" NAME="mux_sel" SIGNAME="axil_regfile_0_mux_sel">
            <CONNECTIONS>
                <CONNECTION INSTANCE="axis_vid_mux_0" PORT="sel"/>
            </CONNECTIONS>
        </PORT>
        ```
        
        - PORT : IP 입출력 핀
            - axil_regfile_0은 mux_sel 이라는 출력 핀이 있다.
            - CONNECTION은 mux_sel이 어디로 이어지는지 나타낸다
                - axis_vid_mux_0 IP의 sel으로 이어진다
                
                ```jsx
                axil_regfile_0
                +--------------------+
                |                    |
                |   mux_sel ---------+-------------------+
                +--------------------+                   |
                                                         |
                                                         ▼
                                              axis_vid_mux_0
                                              +-----------+
                                              | sel       |
                                              +-----------+
                ```
                
        
        근데 image_block.hwh (hardware handoff)에서 base, high, port는 확인했는데, 직접 제어를 위한 offset을 못 찾았다. 
        offset는 component.xml밑에 있다해서 봤는데.. 없다.
        
        그냥 RTL Source를 보려고 했는데 이것도 없다 → FPGA한테 묻기
        
        - 이후
            
            RTL 받음 ㅋㅋㅋ; 오프셋도 찾았다.
            
            - README 안에 HANDOFF (hwh) 파일 설명
            - RTL 안에 verilog에서 직접 보기
            - sw에서 main.c 확인
                
                
            
            QEMU 돌려서 dmesg로 로그를 직접 봐야한다
            
            ```jsx
            dmesg | grep -i vdma-capture  
            find /sys/devices/platform -iname 'vdma_capture*' 
            
            time (echo 1 | sudo tee /sys/bus/platform/devices/pl-bus:vdma_capture@0/capture > /dev/null)
            ```
            
            time에서 두 로그가 구분이 안 되어서 vdma-capture.c가서 수정 후 다시 빌드 → qemu 실행으로 검증하고자 한다
            
        
        이제 offset을 활용한 TPG 제어를 해야한다. → 드라이버 작성
        
        ```jsx
        petalinux-create -t modules --name axil-regfile --enable
        ```
        
        TPG 드라이버를 만들었고 이제 드라이버를 작성할 차례이다.
        
        devmem (드라이버 없이 레지스터를 테스트할 때 사용하는 리눅스 명령어이다)
        
        - attr
        
        ⇒ show/store callback 함수의 시그니처는 커널 API가 고정한다
        
        - driver
            - skel 코드부터 본다 init → exit → probe → remove 등의 기본적 코드부터 본다
            - `~/work/BARN-AI-BSP-Petalinux/petalinux/components/plnx_workspace/device-tree/device-tree` 여기서 pl.dtsi랑 비교해서 볼 수 있다.
            - 여러 dts
                
                ```jsx
                system-user.dtsi (우리가 씀)
                    └─ include: system-conf.dtsi (PetaLinux 관리)
                            └─ include: pl.dtsi (자동생성, .xsa에서 나옴)
                                    └─ include: zynq-7000.dtsi 등 (PS 기본 하드웨어)
                
                → petalinux-build -c device-tree 가 이 전부를 합쳐서 → system.dtb (실제 부팅에 쓰이는 최종본)
                ```
                
            
            ```jsx
            static DEVICE_ATTR_R0(id);
            // probe이 호출될 때 rootfs밑 sysfs에 id 파일이 생긴다
            ```
            
            - 왜 sysfs밑에 id를 만들까?
                - sanity check : 기본적인 동작을 확인하는 것
            
            이를 활용하여 CTRL을 만든다 (sysfs는 시그니처가 있음)
            
            ```jsx
            static ssize_t axil_regfile_ctrl(struct device *dev, struct device_attribute *attr, char *buf) {
                    uint32_t enable, mux;
                    static struct axil_regfile_local *lp = dev_get_drvdata(dev);
            
                    if (sscanf(buf, "%d %d", &enable, &mux) != 2) {
                            return -EINVAL;
                    }
            
                    uint32_t ctrl = (enable & 0x1) | ((mux & 0x1) << 1);
                    iowrite32(ctrl, lp->base_addr + REFG_CRTL);
            }
            static DEVICE_ARRT_RW(ctrl_store);
            ```
            
            - echo에 어떻게 받응할지 store를 통해 구현한다
                - 입력은 sysfs를 통해서
                - store는 단지 현재 sysfs를 통해 들어온 값에 대한 레지스터 반영만 하면 된다
            - cat에 어떻게 반응할지 show를 통해 구현한다
                - store, show 둘 다 함수 선언 시그니쳐를 따라 선언하고 이름도 지켜야 한다
                - ex : ctrl_store, ctrl_show
                - show는 단지 현재 대응하는 sysfs name에 맞게 현재 레지스터 값을 읽어 출력하면 된다
            - probe 함수 안에서 device_create_file을 해줘야 한다
            
            ```c
            rc = device_create_file(dev, &dev_attr_id);
            if (rc) {
            	dev_err(dev, "failed to create sysfs file 'id'\n");
            }
            rc = device_create_file(dev, &dev_attr_ctrl);
            if (rc) {
            	dev_err(dev, "failed to create sysfs file 'ctrl'\n");
            }
            ```
            
            개발 후 axil_regfile 빌드 → 빌드 → qemu 진행
            
- Sensor part
    
    지금 PL이 없는 상황이다 → .xsa가 없다. 그렇다면 BSP는 개발을 못 하나? 그렇지 않다. 
    
    ---
    
    .xsa가 있다면 
    
    - 정확한 레지스터 주소 / 메모리 주소
    - DTS 등을 알 수 있다
    
    .xsa가 없다면
    
    - 센서, 칩에 따른
        - I2C, SPI 같은 통신 프로토콜
    - 제작된 드라이버 확인
    
    일단 SCD4 파트를 개발할 예정이다. SCD4는 리눅스에서 IIO 센서로 분류된다
    
    - IIO : Industrial I/O → ADC가 필요한 센서 (가속도, 조도, 습도)들을 같은 규격으로 관리하는 프레임워크이다.
    
    IIO를 -c kernel에서 따로 켜주면 사용 가능하다.
    
    ```jsx
    petalinux-config -c kernel // 설정 후 적용을 위해 한번 더 실행해야한다
    ```
    
- other driver
    
    센서와 uart가 생긴 버전을 개발할 예정이다.
    
    1. 파일 키워드 찾기
        - `grep -o '<[A-Z]*' image_block.hwh | sort | uniq -c | sort -rn`
        
        !image.png
        
        - 4개의 MEMRANG이 있는데, 이는 IP가 차지하는 주소를 소개하는 키워드가 4개 있다는 뜻이다
    2. 개발 부분 찾기
        - `grep -o '[A-Z_0-9]**I2C[A-Z_0-9]**' image_block.hwh | sort -u`
            - 필요한 키워크를 앞 뒤로 A-Z_0-9 모두 검색을 통해 찾는다
            - uart를 찾는다면 i2c 자리에 uart가 들어갈 뿐이다.
    3. 필요 값 찾기
        - `grep -o 'NAME="PCW_UART1[^"]*" VALUE="[^"]*"' image_block.hwh`
        - 2번을 통해 찾은 키워드를 검색하면 BASEADDR을 고생하지 않고 찾을 수 있다.
        - `grep -o ‘<MEMRANGE[^>]>’  image_block.hwh`
        - MEMRANGE를 전부 볼 수 있다 → BASE_ADDR 확보
    4. 주변 맥락 파악
    
    !image.png
    
    총 4개의 MEMRANGE가 있다. 레지스터가 없으면 제어가 불가하므로 UART, I2C는 건드릴 부분이 없다.
    
    이제 기본으로 추가된 IP 확인과 BASEADDR 확인에 대해 알았다. 이제 제어를 위해 offset 등을 찾을 것이다.
    
    .hwh는 hw배선만 표시하므로 정보를 다 얻었다. 이제 다른 파일을 봐야한다
    
    - docs : 사용자가 작성한 문서에서 offset 확인 가능
    - rtl : 사용자가 작성한 verilog로 직접 확인 가능
    
    이제 새 엑사 적용 및 dts 생성이다
    
    ```c
    // 적용 후
    petalinux-build -c device-tree -x cleanstate
    petalinux-build -c device-tree 
    
    // compatible 확인
    grep -A8 sensor_regs pl.dtsi
    ```
    
    → 모듈 생성 → 드라이버 작성
    
- etc
    - 드라이버가 할 것을 찾기
        - 어떤 역할이 가능한지
    - Petalinux에서 PL로 값을 넘기는 거
    - DMA, AXI같은 부분 개발
        
        
    - 이어하기
        
        !image.png
        
        https://claude.ai/chat/933e1cb8-a27b-46fb-843d-8e07e6a811cc
        
        https://m.blog.naver.com/simula/224108760730
        
        https://velog.io/@tmdtng21/%EB%85%BC%EB%AC%B8-%EB%A6%AC%EB%B7%B0-OpenVLA-An-Open-Source-Vision-Language-Action-Model
        
        https://newhaneul.tistory.com/181
        
        https://kmhana.tistory.com/27
        
        https://www.google.com/search?q=%ED%94%BC%EC%A7%80%EC%BB%AC+AI+%EC%8B%9C%EB%AE%AC%EB%A0%88%EC%9D%B4%EC%85%98+%EB%B0%A9%EB%B2%95&oq=%ED%94%BC%EC%A7%80%EC%BB%AC+AI+%EC%8B%9C%EB%AE%AC%EB%A0%88%EC%9D%B4%EC%85%98+%EB%B0%A9%EB%B2%95&gs_lcrp=EgZjaHJvbWUyBggAEEUYOTIICAEQABgNGB4yCggCEAAYgAQYogQyBwgDEAAY7wUyBwgEEAAY7wUyBwgFEAAY7wXSAQg1Nzc4ajBqN6gCALACAA&sourceid=chrome&ie=UTF-8
        
        !image.png
