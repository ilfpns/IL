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

- adter .xsa build
    
    .xsa빌드가 끝나고 생기는 변화 : 
    
    -  
