```c
sudo curl -L -O https://git.openembedded.org/bitbake/snapshot/bitbake-1.46.0.tar.gz
sudo tar -xzf bitbake-1.46.0.tar.gz
```

일단 bitbake를 받아와 준다

### bitbake

SW를 build/compile 하는 toolchain이다.

```c
bitbake/
	/bin
		/ 여러 중요 파일
```

### Metadata

본래는 데이터를 설명하는 (크기, 시간, 위치 등) 데이터로 쓰이지만 yocto는 이를 build방식과 의존성을 기술한 파일을 의미한다.

파일안에는 2가지의 데이터를 담는다

- 변수
- 실행 가능한 함수

파일 종류는 크게 5개로 나뉜다

- 환경 설정
    - 변수만 기재 가능
    - 이 파일에 선언되는 것들은 전역 변수로 취급한다.
    - 설정 파일은 `.conf` 확장자를 가진다.
- 레시피
- 레시피 확장
- 클래스
    - 빌드를 위해
- 인클루드

⇒ 5개의 파일을 bitbake 거쳐서 buildroot, rootfs, kernel image등을 만든다.
