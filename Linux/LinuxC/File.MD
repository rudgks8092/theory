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

**Synchronized I/O**  
보통 느린 디바이스(디스크, 네트워크)를 대신하여 메모리에 캐시하고 여유 있을 때 실질적 기록을 수행한다.  
그러나 즉각적으로 기록 내용의 반영이 필요한 경우 동기화 기능이 필요하다.  
=> 느린 디바이스를 최대한 동일하게 유지하려 노력함 = 응답성이 떨어지고 시스템 성능이 떨어질 가능성 높아짐  

1. fsync, fdatasync함수를 사용한 동기화
2. open함수로 파일 열때 O_SYNC, O_DSYNC, O_RSYNC 플래그 사용


Process의 fork-exec에서 FD_CLOEXEC를 fnctl로 처리하지 않고, open에서 O_CLOEXEC를 하는 기능도 있다.


**파일은 작업이 마무리되면 반드시 닫아야 메모리 누수를 방지함**  
=> 프로세스가 최대로 열 수 있는 FD(파일, 소켓, 파이프 등)에 제한이 있음


---
### 고수준 파일 처리

장점 : C언어  표준, 추상화된 스트림에 가까운 형태로 이해하기 쉬움, 포맷팅, 버퍼링 제공  

버퍼링은 변수 <-> 버퍼 <-> 커널  2번 복사로 메모리 대역폭을 비효율적으로 사용하는 단점이 존재  
따라서, **응답성과 성능을 중시하면 저수준 위주로 하는게 좋음**  


**FILE 구조체**  
C언어에서 파일 처리를 위한 구조체  
이를 통해 얻어지는 파일 입출력 매개물 = 파일 스트림(가상화된 흐름을 표현하는 장치)   
연속된 공간에서의 가상 데이터 흐름  

```c++
FILE *fp;
if( (fp = fopen("streamfile.txt", "w")) == NULL){
	// ERROR 처리
}
// 파일 닫기
fclose(fp); 
```

고수준 파일 처리의 특징 버퍼링은 기본적으로 완전 버퍼링(Fully Buffered, Block buffering)이다.  
```c++
//라인 버퍼링 지정
setvbuf(stdout, (char*)NULL, _IOLBF, 0);
// => 개행문자를 발견하면 버퍼를 자동으로 비움 = fflush 실행

// 기본 값 완전 버퍼링
setvbuf(stdout, (char*)NULL, _IOFBF, 0);
```
**fflush로 강제로 비워야 할 경우**  
= fork나 exec 계열의 함수를 사용할 경우(데이터 순서 역전이나 파괴 가능성)  


고수준 파일 처리에서 바이너리는 fread, fwrite로 처리  
**정적 캐스팅으로 구조체 등을 받아오면, 시스템에 따른 패딩, 정렬 문제 등이 발생될 수 있음**  
이를 해결 하기 위해 mmap, 소켓 전송에는 XDR규약을 지키는 것이 좋음 (추후 설명)  

---
```c++
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <errno.h>
#include <spawn.h>
#include <sys/wait.h>
char file_log[64];
void cat_logfile();

int main(int argc, char *argv[])
{
	int 	fd0, fd1;
	FILE	*fp0;
	char	buf[64];
	snprintf(file_log, sizeof(file_log), "%s.log", argv[0]);
	if ((fd0 = open(file_log, O_CREAT | O_TRUNC | O_RDWR, 0644)) == -1) {
		/* error */
	}
	write(fd0, "1234567890abcdefghij", 20);
	cat_logfile();	/* print current file */


	// 복제
	if ((fd1 = dup(fd0)) == -1) {
		/* error */
	}
	write(fd1, "OPQRSTU", 7);
	cat_logfile();


	// 복제 open - 고수준
	if ((fp0 = fdopen(fd1, "a+")) == NULL) {
		/* error */
	}
	printf("\tfd0(%d) fd1(%d)\n", fd0, fd1);

	// fd1 오프셋 변경 후, write
	lseek(fd1, 2, SEEK_SET);
	write(fd1, ",fd1,", 5);
	cat_logfile();

	// fd0에 write
	write(fd0, ",fd0,", 5);
	cat_logfile();

	// fp0으로 READ
	int x = fread(buf, 5, 1, fp0);
	printf("\tread buf=\"%.5s\"(%d)\n", buf, x);
	fwrite("(^o^)", 5, 1, fp0); fflush(fp0);
	cat_logfile();
	fclose(fp0);
	close(fd1);
	close(fd0);
	return 0;
}

void cat_logfile()
{
	static int cnt;
	char    *argv_child[] = { "cat", file_log, NULL };
	printf("%d={", cnt++);
	fflush(stdout);
	posix_spawnp(NULL, argv_child[0], NULL, NULL, argv_child, NULL);
	wait(NULL);
	printf("}\n");
	fflush(stdout);
}
```

위 코드는 3개의 FD가 동일한 파일을 바라보는데, 이를 채널을 공유한다하고 Linked Channels라 부른다.  
lseek로 한개의 fd의 오프셋을 변경하여도, 다른 fd들도 채널을 공유하기 때문에 동일하게 변경된다.  

**fdopen(fd1,"a+")로 append 모드로 열면, 추가모드로 동작하여 항상 파일의 끝을 보게 된다. (다른 fd 포함)**  

파일을 새로 열면 Independent Channels로 열림  
=> 각각 버퍼가 따로 존재, 채널 생성으로 메모리 오버헤드 발생 가능성 있음  


**결론으로는 독립된 채널이여도 입출력 순서에 대한 결정은 운영체제에 있다.**  
**따라서, 여러 채널의 사용은 지양해야 한다.**  
=> 성능과 신뢰성을 위해서는 직렬화하는 것이 가장 좋다고 한다.  


**Tip. fflush(stdin)**  
입력 버퍼를 비울 때, 관습적으로 위의 함수형태로 호출하는데 표준이 아니기 때문에 플랫폼에 따라 다를 수 있으므로 지양하자.  


### 패딩/팩, XDR
임플리먼테이션에서 CPU가 버스오류 의 가능성때문에 보통 구조체에 짝수바이트로 패딩을 수행함  
```c++
struct my_st1{
	char str[9];
	char cnt[4];
}
// 9 + 4 = 13 Byte
struct my_st2{
	char str[9];
	int cnt;
}
// 9 + (3) + 4 = 16 Byte
```
**이러한 이유??**  
=> int가 오프셋 주소를 4바이트 경계에서 시작할 수 있도록 하기 위하여  
=> 4바이트 단위 경계의 주소를 CPU가 더 빨리 접근 가능해서  
=> 이런 주소 경계 특정 배수 단위 정렬 규칙 = XDR  


```c++
//위험한 예제 규칙
char *p_buf;
if((p_buf = (char*)malloc(sizeof(char)*65536)) == NULL)
{
	return EXIT_FAILURE;
}
memcpy(p_buf, "1234567890abc",13);
p_buf += 13;
*((long*)p_buf) = 123456;
return EXIT_SUCCESS;
```

위의 코드는 버스 오류를 발생시킨다.  
이러한 메모리 정렬 문제를 해결하기 위하여 13바이트가 아닌 16바이트를 이동 시켜야 한다.  
13~16 메모리 사이는 4바이트 정렬이 수행되어 Padding으로 3바이트가 채워진다.  



**구조체는 숨겨진 패딩이 생기고, 바이트 배열에 넣는 과정은 매우 불편하다**  

**Tip. 묵시적인 패딩과 명시적인 패딩**  
=> XDR이 만들어진 시점인 1995년에는 32bit가 보편적인 운영체제였기 때문에, 현재도 대부분 32Bit를 기준으로 4바이트 정렬을 수행함  


위 구조체 코드의 경우 묵시적인 패딩이 발생한다.  
=> 명확하게 하기 위해서는 명시적인 패딩을 사용한다.  

```c++
struct my_st{
	char str[9];
	char pad[3];
	int cnt;
}
```


또한 배치 순서에 따라 패딩 바이트를 아낄 수 있다.
```c++
// Total = 32 Bytes
struct my_st {
	char str[9];
	// 3바이트 패딩
	long cnt;
	char comment[10];
	// 2바이트 패딩
	long time;
};

// Total = 28 Bytes
struct my_st2 {
	long cnt;
	long time;
	char str[9];
	char comment[10];
	// 1바이트 패딩
};
```

offsetof(구조체,변수명)와 sizeof(구조체)로 패딩 규칙을 파악할 수 있다.  
경우에 따라서, 2바이트 멤버가 있으면 2바이트 패딩, 4바이트 패딩 다양하게 발생할 수 있다.  

**Bit 아키텍쳐에 맞는 설계가 가장 바람직하다.**  
=> 4Bytes, 8Bytes의 배수  

명시적인 코딩을 이용하여 성능을 올리기도 한다.  
=> 캐시라인 크기가 64바이트면 64바이트의 배수로 사용  
=> 가짜공유(false sharing)같은 캐시 미스를 상당수 줄일 수 있어서 성능 향상에 도움이 된다.( 추후 설명 )  
