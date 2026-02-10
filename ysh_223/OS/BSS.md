> **영역**
[[OS]]
---

⇒ 프로세스 구조에서 Data영역은 BSS와 Data영역으로 나뉜다

⇒ RAM에서 채워진다

- BSS : 초기화 값이 없는 전역 변수
- Data : 초기화 값이 있는 전역 변수

```c
int global_data1; //초기화 값이 없는 변수
int global_data2 = 1; // 초기화 값이 있는 변수

int main()
{
    int *data;
    data = (int *) malloc(sizeof(int));
    *data = 1;
    printf("%d\\n", *data);
    return 0;
}
```

![[Pasted image 20260210162240.png]]

---

```c
int a;              // BSS
static int b;       // BSS
int c = 0;          // BSS
static int d = 0;   // BSS
```

- 전역 변수
- static 변수
- 초기값이 명시적으로 0이거나, 아예 없는 경우

### 왜 Data와 BSS를 분리할까?

- 실행 파일 크기 감소
    - BSS영역은 파일에 실제 데이터가 없다
    - 크기 정보만 있어, 로딩 시 OS가 0으로 초기화 한다
- Flash 절약
    - Flash : 크기 정보만 있음
    - RAM : 실행중