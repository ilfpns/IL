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
Layer에서 연산을 거친 이미가 Feature로 나오게 된다.

우리는 CNN에서 이미지 객체를 한번에 "강아지!" 라고 분류하지 않는다.<br> 이미지가 Feature Map을 거치며, 점점 의미있는 숫자 표현으로 바뀌며 분류가 진행된다.<br><br>

각 CNN Layer를 거치며 이미지를 저수준에서 고수준 분류까지 해낸다.  <br>첫 Layer는 선, 모서리를 분류한다면, 둘째 모서리는 질감, 패턴 등을 분류한다. <br><br> 이렇게 점점 고차원 분류를 진행한다.<br><br>

이러한 Layer의 묶음을 Block이라고 칭한다. Block안에는 비슷한 구조의 Layer를 계속 사용하는데, 이 Layer 여러개를 묶은 것이다.<br><br>

결론적으로 백본은 여러 단계로 나뉜 Layer/Block이다. <br> 
저차원에서 고차원 분류를 위한 사이사이의 여러 Layer/Block을 의미한다.
<br> <br>  => Model의 여러 Layer/Block의 모음
<br>=> 이미지에서 Feature를 추출하는 부분 (Layer를 거쳐 Feature가 나오므로)
</details>
<details> <summary>Feature Fusion Neck</summary>  <br>
 Backbone이 여러 단계에서 뽑아낸 Feature를 서로 합쳐서 Detection Head가 쓰기 좋은 형태로 만들어준다.<br><br>

Backbone을 가지고 이미지에서 Feature를 추출하면, layer가 깊어지며 Feature Map이 생길 것이다.  <br> Feature Map : 각 Feature가 어디에 얼마나 위치하는지 나타냄 <br><br>

문제는 Feature Map은 각기 다른 정보를 가진다. 그 까닭은 각 이미지의 Feature가 어디에 얼마나 존재하는지 Layer에 따라서 다르게 나올 것이기 때문이다.
<br><br>

Neck은 이것들을 Fusion시킨다. 대표적으로 다음 기법이 사용된다. <br>
- FPN : 깊은 층의 의미 있는 Feature를 얕을 층으로 전달해 다양한 크기의 객체를 탐지하는 구조 <br>
- PAN : 얕은 층의 위치 정보를 깊은 층으로 다시 전달해 Feature 정보를 더 잘 융합하는 구조 
<br>
<br>
왜 Feature Map을 합쳐야할까?
<br><br>
- 깊은 Layer의 Feature : 자동차는 구분 가능, 하지만 위치는 모름<br>
- 얕은 Layer의 Feature : 자동차는 구분 불가, 하지만 물체 위치는 인식
<br> 이 두 정보를 합쳐 자동차의 위치를 알아내는 것이다.

</details>
<details> <summary>Detection Head</summary>  <br>
이는 Neck에게 융합된 Feature를 받아서 최종적으로 어떤 객체가 어디에 있는지 예측한다.<br>
</details>

- BackBone : 특징 추출
- Neck : 특징 융합
- Head : 특징 예측
이라고 볼 수 있다. 이러한 구조로 Yolo는 CNN이 불가능한 BBox를 통해 객체 데이터를 정형화할 수 있는 것이다.
