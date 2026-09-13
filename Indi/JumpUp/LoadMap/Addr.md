#### 시작

```c
qemu-system-arm \
  -M vexpress-a9 \
  -m 256M \
  -kernel u-boot \
  -device loader,file=../kernel/linux/arch/arm/boot/zImage,addr=0x62000000 \
  -device loader,file=../kernel/linux/arch/arm/boot/dts/arm/vexpress-v2p-ca9.dtb,addr=0x68000000 \
  -device loader,file=../rootfs.cpio,addr=0x69000000 \
  -nographic \
  -audio none
```

- device loader 뒤에 각 파일들을 지정된 Addr에 배치한다
- Base addr: 0x60008000

#### 의문점

```c
alias로 base addr과 offset을 설정하면 매번 입력하지 않아도 되지 않을까?
```

#### 결론

```c
=> setenv kernel_addr 0x62000000
=> setenv dtb_addr 0x68000000
=> setenv rootfs_addr 0x69000000
=> bootz ${kernel_addr} ${rootfs_addr} ${dtb_addr}
```

와 같이 U-Boot shell에서 환경변수를 설정하여 사용할 수 있다

```c
=> setenv bootcmd 'bootz 0x62000000 0x69000000 0x68000000'
=> saveenv
=> boot
```

특수 환경변수로, U-Boot 부팅 후 bootcmd에 저장된 명령어를 자동으로 실행한다

saveenv: 플래시 메모리에 저장하여 비휘발성 저장

```c
/* include/configs/myboard.h */

#define CONFIG_SYS_SDRAM_BASE    0x60000000  /* DRAM 시작 주소 */

#define KERNEL_ADDR   0x62000000
#define DTB_ADDR      0x68000000
#define ROOTFS_ADDR   0x69000000

#define CONFIG_BOOTCOMMAND \
    "bootz 0x62000000 0x69000000 0x68000000"
```

```c
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- myboard_defconfig
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- -j$(nproc)
```

```c
qemu-system-arm \
  -M vexpress-a9 \
  -kernel u-boot \
  -device loader,file=zImage,addr=0x62000000 \
  -device loader,file=vexpress.dtb,addr=0x68000000 \
  -device loader,file=rootfs.cpio,addr=0x69000000 \
  -nographic
```

이렇게 설정한다면, 주소를 참고하여 자동 부팅을 실행한다

#### 자동 실행은?

```c
#!/bin/bash

KERNEL_ADDR=0x62000000
DTB_ADDR=0x68000000
ROOTFS_ADDR=0x69000000

KERNEL=../kernel/linux/arch/arm/boot/zImage
DTB=../kernel/linux/arch/arm/boot/dts/arm/vexpress-v2p-ca9.dtb
ROOTFS=../rootfs.cpio

qemu-system-arm \
  -M vexpress-a9 \
  -m 256M \
  -kernel u-boot \
  -device loader,file=${KERNEL},addr=${KERNEL_ADDR} \
  -device loader,file=${DTB},addr=${DTB_ADDR} \
  -device loader,file=${ROOTFS},addr=${ROOTFS_ADDR} \
  -nographic \
  -audio none
```

- `#!/bin/bash` 로 실행
- `chmod +x run.sh` 권한 부여
- `./run.sh`  실행

#### 실무

```c
Datasheet -> Memmap 확인 -> S/D RAM 시작 주소, 크기 확인 -> 매크로 제작
```

자주 보는 문서

- datasheet: 레지스터 주소, 메모리 맵, 핀 정보
- TRM: 각 IP블록 동작 방식, 초기화 순서
- schematic: 핀 연결 구조
