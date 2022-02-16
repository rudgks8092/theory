# IPC(Inter-Process Communication)

**IPC - 프로세스 사이에서 통신하는 메커니즘의 통칭**  

ex. 파일, mmap, 세마포어, 공유 메모리, 메시지 큐, 파이프, 소켓 등


UNIX 계통에서는 보통 초기 SYSV 유닉스 계열에서 만들어진 3가지 기법을 말함  
1. 공유 메모리
2. 세마포어
3. 메시지 큐

이러한 SysV형식을 따르는 경우를, SVIPC, XSI IPC라고도 한다.  
POSIX에서는 POSIX IPC라고 함  


과거에는 파일을 통한 통신이 대부분이었다. 저장하고 불러와 사용하는...  

--- 
System V 인터페이스 표준이 만들어지면서 이벤트 전달을 위하여 signal이 도입됨  
=> 본래는 프로세스 감시 및 종료 목적이었음  
=> 시그널 종류가 늘어감에 따라 이벤트 통지(event notification)  
=> 상태 변환 기능이 가능하고, 시그널 처리를 프로그래밍 가능해짐

---
이후, 파이프(pipe)가 80년대 표준화되면서 등장함  
=> 2개의 프로세스의 출력과 입력을 연결하여 하나의 공유페이지 기능 제공  
=> ex. ps -ef |grep sunyzero | cut -b 49 -  
=> ps의 출력은 grep의 입력과 연결되고, grep은 다시 cut과 연결되어 가공함

이러한 임시 파이프 외에도  
특정 파일 위치를 지정해 사용할 수 있는 특수한 형태의 named pipe도 있다.

---
## 공유 자원과 IPC
80년대 멀티태스킹이 이슈되어 다양한 기법이 추가됨

대표적으로 **Shared Memory, Semaphore, Message Queue**  

SysV 유닉스에서 통합시켜 SysV IPC라고 불림  
후에 유닉스 통합되어 POSIX의 X/Open Interface(XSI)에 통합되며 XSI IPC가 됨  
<br>
<br>
**※POSIX가 별도로 표준 함수를 만든 이유**  
- Sys V 형식이 기존 POSIX API와 너무 달라서

--- 

이후에 공유 메모리 기능을 확장하여 파일과 메모리 페이지를 맵핑하여 성능을 높이기 위하여 mmap이 나왓다.  
=> mmap은 여러 프로세스가 파일과 메모리를 1:1로 대응시켜 훨신 세련된 데이터 교환이 가능  
=> 파일 페이징이 수행되지 않아, 속도 저하를 방지할 수 있음  

정적 - 통신 매개체 = shared memory, mmap  
동적 - 통신 = message queue, Socket  

POSIX보다 XSI IPC를 오래 써와서 POSIX보다 XSI쪽이 자원도 정보도 많다.  
단, POSIX는 POSIX Thread(pthread)와 혼용할 때 더 편리하여 늘어나고 있는 추세다.

--- 

### 원격 호스트와의 네트워크 통신

IPC는 하나의 호스트 내에서의 통신 방법  
=> **BSD계열 유닉스에서 Socket을 제안하여 만들어짐**  
=> AT&T의 SysV도 TLI(Transport Layer Interface)를 제안했지만 결국 Socket으로..  

둘 다 사용은 가능하지만, 사실상 소스코드 레벨의 호환을 위하여 Socket 사용 추천  

**소켓은 내부 통신 전용으로도 사용 가능**  
=> Unix Domain Socket <-> Network Domain Socket (쉽게 전환 가능)  


확장성과 성능 = Trade off 관계  
=> 소켓이 내부 통신은 가능하지만, 확장성이 높은 만큼 성능이 떨어진다  
GET : 뛰어난 확장성(복잡한 구조 + 데이터 처리)  
LOSE : 상대적 무거움(여러가지를 지원하게 됨으로써 비대해짐)

**IPC의 알려진 속도**  
공유 메모리 - 메시지 큐 - ... - 소켓  

---
### mmap(Memory Mapped I/O)
장치 or 파일 => 메모리 대응 시키는 기법  

1. 복잡한 단계 없이, 메모리 주소에 직접 쓰거나 읽어 빠름
2. 불필요한 메모리 페이징 방지
3. 내용 접근에 시스템 호출 불필요하여 성능 향상

**strace ls를 통한 mmap 확인**

![image](https://user-images.githubusercontent.com/85000118/154199776-9cdd33f3-8637-4e2e-95a6-dc890a536c92.png)

운영체제가 메모리가 부족해지면 디스크로 page out 실행  
=> But, 읽기 전용으로 열린 파일에서 읽은 내용 = 페이지 아웃 불필요  
=> 이미 디스크에 존재하고 있기 때문(필요할 때 다시 메모리 로드)  

