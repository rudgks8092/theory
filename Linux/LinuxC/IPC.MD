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


POSIX 