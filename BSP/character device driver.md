- 서론
    
    2025년에 죽도로 썼던 캐릭터 디바이스 드라이버이다. 또 보니 죽을맛인데, 하나도 기억이 안 난다 : (
    

먼저 BSP sense 파일을 봤다면 알겠지만, Linux는 모든 자원을 파일로 취급한다.

즉, HW도 Linux는 파일로 관리한다.

```c
/dev/ttyAMA0  ← UART (우리가 콘솔로 쓴 것)
/dev/mmcblk0  ← MMC/SD카드
/dev/null     ← 쓰레기통
/dev/mydevice ← 우리가 만들 것
```

이걸 활용해서 character device driver를 짜는데

- character device driver란 뭘까?
    
    OS에서 데이터를 블록 단위가 아닌 문자(Byte) 단위로 I/O를 처리하는 HW장치를 제어하기 위한 SW이다.
    
    ⇒ 하드웨어 입출력을 문자 단위로 처리함
    
    user space에서 직접 데이터를 읽고 쓰는 것이다.
    
    - 여담
        
        다른 종류의 디바이스 드라이버도 존재한다.
        

---

#### 목표

- read() : 문자열 출력
- write() : 받은 데이터를 커널 로그에 출력

```c
mkdir -p /embedded-linux-qemu-labs/mydriver
cd /embedded-linux-qemu-labs/mydriver
```

- QEMU 밑에 디렉토리를 하나 판다

```c
cat > Makefile << 'EOF'
KERNEL_DIR ?= /embedded-linux-qemu-labs/kernel/linux // 이미 설정값이 있다면 설정 X
ARCH := arm // 즉시 대입
CROSS_COMPILE := arm-linux-gnueabihf-

obj-m := mydriver.o

all:
	make -C $(KERNEL_DIR) \
		ARCH=$(ARCH) \
		CROSS_COMPILE=$(CROSS_COMPILE) \
		M=$(PWD) modules

clean:
	make -C $(KERNEL_DIR) \
		ARCH=$(ARCH) \
		CROSS_COMPILE=$(CROSS_COMPILE) \
		M=$(PWD) clean
EOF
```

Makefile에 빌드 설정을 넣어준다.

make하는 데에 필요한 명령어들을 파일에 모아둔 것이 Makefile이다.

```c
#include <linux/module.h>    // 모듈 필수 헤더
#include <linux/fs.h>        // file_operations
#include <linux/cdev.h>      // cdev 구조체
#include <linux/uaccess.h>   // copy_to_user, copy_from_user

#define DRIVER_NAME "mydriver"
#define MSG "Hello from kernel driver!\n"

static int major;            // 디바이스 번호 (커널이 자동 할당)
static struct cdev my_cdev;  // 캐릭터 디바이스 구조체

/* read() 시스템콜 → 이 함수 호출 */
static ssize_t my_read(struct file *file, char __user *buf,
                       size_t count, loff_t *ppos)
{
    int len = strlen(MSG);

    if (*ppos >= len)        // 이미 다 읽었으면 0 반환
        return 0;

    if (copy_to_user(buf, MSG, len)) // 커널→유저 공간 복사
        return -EFAULT;

    *ppos += len;
    pr_info("mydriver: read() called\n");  // 커널 로그 출력
    return len;
}

/* write() 시스템콜 → 이 함수 호출 */
static ssize_t my_write(struct file *file, const char __user *buf,
                        size_t count, loff_t *ppos)
{
    char kbuf[128] = {0};
    int len = min(count, sizeof(kbuf) - 1);

    if (copy_from_user(kbuf, buf, len))  // 유저→커널 공간 복사
        return -EFAULT;

    pr_info("mydriver: write() received: %s\n", kbuf);  // 커널 로그
    return count;
}

/* open() 시스템콜 → 이 함수 호출 */
static int my_open(struct inode *inode, struct file *file)
{
    pr_info("mydriver: open() called\n");
    return 0;
}

/* close() 시스템콜 → 이 함수 호출 */
static int my_release(struct inode *inode, struct file *file)
{
    pr_info("mydriver: release() called\n");
    return 0;
}

/* 시스템콜 → 함수 매핑 테이블 */
static const struct file_operations my_fops = {
    .owner   = THIS_MODULE,
    .read    = my_read,
    .write   = my_write,
    .open    = my_open,
    .release = my_release,
};

/* insmod 시 실행 */
static int __init mydriver_init(void)
{
    dev_t dev;

    // 디바이스 번호 자동 할당
    alloc_chrdev_region(&dev, 0, 1, DRIVER_NAME);
    major = MAJOR(dev);

    // cdev 초기화 및 등록
    cdev_init(&my_cdev, &my_fops);
    cdev_add(&my_cdev, dev, 1);

    pr_info("mydriver: loaded! major=%d\n", major);
    pr_info("mydriver: run: mknod /dev/mydriver c %d 0\n", major);
    return 0;
}

/* rmmod 시 실행 */
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
MODULE_DESCRIPTION("Simple character device driver");
```

file_operations에서 함수를 연결한다.

read호출 시 my_read 호출 같은 식이다.

```c
static const struct file_operations my_fops = {
    .read    = my_read,    
    .write   = my_write,
    .open    = my_open,
    .release = my_release,
};
```

또 user ↔ kernel space는 서로 간섭 할 수 없다. 이를 위해선 함수를 써야한다.

```c
copy_to_user(buf, MSG, len)    // 커널 → 유저 공간
copy_from_user(kbuf, buf, len) // 유저 → 커널 공간
```

디바이스마다 major와 minor 번호가 할당된다. 이는 장치를 식별하는 데에 쓰인다.

- QEMU 활용
    
    ![image.png](attachment:48b90a6d-bb87-483a-bc0d-3df0f84ccda8:image.png)
    
    .ko 파일을 rootfs에 넣어서 활용 할 수 있다.
    

- 함수 호출 타이밍
    - __init, __exit
        - init :  `insmod` 입력 시 딱 한 번 실행된다.
        - exit : `rmmod` 입력 시 딱 한 번 실행된다.
        
         
        핵심
        
        1. __init에서 alloc_chrdev_region(결과를 저장할 변수, 시작 minor번호, minor번호 개수, 이름)으로 major를 받을 수 있음
        2. cdev_init()으로 cdev 구조체에 fops연결
        3. cdev_add()으로 kernel에 cdev 등록 ( → /dev에서 접근 가능)
        4. cdev_del()으로 등록 해제
        5. unregister_chrdev_region()으로 major 반납
    - dev_t  dev
        
        32비트 정수이다.
        
        - major + minor를 합친 것이다.
        
        ```c
        MAJOR(dev)          // dev에서 major 번호 추출
        MINOR(dev)          // dev에서 minor 번호 추출
        MKDEV(major, minor) // major + minor → dev_t 합치기
        ```
        
    - cdev
        
        character device struct이다.
        
        다음 내용들을 포함한다.
        
        ```c
        struct cdev {
            struct kobject kobj;
            struct module *owner;
            const struct file_operations *ops;  // ← 핵심
            struct list_head list;
            dev_t dev;
            unsigned int count;
        };
        ```
        
        나중에 
        
        cdev_init, cdev_add 시에 사용된다
        
    - open, release
        - user가 open, close syscall 호출 시 함수가 호출된다.
        
        close()가 항상 즉시 드라이버 release를 부르지 않는 경우에 대비한다.
        
        파일 참조가 0일 때 release가 호출된다.
        
        실무 open()
        
        - HW init
        - clock 활성화
        - 전원 켜기
        - 동시 접근 방지 (mutex 설게)
        
    - read
        
        드라이버 → 유저 데이터 전달
        
        ```c
        struct file *file   // open()과 같은 file 인스턴스
        char __user *buf    // 유저 공간 버퍼 (여기에 데이터 써줘야 함)
        size_t count        // 유저가 요청한 읽기 크기
        loff_t *ppos        // 현재 파일 읽기 위치 (파일 오프셋)
        
        첫 번째 read(): ppos = 0  → 처음부터 읽기
        두 번째 read(): ppos = 26 → 이미 다 읽었으니 0 반환
        ```
        
        `cat` 같은 명령어 입력 시 open → read → relese
        
    - write
        
        유저 → 드라이버 수신
        
        ```c
        struct file *file        // 동일
        const char __user *buf   // 유저가 쓴 데이터 (읽어와야 함)
        size_t count             // 유저가 쓴 데이터 크기
        loff_t *ppos             // 파일 오프셋
        ```
        
        `echo`같은 명령어 입력 시 open → read → relese
