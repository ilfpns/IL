> **on-disk**
[[Linux]]
---

⇒ 파일시스템의 디스크 상 물리적으로 데이터 구조를 가리키는 용어

⇒ OS가 비휘발성 저장장치에 실제로 사용하는 구조 데이터

### on-disk data

- 디스크 블록에 저장된 비휘발성 데이터 + 메타데이터 구조 전체

> **어떤 이유로 사용할까?**

---

- 디스크에 기록되는 구조는 표준화된 포맷으로 다른 OS/유틸리티가 해석 가능해야 한다
- 이 구조가 손상되면 파일시스템이 마운트 불가/데이터 손상이 됨
- 파일시스템 드라이버는 on-disk 구조를 읽어서 in-memory 구조로 변환함

### on-disk 종류 (ext4 기준)

- superblock
    
- inode table
    
- data block
    
- journal (JBD2)
    
- ext4란?
    
    리눅스가 대표적으로 사용하는 파일 시스템
    

|on-disk 요소|설명|
|---|---|
|**Superblock**|파일시스템 전체 메타정보(크기, 블록사이즈, 포인터 위치 등)|
|**Block group descriptor**|그룹별 위치 정보/할당 정보|
|**Inode table**|각 파일의 메타데이터(inode)의 실제 저장체계|
|**Block bitmap / Inode bitmap**|사용/미사용 블록 또는 inode 상태 비트맵|
|**Data blocks**|파일 내용이 실제 저장되는 블록|
|**Journal / Checkpoint**|일관성 보장 로그(저널, Btrfs CP 등)|

- in-memory
    
    in-memory는 비휘발성 메모리이다
    
    커널은 on-disk 데이터를 불러와 in-memory로 변환해 사용하고, 필요 시 다시 디스크에 write-back 한다