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
