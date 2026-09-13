### what is buildroot

일단 너무 오랜만에 이걸 해서 감이 없어졌다

buildroot는 crosscompiler 환경을 쉽게 구축 할 수 있도록 도와주는 오픈소스 도구이다

YocTo와 buildroot 둘 다 크로스컴파일러이다

![image.png](attachment:67723fb0-d384-420a-8fb0-f459f9f27136:image.png)

걍 빠르고, 경량화 쉽고, 간단하다! 이렇기 때문에 쓰는거다 ㅇㅇ

### 클론 받기

```c
cd /embedded-linux-qemu-labs/buildroot
git clone https://github.com/buildroot/buildroot.git src
```

그 안에 

```c
configs/   ← 보드별 defconfig
package/   ← 패키지 레시피
board/     ← 보드별 커스텀 파일
fs/        ← 파일시스템 생성
output/    ← 아직 없음 (빌드 후 생성)
```

이런 파일이 있고, 이걸 쓴다면? buildroot 하나로 크로스컴파일러 설정이 편해진다

다음과 같은 설정 파일을 보면, 먼가 많이 설정하고 있는 파일이다

```c
cat configs/qemu_arm_vexpress_defconfig

BR2_arm=y                        ← 타겟: ARM
BR2_cortex_a9=y                  ← CPU: Cortex-A9 (우리 QEMU와 동일)
BR2_ARM_ENABLE_NEON=y            ← NEON SIMD 활성화
BR2_ARM_ENABLE_VFP=y             ← 하드웨어 부동소수점

BR2_TARGET_GENERIC_GETTY_PORT="ttyAMA0"  ← 콘솔: ttyAMA0 (우리랑 동일)
BR2_SYSTEM_DHCP="eth0"           ← 부팅 시 eth0 DHCP 자동 설정

BR2_LINUX_KERNEL=y               ← 커널도 같이 빌드
BR2_LINUX_KERNEL_CUSTOM_VERSION_VALUE="6.18.7"  ← 커널 버전
BR2_LINUX_KERNEL_DEFCONFIG="vexpress"           ← vexpress_defconfig 사용
BR2_LINUX_KERNEL_INTREE_DTS_NAME="arm/vexpress-v2p-ca9"  ← DTB

BR2_TARGET_ROOTFS_EXT2=y         ← rootfs를 ext2 형식으로 생성
BR2_TARGET_ROOTFS_EXT2_SIZE="64M"← 64MB 크기

BR2_PACKAGE_HOST_QEMU=y          ← 호스트에 QEMU도 설치
```

첫 빌드 시 하는 일:

- 크로스 컴파일러 다운로드
- 커널 소스 다운로드 (수백MB)
- BusyBox 소스 다운로드
- 기타 패키지 다운로드
- 전부 크로스 컴파일

예상 시간: 30분 ~ 2시간 (네트워크/PC 성능에 따라)

```c
make -j$(nproc)
```

이런 빌드는 

현재 디렉토리 Makefile + 설정 (.config) 기준으로 빌드된다

```c
u-boot 밑에서 하면? 
u-boot 설정 및 Makefile로 빌드

buildroot 밑에서 하면?
buildroot 설정 및 Makefile로 빌드
```

[Add new application in buildroot](https://www.notion.so/Add-new-application-in-buildroot-36c62a53c75780fbbacad982806df270?pvs=21)

![image.png](attachment:f03424d3-8a50-4188-8c85-ced06f7484a2:image.png)

경로로 접근했는데 make가 안 된다

기존에는 특정 dir 밑에 설정들 + Makefile이 문제인 경우가 많았다. 그런데 에러 로그를 보면 경로에 공백이 있다.

```c
echo $PATH
```

를 쳐서 보면 

```c
/mnt/c/Program Files/Git/cmd
/mnt/c/Program Files (x86)/Windows Kits/...
/mnt/c/Program Files/Microsoft VS Code/bin
```

여기 보면 window PATH가 WSL에 그대로 박혀서 공백이 포함된 경로들이 있다.

WSL기본 설정이 windowPATH를 자동으로 퐘시킨다

```c
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin \
make -j$(nproc) 2>&1 | tail -5
```

깨끗한 PATH를 써서 빌드하면 된다

빌드루트는 크로스컴파일러 환경을 쉽게 구축하고, rootfs같이 설정 파일을 간단하게 만들어주는 도구이다

종류는 

buildroot, yocto 이렇게 있다

YocTo는 개인 커스텀이 가능한 OS로 주요 구성 요소 이해가 필요하다
