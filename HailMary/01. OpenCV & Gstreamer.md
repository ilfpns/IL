### OpenCV
컴퓨터 비전 라이브러리이다. 
카메라에게 영상을 받고, 설정하고, 세팅하는 라이브러리이다.
=> 컴퓨터 비전은 이미지나 영상을 보고, 그 안의 의미를 이해하도록 만드는 기술

- 이미지 크기 변경
- 색상 변환
- 블러 
- 이미지 합성
- 객체 검출
- 얼굴 인식
- AI 모델 추론 결과 처리
와 같이 많은 기능은 지원하는 라이브러이다.
### Gstreamer 
멀티미디어 파이프라인 프레임워크이다.

파이프라인은 간단하게 데이터 처리 흐름이라고 생각할 수 있다.
Camera -> Capture -> Decode -> Resize -> Converrt -> Display

OpenCV로 영상을 가지고 무언가를 하면 Gstreamer는 영상을 어떻게 처리하고 이동하는지를 정한다.


```
gst-launch-1.0 videotestsrc ! videoconvert ! autovideosink
```
위와 같은 Pipeline이 있다고 치면 각 다음 기능이 '!' 로 이어지며, 

```
videotestsrc
      ↓
videoconvert
      ↓
autovideosink
```
이런 Pipeline을 구성한다

- videotestsrc : 영상 데이터를 만들어냄
- videoconvert : 영상 포맷을 변환
- autovideosink : 화면에 출력
각 명령을 Element라고 한다.

```
                 GStreamer
┌─────────────────────────────────────┐
│                                     │
│ Camera → Convert → Resize → appsink │
│                              │      │
└──────────────────────────────┼──────┘
                               ↓
                            OpenCV
                               ↓
                          Preprocessing
                               ↓
                           YOLO/AI
                               ↓
                          Postprocessing
                               ↓
                 ┌─────────────┴─────────────┐
                 ↓                           ↓
              Display                    GStreamer
                                             ↓
                                           Encode
                                             ↓
                                           RTP
                                             ↓
                                         Network
```
전체 구조
