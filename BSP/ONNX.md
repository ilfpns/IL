- 서론
    
    대회 관련 BSP 공부를 하다가 ONNX라는 개념을 배웠다. 일단 알아두고 넘어갔는데, 이 역시 경식 선배님이 언급하셨던 기술이기에 공부해본다.
    
    뭔가 대충 보니까 
    
    - ONNX
    - ONNX runtime
    
    에 대해 공부할 거 같다
    

#### 배경

임베디드 환경에서 AI 실행 시 문제가 있음

⇒ 각 학습 프레임워크와 배포 환경이 다르다. 특히 임베디드는 보드와 로컬 PC의 환경이 다르다. 

```c
연구자/개발자          배포 환경
-----------          -----------
PyTorch로 학습    →   Raspberry Pi (ARM CPU)
TensorFlow로 학습 →   NVIDIA Jetson (GPU)
Keras로 학습      →   Intel NUC (Intel CPU)
```

#### ONNX?

Open Neural Network Exchange는 딥러링 모델을 표현하기 위한 표준 포맷이다.
딥러닝 프레임워크 간 호환성을 높혀주는 것이 목적이다.

PDF로 이해하면 편하다, hwp이던 md이던 pdf로 편하게 뽑아 볼 수 있다. 이것처럼 어떤 프레임워크에서 제작된 AI던 ONNX로 쉽게 변현시킬 수 있다.

AI의 계산 그래프의 포터블한 직렬화 포맷을 규정한다.

#### ONNX runtime

포맷의 파일을 실제로 실행하는 엔진이다. ⇒  ONNX 모델을 실행하는 추론 엔진이다.

Execution Provider(EP)는 ORT의 하드웨어 백엔드로, CPU, NVIDIA CUDA GPU, TensorRT, DirectML 등 다양한 하드웨어에서 실행할 수 있게 해주는 소프트웨어 모듈이다.

```c
ONNX Runtime
├── CPU EP        ← 기본값, 어디서나 동작
├── CUDA EP       ← NVIDIA GPU
├── TensorRT EP   ← NVIDIA GPU (더 최적화)
├── OpenVINO EP   ← Intel CPU/GPU/VPU
├── ArmNN EP      ← ARM 기반 임베디드 장치
├── NNAPI EP      ← Android 장치
└── QNN EP        ← Qualcomm 칩
```

ARM 장치에서 ONNX 모델 성능을 가속화하려면 Arm NN Execution Provider를 사용해야 한다.

ONNX runtime은 실행 전 모델의 계산 그래프를 분석하고 최적화를 진행한다

1. Constant Folding
2. Operator Fusion
3. Graph 프루닝

#### BSP ONNX

- 아
    
    물론 내 수준이 낮아서 해보지는 못한다
    

- 추론을 의한 NPU 드라이버 개발
- OS 이미지 최적화
- C/C++ 기반의 ONNX runtime 라이브러리 시스템을 이미지에 포함

⇒ BSP가 onnx runtime을 안정적으로 지원한다면, 앱/웹은 프레임워크에 구애받지 않고, .onnx파일만으로 보드에서 바로 실행 가능
