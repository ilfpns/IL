[[C]]
# snprintf

⇒ 서식 문자열을 이용해 버퍼에 안전하게 문자열을 쓰는 함수

```python
int head_len = snprintf(
    head,             // 출력 버퍼 (head 배열)
    sizeof(head),     // 버퍼 크기 (256)
    "--%s\\r\\n"
    "Content-Disposition: form-data; name=\\"image\\"; filename=\\"esp32-cam.jpg\\"\\r\\n"
    "Content-Type: image/jpeg\\r\\n\\r\\n",
    boundary);        // %s 자리에 boundary 문자열 삽입

```

- head 버퍼 안에 다음과 같은 문자열이 만들어짐

```python
------ESP32CamBoundary1234
Content-Disposition: form-data; name="image"; filename="esp32-cam.jpg"
Content-Type: image/jpeg
```

- 위 문자열의 길이가 반환됨
    
- EX)) boundary길이가 22바이트라면 전체 약 134바이트
    
    - `"--"` : 2
        
    - `boundary` : 22
        
    - `"\\r\\n"` : 2
        
        → 여기까지 = 26
        
    - `"Content-Disposition: form-data; name=\\"image\\"; filename=\\"esp32-cam.jpg\\"\\r\\n"`
        
        - 이 부분 글자 수: 74
    - `"Content-Type: image/jpeg\\r\\n\\r\\n"`
        
        - `"Content-Type: image/jpeg"` = 23
        - `"\\r\\n\\r\\n"` = 4
        - 합 = 27
    
    26 + 74 + 27 = 127 ( +1 null)