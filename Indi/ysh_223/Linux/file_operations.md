![[Pasted image 20260310225304.png]]
/linux/include/linux/fs.h

> **File Operations**
[[Linux]]
---

⇒ 운영체제가 제공하는 6가지 파일 시스템 콜은, 실제로는 file_operations를 통해 디바이스 드라이버 코드로 연결된다

시스템콜 ↔ VFS ↔ File operations

**결론**

⇒ 이 파일에 대한 특정 호출이 들어왔을 때 어떤 함수가 실행할지를 정의한 함수 테이블