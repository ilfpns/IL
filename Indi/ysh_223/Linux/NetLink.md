## 프로세스 간 통신 [[Linux]]

⇒ IPC (Inter-Process Communication)

⇒ 프로세스 간 데이터 통신을 의미한다

## Netlink

⇒ 커널과 사용자 영역 간 데이터 통신을 할 수 있는 IPC 방법이다

⇒ 커널과 사용자 영역에서 각각 내부적으로 사용하는 API를 이용하여 통신한다

```c
// 소켓의 원형
int socket(int domain, int type, int protocol)
```

### 차이점

- 기존 : /dev/my_dev에 `write(fd, ubf, len)` 방식으로 간단하게 진행
- Netlink : 규격화된 헤더를 사용