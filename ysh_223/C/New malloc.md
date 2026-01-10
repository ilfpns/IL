[[C]]
```cpp
int* p = new int;    // int 하나 동적 할당
*p = 10;
```

```cpp
int* arr = new int[5];   // int 5개짜리 배열 동적 할당

for (int i = 0; i < 5; ++i)
    arr[i] = i * 10;

```

자료형 이름 = new 자료형 ← 으로 시작한다

free 대신 delete 를 사용해서 메모리를 해제한다

```cpp
class Student {
public:
    Student() { cout << "생성자 호출!" << endl; }
};

Student* s = new Student[3];  // Student 3개 동적 생성
delete[] s;   
```

그럼 왜 new 를 쓰는가

- 형변환을 해줄 필요가 없다
- 생성자 호출을 해준다 → 정상 상태 보장