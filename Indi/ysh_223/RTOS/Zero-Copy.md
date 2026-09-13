> **Zero Copy**
[[Tag/RTOS]]
---

⇒ 데이터를 복사하지 않고 메모리 주소만 전달하는 구조

### 일반 구조

```cpp
[DMA buf] --memcpy--> [Queue buf] --memcpy--> [App buf]
```

### Zero-Copy 구조

```cpp
[DMA buf] --pointer--> [Task]
```

- 메시지 : 주소에 존재
- 데이터 : 메모리에 그대로 존재

### RTOS에서

- ISR에서 `memcpy` 하면 지연 발생
- 캐시 오염
- 실시간성 깨짐

이런 문제를 해결하기 위해

- ISR : 데이터를 채우고 포인터만 넘김
- Task : 처리 후 버퍼 반환

**Zero-Copy 구조**

```cpp
[고정 버퍼 풀]
   ↓ (DMA/ISR가 채움)
[사용중 버퍼]
   ↓ (포인터 전달)
[처리 Task]
   ↓ (반환)
[자유 버퍼 풀]
```