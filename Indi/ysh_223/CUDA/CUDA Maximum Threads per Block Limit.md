### 문제

---

⇒ 한 Block에 1024개가 넘는 스레드들의 연산을 요구하니 의도하지 않은 값이 나오게 됨.

```c
__global__ my_kernel(void) {
	printf("called\\n");
}

int main(void) {
	my_kernel<<<1, 2048>>> ();
	cudaDeviceSynchronize(); 
	return 0;
}
```

### Maximum Threads per Block Iimit

---

<img width="1201" height="361" alt="image" src="https://github.com/user-attachments/assets/e1662fe8-df24-40d0-907c-878423cf88c2" />

CUDA table

- Dimensionality of thraed block (최대 차원) : 3
- X or Y Dimension (X 또는 Y의 최대 스레드) : 1024
- Z Dimension (Z 최대 스레드) : 64
- Threads per block (블럭의 스레드) : 1024
- Warp size (Wrap 크기) : 32

⇒ 한 블록에 최대 스레드는 1024개이다.

### Why?

---

- 레지스터 공유
    
    ⇒ 한 블록 내의 모든 스레드는 SM이라는 하드웨어 유닛의 자원을 공유
    
    ⇒ 스레드가 너무 많으면 각 스레드가 쓸 수 있는 메모리가 부족
    
- 동기화 (Sync)
    
    ⇒ `__syncthreads()` 를 통해 블록 내 스레드끼리 동기화 할 수 있음
    
    ⇒ 너무 많은 스레드는 관리 비용이 크게 발생하는 위험이 있음
    

### 해결

---

⇒ 한 block당 Thread를 줄이고 Grid의 Block 수를 늘림

```c
__global__ void my_kernel(void) {
    printf("called\\n");
}

int main(void) {
    my_kernel<<<8, 256>>>();
    cudaDeviceSynchronize(); 
    return 0;
}
```
