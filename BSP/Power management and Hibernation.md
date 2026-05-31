## Power management

유휴전력 관리를 가장 큰 목적으로 한다. 기기가 사용되지 않는 시간에 적절히 sleep모드로 유휴전력 관리가 목적이다.

## before..

ACPI 표준에 대해 알아보자

⇒ OS에 의해 process, pc, pcb, hw 등 기기들의 전원상태를 세밀하게 조정하는 표준이다.

전력 효율이 좋아야하는 핸드폰에서 이 표준이 많이 사용된다.

4가지 state 

1. working : 사용중
2. sleeping : 리부트없이 사용 가능
3. soft off : 재사용을 위해 부팅 필요
4. mechanical off : 전원 종료

## Hibernation

이는 메모리 상태를 disk의 swap여역에 저장하여 컴퓨터 reboot 시 그 이미지를 빠르게 부팅하는 기술

부팅을 kernel space에서 진행한다면 Swsusp라고 한다. 

bootloader영역에서 진행하면 Snapshot 부팅이라 불린다.

**flow**

1. 안정적인 이미지를 위해서 모든 프로세스 상태를 Freeze한다.
    - CD-ROM이 동작할 때 이미지를 얻으면, 부팅은 빨라도 바로 CD-ROM이 실행되어 전력 소모가 큼
2. 메모리 영역 점검
    - 이미지를 저장할 메모리가 있는지 확인한다. (이미지를 만들었는데 swap 영역 메모리가 부족하면 낭패)
    - 이미지를 만드는데 필요한 메모리가 충분한지 확인
3. 이미지 생성
4. freeze 종료

- 이 작업은 장비가 사용되지 않는 시간에 이뤄진다.

- 생각
    
    cron으로 일정 주기로 freeze 후 snap
    
    우리 기기에서 dual 보드를 사용하기 때문에 계속해서 snap을 찍으면서 연산 가능?
