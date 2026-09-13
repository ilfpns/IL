### File System

⇒ 파일이나 자료를 쉽게 발견 및 접근할 수 있도록 보관 또는 조직하는 체제

- 저장장치 내에 저장된 여러 개의 파일들을 고유하게 분류할 수 있는 체계가 필요하다
    
    ⇒ 특정한 약속으로 분류됨 (File System)
    

### VFS

⇒ VFS는 공통 APi를 받아서 실제 파일시스템 구현으로 디스패치하는 중간 계층이다

![[Pasted image 20260210012138.png]]

다음 사진과 같이, 다른 파일 시스템을 가져도

사용자는 open() 함수를 호출했을 때 VFS가 알아서 이를 연결해준다

⇒ 어떤 사용자 함수를 호출하더라도, VFS가 자체적으로 파일시스템에 알맞는 함수를 호출한다

- 사용자는 연결된 파일 시스템을 고려할 필요가 없음

⇒ 파일 추상화 계층

⇒ 이런 VFS이루는 4가지 객체가 있다

1. SuperBlock
2. Inode
3. Dentry
4. File

### SuperBlock

⇒ 본질적인 파일시스템 메타데이터이다

⇒ 각 파일시스템 인스턴스마다 하나의 SuperBlock이 존재한다

⇒ 매우 중요하기 때문에 복사본을 여러 곳에 저장해둔다

구성 요소

- 파일 시스템의 유형
- 파일 시스템의 상태
- Inode 정보

등이 들어있다

### Inode

⇒ 파일 각각의 메타데이터를 저장한다

구성 요소

- 파일의 주인(User, Group)
- 접근 권한
- 파일 타입
- 파일 데이터의 위치(블록 포인터)

등이 들어있다

### Dentry

⇒ 아이노드의 번호와 파일 이름을 관련하여 아이노드를 연결시켜준다

⇒ 캐시를 유지하여 자주 접근되는 경로를 더 빠르게 접근하도록 도움

⇒ 사용자 패턴에 따라 메모리에서 동적으로 생기고, 없어진다

**왜 사용하는가?**

- inode는 이름이 없다
    
- 사용자는 /my_file/a.txt 처럼 이름으로 접근
    
- 그래서 이름 ↔ inode 매핑 작업이 필요하다
    
    ⇒ dentry 객체는 해당 이름이 가르키는 Inode포인터를 포함한다, VFS는 이를 캐시로 가짐
    

**설명**

`/home/yun/Document/hello.txt` 다음과 같은 주소가 있을 때

- dentry가 없는 경우
    1. `home` 비교
    2. `yun` 비교
    3. ….
    4. 항상 문자열을 비교하는 것은 비용이 큼
- dentry가 있는 경우
    1. `dentry(home)` 비교
    2. `dentry(yun)` 비교
    3. …..
    4. 주소를 비교하기 때문에 문자열보다 비용이 적음

⇒ 경로를 개체화해서 (주소로 저장) inode와 연결함

```c
dentry("/")
 └─ dentry("home")    
     └─ dentry("yun")
         └─ dentry("Document")
             └─ dentry("hello.txt")
```

주소는 다음과 같은 구조를 가진다

- / dentry는 해시 테이블에 home dentry의 주소를 가진다
- hone dentry는 해시 테이블에 yun dentry의 주소를 가진다
- yun dentry는 해시 테이블에 Documentdentry의 주소를 가진다

**접근**

1. 시작 dentry cache는 비어있음 (/ dentry 주소는 가지고 있음)
2. home 검색 (/ 의 해시 테이블에서)
3. 키를 home으로 해시를 진행하여 이에 맞는 home dentry로 이동
4. ….
5. hello.txt 검색 (Document의 해시 테이블에서)
6. 키를 hello.txt로 해시를 진행하여 이에 맞는 hello.txt로 이동

### File

⇒ File 객체는 프로세스가 열린 파일의 상태를 표현하며, 파일 오프셋과 접근 모드를 포함한다

구성 요소

- 열린 파일 상태(open instance)
- 파일 오프셋(offset)
- 접근 모드
- file_operations
- 여러 프로세스가 같은 inode라도 File 객체는 다를 수 있음

### 결론

VFS는 공통 파일 API를 dentry 기반 경로 캐시로 해석하고, inode 및 file 객체를 통해 실제 파일시스템 연산으로 연결한다.