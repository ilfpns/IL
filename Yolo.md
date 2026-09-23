Jetson 보드에서 이미지 탐색을 위해서 Yolo모델을 사용하기로 헀다. 
이번 장에서는 크게 두 가지를 볼 것이다. 

1. why Yolo?
2. what is Yolo?

---
### Why Yolo
Yolo는 CNN을 기반으로 하는 AI이다. CNN은 이미지에 대한 classification을 진행한다. 

CNN은 Input을 가중치 계산을 하고 Output을 내놓은 신경망 구조이다. 특징 추출 등에 사용되며, 이미지를 구분하는 역할을 한다.

반면 Yolo는 CNN Backbone을 기반으로 하는 객체 탐지 모델이다. 이미지 분류보다는 객체 탐지에 더 적합한 모델이라는 것이다. 그러므로 어떤 객체가 어디에 있는지 판단하기에 용이하다. 

또한 CNN과 달리 Yolo는 BBox 기능을 지원한다. 

<details> <summary>BBox란?</summary> 
<br>
Bounding Box : 객체를 숫자로 바꿈 <br>
이는 이미지 속 객체를 정형 데이터로 가공하기 위해서 사용한다.
<br><br>
객체를 감싸는 직사각형 등의 도형으로 객체의 좌측 상단 좌표, 도형의 height, weight를 정형화한다.<br>
<br>
왜 객체를 정형 데이터로 사용해야할까?<br>
=> 정형 데이터는 가공하여 다른 여러 정보를 얻을 수 있음<br>
<br>
1. 객체 간 사이 거리<br>
2. 시간에 따른 이동<br>
3. 객체 간 포개짐<br>

</details>

우리는 프로젝트에서 '객체 간 사이 거리' 정보와 '객체 간 포개짐' 정도를 필수적으로 알아야 한다.

또한 Yolo는 ONNX화의 호환성이 좋다. 우리는 Pytorch framework에서 개발을 할 예정이다. 여기서 학습을 진행하고, Jetson을 위한 TensorRT 가속기를 위한 변환을 하고, CUDA에 맞춰 산출물을 만들어야 한다.

구조는 다음과 같은 것이다.
```
Pytorch (학습) -> ONNX -> TensorRT -> CUDA 
```
IR인 ONNX가 OP (연산식) 규격을 맞추고, opset 버전 관리를 해줄 에정이다.
<details> <summary>Pytorch 사용 이유</summary> 
<br>
우리는 왜 Pytorch를 사용할까?
<br><br>
개발자가 미리 만들어진 기능과 구조를 이용해 개발물을 만들어내는 것을 돕는 툴을 Framework라고 한다.<br><br>

LIbrary와의 차이는 개발에서의 제어권이 Framework에게 있어서 흐름을 Framework가 가져간다는 점이다.<br> Library는 개발자가 호출하고, Framework는 개발자를 호출하는 구조이다. <br><br>

이런 상황을 IoC (제어의 역전) 이라고 한다.<br>
</details>

Yolo에서는 ONNX 배포가 간단해서 쉬운 개발 난이도를 가진다.

우리는 이러한 이유들로 Yolo 모델 중 최신 버전인 Yolov8을 사용한다.

---
### What is Yolo

Yolo는 CNN 하나만을 기반으로 하지 않는다.
- Yolo model = CNN Backbone + Feature Fusion Neck + Detection Head

이렇게 세 조합이 Yolo를 구성한다. 한 파트를 하나씩 알아보자  

<details> <summary>CNN BackBone</summary> 
<br>
백본은 딥러닝 모델에서 Input data feature를 뽑아내는 네트워크이다.
<br> 조금 더 설명을 해보겠다. <br><br>

우리는 CNN에서 이미지 객체를 한번에 "강아지!" 라고 분류하지 않는다.<br> 이미지가 Feature Map을 거치며, 점점 의미있는 숫자 표현으로 바뀌며 분류가 진행된다.<br><br>

각 CNN Layer를 거치며 이미지를 저수준에서 고수준 분류까지 해낸다.  <br>첫 Layer는 선, 모서리를 분류한다면, 둘째 모서리는 질감, 패턴 등을 분류한다. <br><br> 이렇게 점점 고차원 분류를 진행한다.<br><br>

결론적으로 백본은 여러 단계로 나뉜 Layer이다. <br> 
저차원에서 고차원 분류를 위한 사이사이의 여러 Layer를 의미한다.
<br> <br>  => Model의 여러 Layer의 모음

<br>
</details>


