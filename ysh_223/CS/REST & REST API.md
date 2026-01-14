> **REST {Representational State Transfer}**
[[CS]] 
---

⇒ 자원을 이름으로 구분하여 해당 자원의 상태를 주고 받는 모든 것

- HTTP URI을 통해 자원을 명시
- HTTP Method(POST, GET, PUT) 를 통해
- 해당 자원(URI)에 대한 CRUD Operation을 적용하는 것

### REST 구성 요소

1. 자원 : HTTP URI
2. 자원에 대한 행위 : MTTP Method
3. 자원에 대한 행위으 내용 : HTTP Message Payload

### 특징

1. 서버-클라이언트 구조
2. 무상태
3. 캐시 처리 가능
4. 계층화
5. 인터페이스 일관성

> **REST API**

---

⇒ REST의 원리를 따르는 API

### REST API의 명명 규칙

**1**. URI는 동사보다는 명사를, 대문자보다는 소문자를 사용하여야 한다.

> Bad Example [http://khj93.com/Running/Good](http://khj93.com/Running/Good) Example  [http://khj93.com/run/](http://khj93.com/run/)

2. 마지막에 슬래시 (/)를 포함하지 않는다.

> Bad Example [http://khj93.com/test/](http://khj93.com/test/)  Good Example  [http://khj93.com/test](http://khj93.com/test)

3. 언더바 대신 하이폰을 사용한다.

> Bad Example [http://khj93.com/test_blogGood](http://khj93.com/test_blogGood) Example  [http://khj93.com/test-blog](http://khj93.com/test-blog)

4. 파일확장자는 URI에 포함하지 않는다.

> Bad Example [http://khj93.com/photo.jpg](http://khj93.com/photo.jpg)  Good Example  [http://khj93.com/photo](http://khj93.com/photo)

5. 행위를 포함하지 않는다.

> Bad Example [http://khj93.com/delete-post/1](http://khj93.com/delete-post/1)  Good Example  [http://khj93.com/post/1](http://khj93.com/post/1)

> **RESTful**

---

⇒ REST API 설계 규칙을 올바르게 지킨 시스템을 명명