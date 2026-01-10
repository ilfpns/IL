[[C]]
```cpp
template <typename T>
T add(T a, T b) {
	cout << a + b << endl;
}

int main () {
	add(3, 4);
	add(3.3, 4.4);
	add('3', '4');
	
	add<int>(3, 4);
}
```

이는 함수템플릿 이라고 한다

|typename|이 자리에 타입이 들어옴을 컴파일러에게 알리는 예약어|
|---|---|
|T|타입의 이름|

typename 뒤에 내가 자료형으로 쓸 이름을 쓴다

함수를 호출하면서 매개변수의 값을 입력하면 알아서 타입을 정한뒤 실행해준다

마지막에 add<int>는 int 형으로 해석 코드를 호출한다는 의미