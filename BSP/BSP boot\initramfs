### initial RAM filesystem

램 파일 시스템을 초기화 하는 것이다. 그렇다면 커널이 부트되기 전에 필요한 시스템이다. 

커널 ↔ 디스크의 마운트 전에 일어나야 하기 때문이다.

그렇다면? boot/, u-boot/ 밑에 위치해 있을 것이다.

### 역할

1. 커널 로드
    
    ⇒ bootloader가 kernel과 함께 initramfs 이미지를 메모리에 로드한다
    
2. 초기 설정
    
    ⇒ kernel이 실행되며 initramfs를 임시 fs로 지정한다. 이 상태에서 device driver나 rootfs가 mount된다
    
3. rootfs mount
    
    ⇒ initramfs의 작업이 완료되면, 실제 rootfs가 mount된다
    
4. initramfs해제
    
    ⇒ rootfs가 mount되면, initramfs는 메모리에서 해제된다
    

⇒ 임시 fs으로 사용되며, rootfs, driver, fsimage 등을 담는다

!image.png

### 실패

rootfs 마운트 실패 시 initramfs 쉘로 빠진다. 

**원인**

1. 잘못된 UUID or LABEL
2. file system crush
3. change disk partition
4. HW 모듈 누락
5. kernel or initramfs 경로 오류
6. hw 인식 문제

### 해결

- UUID 확인 : initramfs에서 `lsblk -f`로 disk의 UUID를 학인한다 ( → bootloader 설정과 맞는지 확인)
- 모듈 로드 : modprobe으로 직접 kernel module을 로드하고, fs를 수동으로 마운트 가능한지 확인
- fs 검사 : `fsck` 명령어로 무결성을 검사한다
- 재설치
