> **Before**
[[Linux]]
---

커널도 결국 메모리 위에서 돌아간다

이런 커널은 속도를 위해 page out이 되지 않도록 한다

페이지를 RAM에서 직접 in/out하면 비용이 발생하기 때문이다

하여 2가지 메모리 할당 방법을 사용한다

- Buddy System
- Slab Allocator

> **Buddy System**

---

⇒ 고정된 크기의 segment를 할당해준다

⇒ 최소 4K를 할당한다

![image.png](attachment:c71141f2-3ac5-4c97-be81-20574f272dec:image.png)

다음 사진은 21K 데이터 요청이 왔을 때 할당하는 방식이다

다음과 같이 같은 size로 최적해를 찾아 나눠지기에 Buddy라는 이름이 사용된다

- 큰 size에 요청에 대해, 메모리를 합쳐 size를 늘려 제공을 빠르게 할 수 있다

> **Slab Allocator**

---

Slab : 물리적으로 연속된 메모리로 구성돼 있는 영역

Slab은 미리 다양한 size의 Cache를 만들어 놓는다

⇒ 미리 할당해 놓은 작은 메모리 조각을 kmalloc에 요청에 따라 가장 가까운 메모리 조각을 반환한다

### Slab 구성 요소

- Slab Cache
    
    ⇒ 자주 사용하는 구조체에 대한 동적 메모리를 미리 확보하고 관리하는 주체
    
- Slab page
    
    ⇒ buddy system에서 관리되는 ordem-N 단위의 페이지
    
- Slab Object
    
    ⇒ Slab Cache가 할당해 놓은 메모리이다
    

![image.png](attachment:8e7f54e4-e8f3-444f-88ae-4f006aa40d16:image.png)

다음과 같은 방법으로 미리 만들어진 메모리에 값을 저장하고

사용이 끝나면 메모리를 회수한다

### Slub

- Slab의 업그레이드 버전으로
- slub 객체는 partial list만을 사용하여 관리함
- 기존 slab객체에 들어있는 메타데이터 정보를 최적화한 구조가 slub이다