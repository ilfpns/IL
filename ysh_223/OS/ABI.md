### Application Binary Interface

---

⇒ 두개의 바이너리 프로그램 모듈 사이의 인터페이스

⇒ 0과 1만이 있는 low-level 수준에서의 인터페이스

- interface?
    
    ⇒ 데이터를 주고받기 위해 서로가 미리 맞춰놓은 규격
    
- Window App과 macOS App이 호환이 되지 않는 이유는 ABI가 호환되지 않기 때문이다
    

### 0과 1의 규격을 어떻게 맞출까?

- Flow
    
    기존 : 첫 번째 숫자는 CPU의 EAX 레지스터에 담아 보내는 인터페이스 (규칙)을 가짐
    
    새로운 설계 : 여전히 첫 번째 숫자는 EAX 레지스터에 담아 간다고 인식
    
- Source Compatibility 소스 호환성
    
    ⇒ 새로운 컴파일러가 이전 버전으로 작성된 코드를 컴파일 할 수 있는 것
    
    ⇒ 여러 버전에서 single code기반을 유지하며, 사용자가 새로운 버전의 코드를 이용할 수 있게 한다
    
    - 소스 호환성이 없다면?
        
        ⇒ 모든 소스코드와 패키지가 동일한 버전으로 작성되어야 함, 이는 version lock을 유발한다
        
- Binary Compatibility / ABI 호환성
    
    상황 : 라이브러리 파일만 새 버전으로 교체
    
    결과 : 다시 컴파일하지 않아도 새 라이브러리와 오류 없이 맞물려 돌아감
    

### ABI가 다루는 것들

- CPU instructions
- 함수 호출
- OS 호출
- Data layout
- type metadata
- mangling
- runtime
- stdlib

⇒ 만약 ABI가 수정된다면?

⇒ 수정된 규격에 맞도록 다시 컴파일 하면 된다

### ABI Stability / ABI 안전성

⇒ 라이브러리가 바뀌어도, 새 컴파일 없이 프로그램이 잘 돌아가는 상황

⇒ 기존 인터페이스를 지킴