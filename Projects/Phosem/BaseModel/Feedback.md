## 피드백

실상 이후 갑작스러운 피드백을 받았다. 피드백 내용 요약은 다음과 같다

```javascript
- 신경망이 너무 크다
	- WideResNet50 -> 3 layers
	- backbone
		- 3 layers가 WideResNet을 두 번 더 사용한 것인가?
		- fully connected layer를 두 개 붙여서 한 것인가?
		- 별도의 conv를 뒤에 붙인 것인가?


- NN의 전통적인 역할
	- 서로 다른 layer를 중첩시켜 신경망을 구성
	- 시퀀셜 밑에 layer가 쌓이는 구조
	- Conv -> Batch Normalization -> ReLU -> Max Pooling이 기본적인 컨볼루션 블록
	- 채널 개수만큼 반복 -> Layer 뒤에 Average Pooling Flatten 리니어 FC

- 데이터 개수 불균형 리밸런싱
	- 1000개의 양품, 100개의 불량품
	- 첫 epoch -> 100개의 불량품, 양품 1 ~ 101까지 가지고 학습
	- 다음 epoch -> 같은 불량품, 양품 101 ~ 201까지 가지고 학습
	- 반복
	- 이후 데이터 증강을 적용해야 과적합 방지

- 좋은 데이터
	- ML이 분류하지 못하면 안 됨, 큰 변화는 비전으로 처리 가능
	- 전지적 다이오드는 정확한 양/불 검사가 가능함
		- 포셈이 말한 뉘앙스는 그냥 감으로 한 듯함
	- 환경에 대한 고정이 필요하다 -> 계속 바뀌는 환경과 이로 인한 인풋은 좋지 않다

- 실제 환경
	- 데이터에 대한 판단을 근거를 가지고 할 수 있으면 좋다 -> 실제로 얻는 방법이 좋다
		- 카이스트 나노종합기술원
	- 불량에 대한 근거가 필요하다

- 향후 개발 방향
	- 망 사이즈 & 기본적인 아키텍처 잡기
		- 세부적
			- 채널 크기
			- kernel filter
			- 망 깊이
	- 망 크기를 기계적으로 스캔하여 over-fitting을 하여 원하는 accuracy를 맞추는 것이 좋다
```

<details>
<summary>회의 결과</summary>

```javascript
New Dev Pipeline
1. 같은 모델, 다른 학습
	- 동호 => 양 / 불 판정 (문제 >>> 어떻게 접합부 검증?)
		- Train : 양/불이 라벨링된 사진
		- Input : 새로운 사진
		- Output :
			1. 양 : 양품 데이터로 판단
			2. 불 : 세현 파트 Input으로 전달

	- 세현 => 불 판정의 근거 파악
		- Train : 양 사진과 불 사진에 대한 불 근거가 라벨링된 사진
			- 스크래치
			- 이물질
		- Input : 불량 데이터
		- Output : 불량 판정에 대한 근거


Arch

New Data -> 3*3 데이터를 자르기 -> PatchCore 양/불 판정
						-> 양 : pass
						-> 불 : 판단 근거 추론 -> 접합부 문제 -> pass

```

</details>
