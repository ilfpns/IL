[[ESP32]]

# #include <…>

**⇒ 컴파일러가 시스템 include path에서 바로 찾아옴**

**⇒ 흔히 내부 라이브러리라 불림**
[[System include]]


# #include “…”

**⇒ 현재 소스파일 위치를 먼저 찾고, 없으면 include path를 찾음**

**⇒ 흔히 외부 라이브러리라 불림**
[[double-quote include]]
