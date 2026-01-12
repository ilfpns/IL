CPU 부팅 중 setup_arch() 에서
[[Kernel]]
물리 메모리를 수집 → 가상주소로 매핑 → 페이지 단위로 커널이 쓰게 만드는 과정 설명

1. 물리 메모리 수집
2. 가상주소 매핑
3. 페이지 단위로 제작

### Flow

```cpp
1. 물리 메모리 수집 (memblock)
2. 물리 → 가상 주소 매핑 (page table 준비)
3. 메모리 모델 초기화 (sparse)
4. Zone / free page 초기화
5. 커널이 페이지 단위로 사용 가능 상태
```

1. Setup_arch()
    
    - x86_64: `arch/x86/kernel/setup.c`
    - arm64: `arch/arm64/kernel/setup.c`
    - riscv: `arch/riscv/kernel/setup.c`
    
    ⇒ 해당 CPU의 메모리가 어떻게 생겼는지 수집
    
2. memblock에 물리 메모리 수집
    

- BIOS / DT / FW 정보 기반
- 사용 가능 메모리
- 예약 메모리(reserved) 구분

```
memblock.memory   → 사용 가능
memblock.reserved → 커널, device, hole
```

1. 물리 → 가상 주소 매핑 (MMU)
    
    아키텍처별로 다름:
    
    - x86_64
        
        `init_mem_mapping()`
        
        `__kernel_physical_mapping_init()`
        
    - arm64
        
        `map_mem()`
        
        `__map_memblock()`
        
    - riscv
        
        `create_pgd_mapping()`
        
    
    공통 개념
    
    1. PGD
    2. P4D
    3. PUD
    4. PMD
    5. PTRE
    6. PAGE
2. spares_init()
    
    - 물리 메모리를 PFN 단위로 분해
    - mem_section[] 구성
    
    ⇒ 페이지들이 어디에 존재하는지 구조화
    
3. 실제로 쓸 수 있는 free page 계산
    
4. memmap_init()
    
    - `struct page` 초기화
    - `SetPage()` 수행
    - buddy allocator에 연결
5. kassan_init()
    
    - 메모리 오염 검사
    - 디버깅용

|CPU|차이점|
|---|---|
|x86_64|e820 기반, paging_init 이후 공통|
|arm64|bootmem_init 안에서 공통 초기화|
|riscv|misc_mem_init 안에서 공통 초기화|