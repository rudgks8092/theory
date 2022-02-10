### 파일 처리

유닉스 계열의 파일처리  
1. 저수준 파일 처리(Low Level File Handling) = 유닉스 계열에서만 지원
2. 고수준 파일 처리(High Level File Handling) = C언어 표준  


**Tip. 파일 시스템 이벤트 감시 inotify**  
리눅스(유닉스X - Man페이지로 확인 필요)  
파일 시스템 이벤트를 감시하는 기능  


저수준 파일 처리의 파일 기술자(FD)는 소켓, 디바이스 핸들링 등 모두 동일한 방식 사용  

저수준 파일 처리는 POSIX IPC 관련 함수가  서로 비슷함  

```c++
// 저수준 파일 처리
int fd = open("lsp.txt",O_CREAT | O_RDWR | O_EXCL, 0644);
// POSIX 공유메모리 열기
int shm_fd = shm_open("/pshm_lsp, O_CREAT | O_RDWR | O_EXCL, 0644);
```

**저수준 파일 처리**  
- 비직관적인 함수 형태
- 유닉스 및 호환 계열에서만 사용 가능
- pread, pwrite를 통해 원자적(atomic) 실행을 보장

**고수준 파일 처리**  
- 직관적이고 사용이 편리한 함수 형태
- C언어를 지원하는 모든 플랫폼 사용 가능
- 라이브러리 레벨의 버퍼링이 있음

