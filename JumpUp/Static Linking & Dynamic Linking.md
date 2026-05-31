### Linking이 뭘까?

컴파일 과정은

1. 전처리
2. 어셈블리
3. 링킹 

과정으로 진행된다. 

전처리에서 코드를 다듬은 i파일을 어셈블리에서 s파일로 만든다. 이렇게 목적 파일 (object file)이 만들어진다.

OBJ file + Lib file = Executable FIle

링커는 다음과 같은 일을 한다

- 여러 소스코드 파일을 하나로 합친다. obj파일을 하나로 합친다
- 합쳐진 obj에 lib를 합친다

printf 코드라면 컴파일 (obj file 생성) + iostream lib 추가

### static linking

- 실행파일을 만들 때 lib을 포함시켜, exe파일을 만드는 정적 링킹

장점 

- 컴파일 하면서 라이브러리가 복사되어 바로 실행파일이 된다
- exe에 있기 떄문에, 별도 lib이 필요없다.
- 미리 컴파일 했기 떄문에 컴파일 시간 단축

단점

- exe안에 라이브러리가 있기 떄문에 메모리를 많이 먹는다

### Dynamic linking

프로그램에서 사용하는 lib 모듈의 주소만 가지고 있다가, runtime에 exe에서 lib을 필요로 할 때 해당 주소로 가서 lib을 쓰고 돌아오는 방식이다.

이를 DLL라고 한다, 동적 링킹 시 사용되는 lib이다

- stub
    
    lib이 mem에 존재하지 않을 때, lib가 mem에 상주할 수 있도록 lib 루틴을 적재하는 방법을 알려주는 코드
    
    이는 모든 lib 루틴에 들어있다
    
    - runtime 시 해당 routine이 불리면, 해당 stub은 자신을 routine이 들어가는 주소와 바꿔치기한다
    - stub이후 stub은 사라지며, 남은 코드는 연결된다

장점

- 메모리 요구 사항이 적다

단점 

- lib가 위치한 주소까지 jump 하므로 overhead가 발생한다
- 불필요한 코드가 추가된다 ( for jump)
