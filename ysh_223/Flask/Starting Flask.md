[[Flask start]]
```python
from flask import Flask

app = Flask(__name__)

@app.route('/')
def home():
    return "Hello, Flask!"

if __name__ == '__main__':
    app.run(debug=True)
```

1. `from flask import Flask`
    - Flask 라이브러리에서 Flask 클래스를 가져온다
    - 이 클래스가 웹 애플리케이션의 핵심이다
2. `app = Flask(__name__)`
    - FLask 애플리케이션 인스턴스를 생성한다
    - `__namme__` 은 현재 모듈의 이름을 나타내는 Python 내장함수이다
    - Flask가 템플릿, 정적 파일 등의 위치를 찾는데 사용됨
3. `@app.route('/')`
    - 데코레이터라고 불린다
    - 특정 URL 경로(’/’)와 함수를 연결한다
    - 사용자가 웹사이트의 루트 경로에 접속하면 아래 함수가 실행된다
4. `def home():`
    - 브라우저에 표시될 내용을 반환한다
    - HTML, 문자열, JSON, 템플릿 등을 반환할 수 있다
5. `if __name__ == '__main__':`
    - 이 파일이 직접 실행될 때만 아래 코드가 실행된다
    - 다른 파일에서 import할 때는 실행되지 않는다
6. `app.run(debug == True):`
    - Flask 개발 서버를 시작한다
    - `debug==True` 코드 변경 시 자동 재시작, 에러 정보 표시