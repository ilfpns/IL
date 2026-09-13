크로스컴파일 환경을 쉽게 구축해주는 도구

rootfs를 쉽게 구성한다

```c
ls /embedded-linux-qemu-labs/buildroot/src/output/images/
```

| 항목 | 우리 방식 | Buildroot 방식 |
| --- | --- | --- |
| rootfs | `-initrd rootfs.cpio` (RAM에 올림) | `-drive file=rootfs.ext2,if=sd` (SD카드 에뮬레이션) |
| root | `root=/dev/ram` | `root=/dev/mmcblk0` (SD카드) |
- ext2를 SD 카드로 마운트하는 방식

```c
cd /embedded-linux-qemu-labs/buildroot/src/output/images

PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin \
./start-qemu.sh --serial-only
```

### 비교

- 현재 방식 : QEMU → RAM에 rootfs 올림 → 커널이 RAM에서 실행
- buildroot 방식 : QEMU → SD카드 (가상) → 커널이 SD 에서 실행

우리 방식    = USB 메모리에서 부팅 (임시, 빠름, 전원 끄면 사라짐)
Buildroot    = 하드디스크에서 부팅 (영구, 실제 보드와 동일)

### initrd

커널이 부팅할 때 rootfs 마운트 전에, 임시로 쓰는 file system

```c
커널 시작
    ↓
initrd(임시 rootfs) 마운트   ← RAM에 올라감
    ↓
/init 실행 (초기화 작업)
    ↓
진짜 rootfs로 전환 (switch_root)
    ↓
시스템 정상 동작
```

### 수동/자동

우리가 직접 만든 rootfs는 

아래에 있는 rootfs가 우리가 직접 만든 수동 rootfs이다. 뭔가 파일이 비교적 적다

!image.png

아래에 있는 rootfs는 buildroot에서 만들어진 FHS를 완벽하게 준수하는 구조이다.

!image.png

```c
ps aux // 실행중인 프로세스
cat /etc/os-release // buildroot 버전
ip addr // 네트워크 상태 (DHCP로 IP 받은 거 확인)
free // 메모리 사용량
```
