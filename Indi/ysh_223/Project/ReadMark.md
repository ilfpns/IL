[[Project]]
# 이슈텔러

## 기능

- 마지막으로 읽은 책의 페이지 인식
- 한 번의 독서 활동에서 읽은 평균 페이지 양
- 오랫동안 책을 읽지 않는다면 사용자에게 알림
- 현재 책갈피가 꽃혀있는 페이지 즐겨찾기 가능
- 한 번의 독서 활동에서 읽은 페이지 양
- 물리 버튼을 통한 모드 변경
- 모바일에서 특정 미션을 통한 캘린더 꾸미기 가능
- 충전 가능한 배터리로 작동

---

- 마지막으로 읽은 책의 페이지 인식
    
    - 소형 카메라로 번호 촬영 → **Raspberry Pi Camera Module (3,900)**
    - 페이지 번호 인식 → **Raspberry Pi Zero 2 W (23,500)**
    - OCR기능을 사용할 수 있으며 저렴한 편
    - OCR을 통해 받아온 페이지의 숫자는 특정 변수에 저장되어 여러 값을 계산하는데 사용됨
---

- 오랫동안 책을 읽지 않는다면 사용자에게 알림
    
- 책갈피쪽에 무게를 감지하는 부품이 있음 → SEN0294 RP-C18.3-ST Thin Film Pressure Sensor (7,500)
    
- 사용자는 앱에서 마지막으로 읽은 날짜와 읽지 않은 날을 볼 수 있음
    
- 읽은 부분이 적어 무게가 측정조차 안되는 상황이 있을 수 있음 ← 이 부분에 해결 방안을 고려해야함
    
    - 해당 부품은 압력에 따라 아날로그 전압 값이 달라지는 형태
        
    - 무게 감지에 대한 예민도 설정을 해야할 듯 함
        
        → 아주 작은 변화도 반응(예민)
        
        →일정 수준 이상의 변화만 반응(덜 예민)

---

- TTP223로 모드를 바꿀 수 있음 → **TTP223 (500)**
    1. mode 1 → 기본 페이지 (한 번 누름)
        
        - 마지막으로 읽은 페이지
        - 전체 페이지중 읽은 페이지 (34/233)
        - 프로그래스 바 (가로 or 세로)
        - 시간?
        - 배터리
    2. mode 2 → 날짜(두 번 누름)
        
        - 마지막으로 읽은 날짜
        - 처음 읽기 시작한 날짜
        - 몇일 연속 읽었는지 표시
    3. mode 3 → 독서 활동 정보 (세 번 누름)
        
        - 그날 읽은 페이지 양
        - 읽는데 소요된 시간
        - 평균적으로 읽는 페이지
    4. mode 4 → on/off (길게 누름)
        
        - 종료
        - 책에 대한 정보 초기화
---

→ 이 모든 정보들을 안드로이드, 디스플레이로 송출함

얇은 디스플레이이면서 전력 소비가 적으면서 작은 모델 →  **SSD1306 0.96** (3,500)

[https://www.devicemart.co.kr/goods/view?no=12538774→대비책](https://www.devicemart.co.kr/goods/view?no=12538774%E2%86%92%EB%8C%80%EB%B9%84%EC%B1%85)

---

충전 가능 모듈

- **TP4056** (680)

---

### 고려할 점

- 디스플레이 방향을 양 방향으로 할 것인가? → 책의 한 면을 찍을 때 디스플레이가 뒤집어질 수 있음
- 포고핀 충전 단자 고려
- 압력 센서가 존재함 → 낮은 무게를 측정하지 못함 → 얇은 책은 ?

책에 끼는 부분은 얇음, 스캐너와 압력센서가 있음

## 기능 명세

- 전원

전원을 킴(on/off or 버튼을 눌러 초기화하여 다음 책으로 넘어갈 때)

처음 초기화를 진행후 맨 처음 찍는 페이지

→ 책의 마지막 페이지로 인식

이러한 방법으로 책의 마지막 페이지를 구함

길게 눌렀을 때 처리해야할 상황

1. 초기화
2. 전원 종료

```cpp
if (mode == 4) {
    lastPage = 0;
    lastReadDate = 0;
    dailyPageCount = 0;
    readingTime = 0;
    averagePages = 0;
    // 필요 시 EEPROM 초기화도 포함
}
```

→ 모든 변수를 초기화 하여 값을 초기화

```cpp
#include <cstdlib>

system("sudo shutdown now");
```

→ 디스플레이 종료(배터리 아끼기)

- 마지막으로 읽은 페이지

캠을 통해 페이지 번호를 인식받음

32P를 읽으면 32P그대로 들어옴

→ 고려해야할 점

1. 제대로 되지 않은 익식
    - B2P, 3zP등 제대로된 인식에 실패할 경우 숫자만 추출했을 때 해당 페이지가 책의 마지막 페이지 범위 내의 숫자인지 파악 그렇지 않으면 다시 페이지 인식 시도
2. 숫자만 받아야함
    - 32P를 제대로 읽었다면 숫자인 32만 받음
    - re.findall(r'\d+', '32p') 이러한 방식으로 숫자만 추출 가능

이렇게 인식된 마지막 페이와, 독서 활동날의 마지막 페이지를 구하고 디스플레이에 표시

426쪽의 책에서 32P까지 읽었을 때 (32/426) 방식으로 표현

→ 책의 진행도를 연결한 프로그래스바 디스플레이에 추가

→ “u8g2” 라이브러리를 사용하여 프로그래스바 제작

```cpp
u8g2.setFont(u8g2_font_ncenB08_tr);
        u8g2.drawStr(0, 12, "Progress:");
        
        // 진행도 표시
        char buf[30];
        sprintf(buf, "%d/%d (%d%%)", current_page, total_pages, progress);
        u8g2.drawStr(0, 24, buf);

        // 프로그래스 바
        u8g2.drawFrame(0, 40, bar_width, 10);
        u8g2.drawBox(0, 40, filled, 10);
```

- 1회 독서 활동시 평균 독서 페이지 수, 한 번의 독서 활동에 읽은 수
    
    그날 읽은 책의 장을 today_read_page (한 번의 독서 활동에 읽은 수)
    
    읽기 시작한 날을 day_do_read
    
    라고 선언할 경우
    
    (today_read_page / day_do_read) = 평균 독서 페이지 수
    
- 오랫동안 책을 읽지 않으면 알림
    
    사용자가 앱에서 알림 주기를 설정
    
    감압센서가 눌려있을 때 책을 읽지 않을걸로 간주하여 day_do_not_read가 카운팅됨
    
    → 사용자가 원한 알림 일수에 도달했을 때 알림
    
- 즐겨찾기
    
    현재 책갈피가 꽃혀있는 페이지(마지막으로 읽은 페이지)를 앱에서 즐겨찾기 할 수 있음
    
- 모바일 캘린더
    
    총 2개의 탭이 있음
    
    1. 한 달의 달력을 1탭에 띄어줌 하루에 사용자가 지정한 페이지 수 이상을 읽으면 해당 날짜에 도장을 찍어줌
        
        →사용자 입장에서 도장을 몹는 재미, 성취감
        
        →과거에 찍은 도장도 확인
        
    2. 사용자에 독서 활동 관련 정보를 확인할 수 있는 페이지
        
        → 디스플레이에서 확인하던 전체적인 정보들을 확인할 수 있음
        
    3. 사용자가 지정한 페이지를 5p읽기(Goal)라고 정했을 때
        
        Goal 만큼 읽음 → 동 도장
        
        Goal + 5p 읽음 → 은 도장
        
        Goal + 10p읽음 → 금 도장
        
        ```cpp
        import React, { useState } from 'react';
        import { View, StyleSheet } from 'react-native';
        import CalendarPicker from 'react-native-calendar-picker';
        
        const App = () => {
          const [selectedDate, setSelectedDate] = useState(null);
        
          const customDatesStyles = [
            {
              date: new Date(2025, 4, 10), // 2025-05-10
              style: { backgroundColor: '#fff' }, // 배경 유지
              textStyle: { color: 'red', fontWeight: 'bold' }, // 텍스트 스타일 변경
            },
            {
              date: new Date(2025, 4, 15), // 2025-05-15
              textStyle: { color: 'blue', fontWeight: 'bold' },
            },
          ];
        
          return (
            <View style={styles.container}>
              <CalendarPicker
                onDateChange={setSelectedDate}
                customDatesStyles={customDatesStyles}
                selectedDayColor="#d3d3d3"
              />
            </View>
          );
        };
        
        const styles = StyleSheet.create({
          container: {
            flex: 1,
            marginTop: 50,
          },
        });
        
        export default App;
        ```
        
    
    n일마다 읽을 페이지를 설정하는 기능을 추가하여 그에 맞게 책을 읽으면 달력을 채울 수 있음
    
    달력은?
    
    → 한 달에 달력을 띄어줌 그 달력에 도장을 찍어줌(읽은날)
    
    이런 방법으로 독자는 책을 읽으며 성취감도 얻을 수 있다
    
    연속해서 책을 읽을 경우 변수(Steady) 1증가
    
    → 하루에 한 번
    
    → 날짜가 바뀌어야 다음 카운팅 가능
    
- 모드 변경
    

```cpp
int mode = 0;
unsigned long lastPress = 0;
int pressCount = 0;

void loop() {
    int sensor = digitalRead(touchPin);

    if (sensor == HIGH) {
        unsigned long now = millis();

        if (now - lastPress > 1000 && sensorStillHigh()) {
            // 길게 누름: 모드 4
            mode = 3;
            pressCount = 0;
        }
        else if (now - lastPress > 250) {
            // 짧게 누름: count++
            pressCount++;
            lastPress = now;
        }

        delay(100); // 디바운싱
    }

    // 일정 시간(예: 1초) 동안 누르지 않으면 모드 확정
    if (millis() - lastPress > 1000 && pressCount > 0) {
        if (pressCount == 1) mode = 0;
        else if (pressCount == 2) mode = 1;
        else if (pressCount == 3) mode = 2;
        pressCount = 0;
    }

    displayMode(mode);
}
```

→ 짧게 누르면 pressCount증가 증가된 수에 따라 모드 변경

→ now - lastPress > 1000 를 통해 버튼이 1초 초과로 눌림을 감지함, 동시에 sensorStillHigh()값으로 버튼이 현재 눌려있는 상태가 지속되어야함

- 디스플레이 종료

버튼이 물리적으로 눌림 or 새로운 페이지가 인식됨 or 감압센서가 작동함

```cpp
#include <chrono>

using namespace std::chrono;

bool screenOn = false;
auto lastActivityTime = steady_clock::now();

void loop() {
    bool active = isButtonPressed() || isNewPageDetected() || isPressureSensorActive();

    if (active) {
        lastActivityTime = steady_clock::now();
        if (!screenOn) {
            displayOn();     // OLED ON
            screenOn = true;
        }
    }

    // 3초 동안 아무 입력 없으면 화면 OFF
    if (screenOn && duration_cast<seconds>(steady_clock::now() - lastActivityTime).count() >= 3) {
        displayOff();        // OLED OFF
        screenOn = false;
    }

    delay(50); // 또는 적당한 루프 대기
}

```

---

# Backend

|이름|자료형|정보|
|---|---|---|
|id|int|기본키|
|data|string|2025-05-04|
|end_page|int|책 읽은 후 마지막 페이지|
|sec|int|읽은 시간|
|start_page|int|책 읽기 시작한 페이지|
|total_pages|int|책 전체의 페이지|

|이름|자료형|정보|
|---|---|---|
|last_page|int|마지막으로 읽은 페이지|
|last_data|text|2025-05-02|
|mode|int|모드 1~4|

---

**Raspberry Pi Zero W + Raspberry Pi Camera Module:  \30,520**

**0.96인치 OLED 디스플레이 (SSD1306): \ 3,500**

**SEN0294 RP-C18.3-ST Thin Film Pressure Sensor: ₩7,500**

**TTP223 터치 센서 모듈: ₩5,700**

**TP4056 + MT3608 = \680**

**필요 예산: ₩47,900**