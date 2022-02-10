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

**원자성의 중요성**  
여러 스레드에서 fprintf로 파일을 쓰려할 때, 작업 도중 중간에 끼어드는 경우 문제가 발생할 여지가 상당히 높음  
=> 저수준의 경우, 파일위치 값을 가지고 쓰지않고, 절대적인 offset에 파일을 쓰기 때문  

**Tip. 파일 출력 섞이지 않게 하려면?**  
1. 원자성을 가진 저수준 파일 입출력의 사용  
2. Lock을 이용한 출력 과정의 보호 <-> 성능 저하 가능성  
3. 직렬화를 이용하여 전문적으로 출력을 맡아하는 프로세스나 스레드 <-> 설계가 복잡하고 무거워질 수 있으나, 신뢰성 높고 응답 좋음  
4. 메모리 맵 파일(mmap)을 이용한 메모리에 쓰고 파일로 동기화 시키는 방법  


**입출력 포맷팅**  
본래 저수준 포맷팅은 sprintf, write를 이용하여, 2번의 작업과정을 거쳐야 하였으나,  
**POSIX 함수 dprintf**를 이용하여 간편해졌다.  
다만, 아직 입력에 대한 fscanf의 입력은 지원되지 않는다.


### 저수준 파일 처리
```c++
// 함수의 헤더를 모를 때는 man 페이지를 활용
#include <unistd.h>
#include <stdio.h>
#include <stdlib.h> // EXIT_SUCCESS, EXIT_FAILURE
#include <sys/types.h>
#include <sys/stat.h>
#include <errno.h>
#include <string.h>
#include <fcntl.h>

int main()
{
	int fd;
	char buf_msg[64];
	if ((fd = open("fd_test.log", O_CREAT | O_WRONLY, 0664)) == -1) {
		fprintf(stderr, "FAIL ; Open: %d(%s)\n", errno, strerror(errno));
		exit(EXIT_FAILURE);
	}

	dprintf(fd, "PID[%d} LOW LEVEL FILE HANDLE\n", getpid());
	strcpy(buf_msg, "write:Test Message\n");
	write(fd, buf_msg, strlen(buf_msg));
	close(fd);
	return EXIT_SUCCESS;
}
```

