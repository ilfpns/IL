Input/Ouput ConTroL : 하드웨어 디바이스에 대해 제어 명령을 보내는 시스템콜

```c
ioctl(fd, 명령어, 인자)
```

- file descriptor (fd) : 열린 파일을 가리키는 정수 번호
- 인자 : 전달 명령어 자료형 (ex : int → int 형 명령어 전답)

read/write : 데이터 스트림 (순차적)

ioctl : 명령 + 데이터 (비순차적, 특정 동작 지시)

---

실무로 해보기

```c
MYDRV_IOCTL_SET_VALUE  → 값 설정
MYDRV_IOCTL_GET_VALUE  → 값 읽기
MYDRV_IOCTL_RESET      → 초기화
```

```c
cat > /embedded-linux-qemu-labs/driver/prioctl/myioctl.h << 'EOF'
#ifndef MYDRIVER_H
#define MYDRIVER_H

#include <linux/ioctl.h>

/* magic number - 다른 드라이버와 충돌 방지용 */
#define MYDRV_MAGIC 'M'

/* ioctl 명령어 정의 */
#define MYDRV_IOCTL_SET_VALUE  _IOW(MYDRV_MAGIC, 1, int)  /* 값 설정 */
#define MYDRV_IOCTL_GET_VALUE  _IOR(MYDRV_MAGIC, 2, int)  /* 값 읽기 */
#define MYDRV_IOCTL_RESET      _IO(MYDRV_MAGIC,  3)        /* 초기화 */

#endif
EOF
```

헤더파일이다. 

point

- ioctl 헤더 호출
- magic number 설정
    - 다른 드라이버와 충돌나지 않도록 구분
    - `Documentation/userspace-api/ioctl/ioctl-number.rst`  밑에 코드와 안 겹치는 걸로 선정해야 함
- 명령어 정의
    
    ```c
    _IO   = I/O, 데이터 전달 없음 (단순 명령)
    _IOR  = I/O Read  → 드라이버에서 유저로 데이터 전달
    _IOW  = I/O Write → 유저에서 드라이버로 데이터 전달
    _IOWR = I/O Write/Read → 양방향
    ```
    

```c
cat > /embedded-linux-qemu-labs/driver/prioctl/ioctldriver.c << 'EOF'
#include <linux/module.h>
#include <linux/fs.h>
#include <linux/cdev.h>
#include <linux/uaccess.h>
#include "myioctl.h"

#define DRIVER_NAME "mydriver"

static int major;
static struct cdev my_cdev;
static int stored_value = 0;

static int my_open(struct inode *inode, struct file *file)
{
    return 0;   /* 열기만 하면 됨, 로그도 필요 없음 */
}

static int my_release(struct inode *inode, struct file *file)
{
    return 0;   /* 닫기만 하면 됨 */
}

static long my_ioctl(struct file *file, unsigned int cmd, unsigned long arg)
{
    int val;

    switch (cmd) {

    case MYDRV_IOCTL_SET_VALUE:
        if (copy_from_user(&val, (int __user *)arg, sizeof(int)))
            return -EFAULT;
        stored_value = val;
        pr_info("mydriver: SET_VALUE = %d\n", stored_value);
        break;

    case MYDRV_IOCTL_GET_VALUE:
        if (copy_to_user((int __user *)arg, &stored_value, sizeof(int)))
            return -EFAULT;
        pr_info("mydriver: GET_VALUE = %d\n", stored_value);
        break;

    case MYDRV_IOCTL_RESET:
        stored_value = 0;
        pr_info("mydriver: RESET, value = %d\n", stored_value);
        break;

    default:
        return -EINVAL;
    }

    return 0;
}

static const struct file_operations my_fops = {
    .owner          = THIS_MODULE,
    .open           = my_open,       /* 필수 */
    .release        = my_release,    /* 필수 */
    .unlocked_ioctl = my_ioctl,
};

static int __init mydriver_init(void)
{
    dev_t dev;
    alloc_chrdev_region(&dev, 0, 1, DRIVER_NAME);
    major = MAJOR(dev);
    cdev_init(&my_cdev, &my_fops);
    cdev_add(&my_cdev, dev, 1);
    pr_info("mydriver: loaded! major=%d\n", major);
    pr_info("mydriver: mknod /dev/mydriver c %d 0\n", major);
    return 0;
}

static void __exit mydriver_exit(void)
{
    cdev_del(&my_cdev);
    unregister_chrdev_region(MKDEV(major, 0), 1);
    pr_info("mydriver: unloaded!\n");
}

module_init(mydriver_init);
module_exit(mydriver_exit);

MODULE_LICENSE("GPL");
MODULE_AUTHOR("sehyeon");
MODULE_DESCRIPTION("ioctl driver");
EOF
```

- 테스트
    
    ```c
    cat > /embedded-linux-qemu-labs/driver/prioctl/test_ioctl.c << 'EOF'
    #include <stdio.h>
    #include <fcntl.h>
    #include <unistd.h>
    #include <sys/ioctl.h>
    #include "myioctl.h"
    
    int main()
    {
        int fd, val;
    
        fd = open("/dev/mydriver", O_RDWR);
        if (fd < 0) {
            perror("open failed");
            return 1;
        }
        printf("opened /dev/mydriver\n");
    
        /* SET_VALUE 테스트 */
        val = 42;
        ioctl(fd, MYDRV_IOCTL_SET_VALUE, &val);
        printf("SET_VALUE: %d\n", val);
    
        /* GET_VALUE 테스트 */
        val = 0;
        ioctl(fd, MYDRV_IOCTL_GET_VALUE, &val);
        printf("GET_VALUE: %d\n", val);
    
        /* RESET 테스트 */
        ioctl(fd, MYDRV_IOCTL_RESET, 0);
        printf("RESET done\n");
    
        /* RESET 후 GET_VALUE */
        ioctl(fd, MYDRV_IOCTL_GET_VALUE, &val);
        printf("GET_VALUE after RESET: %d\n", val);
    
        close(fd);
        return 0;
    }
    EOF
    ```
    
    ```c
    cp /embedded-linux-qemu-labs/driver/prioctl/ioctldriver.ko \
       /embedded-linux-qemu-labs/rootfs/
    
    cp /embedded-linux-qemu-labs/driver/prioctl/test_ioctl \
       /embedded-linux-qemu-labs/rootfs/
    
    # cpio 재생성
    cd /embedded-linux-qemu-labs/rootfs
    find . | cpio -H newc -o > ../rootfs.cpio
    
    # uboot 이미지
    cd /embedded-linux-qemu-labs
    ./u-boot/tools/mkimage -A arm -O linux -T ramdisk \
      -d rootfs.cpio rootfs.cpio.uboot
    ```
    
- flow
    
    ```c
    [호스트 (WSL)]
        myioctl.h 작성        ← ioctl 명령어 정의
        ioctldriver.c 작성    ← 드라이버 소스
        make                  ← 크로스 컴파일 → ioctldriver.ko
        test_ioctl.c 작성     ← 유저 테스트 프로그램
        gcc -static           ← 크로스 컴파일 → test_ioctl
        rootfs에 복사
        cpio + mkimage        ← rootfs.cpio.uboot 재생성
            ↓
    [QEMU (ARM)]
        insmod ioctldriver.ko ← 드라이버 로드
        mknod /dev/mydriver   ← 디바이스 파일 생성
        /test_ioctl           ← 유저 프로그램 실행
        
        
       
    ioctl(fd, cmd, arg)
           ↑    ↑    ↑
           |    |    +-- 데이터 포인터 (SET/GET) 또는 0 (RESET)
           |    +------- 명령 번호 (myioctl.h에서 정의한 것)
           +------------ 어떤 디바이스?
    ```
    
    ```c
    opened /dev/mydriver      ← open() → my_open() 호출
    SET_VALUE: 42             ← 유저가 42 설정
                                 copy_from_user(&val, arg, 4)
                                 stored_value = 42
    GET_VALUE: 42             ← 드라이버에서 42 읽어옴
                                 copy_to_user(arg, &stored_value, 4)
    RESET done                ← stored_value = 0
    GET_VALUE after RESET: 0  ← 0 확인
    ```
    

```c
_IOW('M', 1, int) → 32비트 숫자 자동 생성
 ↑    ↑  ↑   ↑
방향 매직 번호 타입크기

[31:30] 방향: 01 = IOW (유저→커널)
[29:16] 크기: sizeof(int) = 4
[15:8]  매직: 'M' = 0x4D
[7:0]   번호: 1
```

- 링킹 선택
    
    dynamic : 필요할 떄 주소로 가서 헤더 참조
    
    static : 컴파일 시 실행 파일에 라이브러리 포함
    
    ⇒ static 선택 : rootfs에 glibc lib이 없기 때문, buildroot로 만든 rootfs는 lib이 있어서 dynamic이 가능하다
    
    Static linking:
    
    - 바이너리 안에 라이브러리 코드 전부 포함
    - test_ioctl (448KB) ← libc 코드까지 다 들어있음
    - 외부 의존성 없음
    
    Dynamic linking:
    
    - 바이너리는 작음
    - 실행 시 /lib/libc.so.6 같은 파일을 불러서 씀
    - test_ioctl (작음) + /lib/libc.so.6 (공유)
    
    어떤 내용을 알아야 할까?
    
    - `ldd` : 의존성 확인
        - ldd test_ioctl → “statically linked”
    - sysroot 개념
        - crosscompile 시 타깃용 lib 경로
    - 실행 안 됨
        - No such file or directory = no lib
        - 어떤 .so가 없는지 찾아서 rootfs에 추가

- 비교
    
    ```c
    # static 바이너리 크기 
    arm-linux-gnueabihf-gcc -static -o test_ioctl_static test_ioctl.c
    ls -lh test_ioctl
    
    # dynamic으로도 빌드해서 크기 비교
    arm-linux-gnueabihf-gcc -o test_ioctl_dynamic test_ioctl.c
    ls -lh test_ioctl test_ioctl_dynamic
    ```
    
    !image.png
    
    이미지를 보면 
    
    - dynamic : 7.9k
    - static : 438k
    
    로 엄청나게 차이난다.
    
    ```c
    arm-linux-gnueabihf-readelf -d test_ioctl_dynamic | grep NEEDED
    // dynamic bin이 뭘 필요로 하는지 검사
    ```
    

!image.png

- libc.so.6
- ld-linux-armhf.so.3

이 둘은 rootfs의 output/target/lib 밑에 헤더로 존재한다

- `.so`  : 공유 lib이 모아져 있는 확장자 (헤더 모음)
- `.6`  : lib의 주 버전
- libc.so.6 = glibc를 의미한다
