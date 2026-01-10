[[Flask]]
```python
topics = [
    {'id' : 1, 'title' : 'HTML', 'body' : 'HTML is. . .'},
    {'id' : 2, 'title' : 'CSS', 'body' : 'CSS is. . .'},
    {'id' : 3, 'title' : 'JavaScript', 'body' : 'JavaScript is. . .'},
]

@app.route('/')
def home():
    liTags = ''
    for topic in topics:
        liTags = liTags + f'<li><a href = "/read/{topic["id"]}/">
	        {topic["title"]}</a></li>'

    return f'''<!doctype html>
    <html>
        <body>
            <h1><a href = "/">WEB</a></h1>
            <ol>
                {liTags}
            </ol>
            <h2>welcome</h2>
            Hello, web
        </body>
    </html>
    '''
```

⇒ topics는 list이다 그 안에 딕셔너리로 값을 관리할 수 있다

⇒ topics를 순회하며 `liTags` 에 값을 넣는다

⇒ 딕셔너리 부분을 데이터 베이스와 연동되는 코드로 바꾸면 실전 예제로 사용할 수 있다