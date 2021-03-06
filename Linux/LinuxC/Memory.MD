# 메모리

메모리는 나누는 기준에 따라 여러가지  
(ex. 가상메모리, 공유메모리, 데이터 영역, 코드 영역 등)


### 프로세스 메모리
프로세스가 메모리에 로딩될 때,  

1. 텍스트(text)  
=> 프로그램 실행코드
2. 데이터(data)  
=> 전역변수나 정적 심볼 사용 - 조금 더 많은 영역 분리 가능  
=> 기본적인 3가지
    1. .rodata = 읽기 전용으로 초기화(ex. 포맷스트링, 상수문자열)  
    2. .data = 읽기/쓰기가 가능한 영역으로 초기화(ex. 초기값 있는 전역변수)  
    3. BSS = 초기화되지 않는 영역  
    => C의 startup 함수가 main 실행 전 0으로 초기화
3. 스택(Stack)  
=> 특정 스코프에서 자동 생성, 파괴되는 내용
4. 힙(Heap)  
=> 동적으로 생성 및 파괴하는 내용


**스택 메모리의 특징 단점**  
스택은 필요에 따라 할당하면서 키우는데, 반환 이후에 별도로 해제하지 않는다.  
=> ex. 1000 사이즈의 스택을 사용하는 함수 호출이후 리턴하여도 늘어난 크기가 줄어들지 않음(사용은 가능함)  

이러한 이유로 스택 크기를 제한할 수도 있다. (1MB 이상이면 프로그램 종료)  
=> 전역변수는 data 영역이기에 1MB이상 사용 가능  

또한, 쓰레드는 메인 쓰레드와 별도의 스택 공간을 사용함  


### VLA(Variable Length Array)
본래 가변 크기의 변수 선언은 힙으로 사용했었으나, C99부터 가능해졌음  
```c++
int val(int vlen)
{
    char buf[vlen];
}
```

VLA는 블록(스코프) 안에 존재햐아 사용 가능하다. (앞의 메모리 영역 문제)  
실행파일에 objdump나 readelf를 사용하면 배치 위치를 볼 수도 있다.  


## 메모리 락
페이징 락 기능  
=> 메모리 페이지에 대해 페이징을 금지하는 기능(항상 램에만 있도록 강제함)  
보통의 프로세스는 paging in/out(swap in/out)으로 메모리 효율을 위해 디스크와 교환작업을 수행함(페이징)

**사용이유**  
1. 실시간 처리가 중요한 어플리케이션 메모리 접근 지연 방지(보통은 디스크 스왑될 수 있음)  
2. 보안이 중요한 데이터의 디스크 저장 방지(디스크의 비휘봘 고려)  

mlock, mlockall, munlock, munlockall

인자 주소 위치부터 길이 만큼 lock 시키는 형태  
=> 주의사항으로 페이지 크기의 경계에서 시작해야함  
=> sysconf(_SC_PAGESIZE)로 확인 가능하며 보통은 4KiB  
=> 페이지 경계, 크기의 배수가 아니면 오류 발생 가능  

mlockall은 프로세스 메모리 전체를 lock하는데, flag 지정 가능함  
1. MCL_CURRENT = 현재 프로세스에 할당된 페이지 전체
2. MCL_FUTURE = 앞으로 할당되는 모든 페이지

ulimit -l로 확인한 시스템 한계를 넘으면 에러가 발생함