## if
[[C]]
C언어에는 2가지의 조건문이 있다

- if
- switch

이 두 조건문은 사용되는 상황이 다르다

1. if
    - 3개 이하의 조건
    - 대소 구분
2. switch
    - 4개 이상의 조건
    - 연속된 값 구분

### (void)

```c
#include <stdio.h>
#define UNUSED(X) (void)X

void HAL_GPIO_EXTI_Callback(uint16_t pin_number)  {
    UNUSED(pin_number);
    printf("Button pressed! LED Toggled.\\n");
}

int main(void) 
{
    HAL_GPIO_EXTI_Callback(13); 
    return 0;
}
```

다음 코드에서 `HAL_GPIO_EXTI_Callback` 함수의 파라미터는 값이 주어지지만 함수에서 사용하지 않는다

이때 컴파일러는 사용되지 않은 파라미터에 대해 경고를 띄운다

⇒ 사용자 정의 매크로를 사용하므로써 아무 의미가 없는 캐스팅을 통해 경고 무시

⇒ 정해진 규격이 있는 라이브러리, 프레임워크에서 파라미터를 받고 쓰지 않는 일이 있기 때문에 사용