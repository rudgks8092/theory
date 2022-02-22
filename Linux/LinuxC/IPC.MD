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

읽기 전용 = 실행 파일, 동적 라이브러리 로딩(strace 확인)  

위의 ls는 mmap으로 중간중간 읽기 전용 설정 파일을 맵핑하고 있다.  
이렇게 작성된 mmap은 모든 사용자가 공유하여 효과적으로 사용 가능  
또한, 포인터 변수에 맵핑되어 lseek없이 바로 원하는 주소를 읽을 수 있음  


mmap을 이용하여 파일에 쓸 경우, 동기화 작업이 진행되어 원하는 상황이 되지 않을 수 있음  

mmap은 위처럼 **읽기 전용**에 특화되어 있지만,  
쓰기의 장점도 존재한다.  
1. write + 파일 보다 더 빠르고 일관성 유지하기 쉬움
2. 시스템 콜 없이 커널이 자동 동기화 시도
3. IPC용으로 활용이 가능
4. 하드웨어 장치와 매칭한 경우, 하드웨어 메모리 접근에 용이  

### 추가 특징
**Lock 매커니즘이 필수**  
**대응된 메모리 맵의 크기를 넘어서는 경우, 파일에 영향주지 않음**

---
**mmap 수행 방식**  

먼저 파일을 열어 맵핑시켜야하는데, 대응시킬 메모리보다 커야함  
=> 작을경우, ftruncate를 이용해서 키워야 함


1. 공유(shared) 방식 = 지속 동기화
2. 사설(private) 방식 = 최초 동기화

공유 방식은 커널에서 지속적으로 동기화하여 모든 프로세스가 동일한 내용을 보도록 한다.  
사설 방식은 mmap이 생성된 뒤에 프로세스가 독자적으로 사용하는 메모리 공간으로 전용될 수 있음  
=> 사설 방식은 프로세스 고유 메모리 공간으로 전환되어 스왑 아웃 가능한 페이지가 됨  
=> 메모리 낭비 or 성능상의 이점 사라질 수 있음

**사설mmap에서는 스왑 금지를 mlock으로 해야함**  
메모리조언(posix_mdvise)를 이용하는 방법도 있음(뒤에 작성)  


mmap = thread-safe (내부 lock mechanism)  
=> 공유받은 mmap이 최종 쓰여진 것이란 보장은 없기에 msync로 동기화 후, 읽어야 함  

사용한 mmap은 munmap으로 해제해야 하는데, 다른 프로세스 사용여부 체크와 msync 이후에 하는 것이 좋다.  


```c++
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <fcntl.h>
#include <errno.h>
#include <string.h>
#include <sys/mman.h>
#include <sys/stat.h>

int main(int argc, char *argv[])
{
	int		fd, sz_page;
	char	*p_map;
	struct stat	stat_buf;

	if (argc != 2) {
		printf("Usage : %s <mmap file>\n", argv[0]);
		return EXIT_FAILURE;
	}
	printf("System Page Size = %d\n", (sz_page = sysconf(_SC_PAGESIZE)));

    // 메모리 대응 파일 열기
	fd = open(argv[1], O_RDWR|O_CREAT, 0664);
	if (fd == -1) {
		printf("Fail: open(): (%d:%s)\n", errno, strerror(errno));
		return EXIT_FAILURE;
	}

	if (fstat(fd, &stat_buf) == -1) {
		printf("Fail: fstat(): (%d:%s)\n", errno, strerror(errno));
		return EXIT_FAILURE;
	}
    // 적합하게 사이즈 늘리기
	if (stat_buf.st_size < sz_page) {
		ftruncate(fd, sz_page);
	}

    // mmap 수행
    // 1번째 : 리턴 받기 원하는 주소 시작 번지(0이면 auto)
    // 고정주소를 원하면 MAP_FIXED 플래그를 주고 페이지 크기의 경계(배수)를 입력
    // MAP_SHARED, MAP_PRIVATE 는 동시 사용 불가
    // 기타 여러가지 있으므로 검색 필요
	p_map = mmap((void *)0, 64, PROT_READ|PROT_WRITE, MAP_SHARED, fd, 0);
    // mmap 실패
	if (p_map == MAP_FAILED) {
		printf("Fail: mmap(): (%d:%s)\n", errno, strerror(errno));
		return EXIT_FAILURE;
	}
	sprintf(p_map, "%s %p", "Testing synchronized mmap.\n", p_map);

	/* synchronizing mmap. */
	printf(">> Synchronizing Memory Mapped File.\n");
	if (msync(p_map, 64, MS_SYNC) == -1) {
		printf("Fail: msync(): (%d:%s)\n", errno, strerror(errno));
		return EXIT_FAILURE;
	}

	printf(">> Unmapping Memory Mapped File.\n");
	if (munmap(p_map, 64) == -1) {
		printf("Fail: munmap(): (%d:%s)\n", errno, strerror(errno));
		return EXIT_FAILURE;
	}
	if (fsync(3) == -1) {
		printf("Fail: fsync(): (%d:%s)\n", errno, strerror(errno));
		return EXIT_FAILURE;
	}
	printf(">> Closing file.\n");
	close(fd);

	return EXIT_SUCCESS;
}

```

#### MMAP 종류
1. MAP_SHARED  = 공유
2. MAP_PRIVATE  = 사설(보통 읽기전용에 사용)
3. MAP_FIXED  = 고정 시작 번지 지정
4. MAP_ANONYMOUS  = 장치와 연결되지 않는 임시 mmap
5. MAP_HUGETLB  = 대용량 처리
6. MAP_LOCKED  = 페이지 락 이용(mlock)
7. MAP_UNINITIALIZED  = mmap으로 할당된 공간 초기화 X(성능 중시)


msync는 mmap에 자주 활용되는 함수로 비동기, 동기 등의 설정이 가능하다.  

munmap의 행위는 연결을 끊는 행위  
=> 다른 프로세스가 사용하지 않을때 시스템이 mmap을 제거함  


---
### 메모리 사용 패턴 조언(memory advise)
mmap이나 shared memory는 메모리를 어떻게 다룰지에 대한 힌트 명시가 가능하다

posix_madvise, madvise, posix_fadvise ...


**실행 파일의 로딩 과정은 mmap을 통해서 한다(앞의 ls)**  
=> 실행파일이 다른 파일을 읽는 경우는??  

mmap으로 할 수 있으나, 프로그래머의 의도에 따라서 읽는 위치가 다를 수 있다.  
=>처음? 끝? 중간? 운영체제의 캐싱을 위한 힌트를 위해 사용하게 된다.  

--- 
madvise는 BSD에서 도입된 함수로 대부분의 리눅스 유닉스에서 지원  

POSIX 표준화를 통해 posix_madvise로 제공  

이 또한 SYSV 시스템의 기능들처럼 기능면에서는 madvise가 좋다.  
하지만 호환성에서는 표준함수인 posix_madvise가 좋다.  
( 산업용 임베디드나 구형 시스템에서는 함수가 아무일도 안하는 경우가 있음 )  


```c++
int posix_madvise(void *addr, size_t len, int advice);
```
주소의 시작부터 len만큼의 공간에 적용  

- POSIX_MADV_NOMAL = 기본 값
- POSIX_MADV_SEQUENTIAL = 순차 접근(순서대로 Prefetching - 대용량 적합)
- POSIX_MADV_RANDOM = 랜덤 접근
- POSIX_MADV_WILLNEED = 빠른 시간 내 사용(되도록 메모리에 상주하도록)
- POSIX_MADV_DONTNEED = 사용안함

 posix_fadvise는 파일에 조언 적용


 ---
 ## SysV, Posix IPC
 
 ### XSI IPC 형식의 특징

 구식이지만, 활발하게 쓰임 ( 유닉스 파생된 모든 운영체제가 지원하기 때문 )  

함수 외 명령어로 (ipcs, ipcrm) 확인 및 제거 기능도 지원한다.  

- IPC key
- IPC ID (고정이 아님 - IPC key를 통해 획득 = 파일기술자와 비슷)
- 소유권자
- 소유권한(mode)

**위 속성을 모든 XSI IPC가 가지고 있다**  
IPC_PRIVATE로 사설 IPC도 있음 - IPC ID를 알아야지만 접근 가능함(key가 무의미한 값)  

![image](https://user-images.githubusercontent.com/85000118/154230666-1aca3131-897e-4862-869b-aaffd3251cb6.png)

**IPC 사용 과정**

msgget을 통하여 입력된 값(key)를 토대로 만들 수 있지만,  
ftok를 이용하여 파일경로를 토큰형태로(Hash) 만들 수도 있다.  

**Key는 Unique하게 만드는 것을 권장**

IPC_PRIVATE를 설정하면 랜덤한 값이 생성된다.  
=> 생성된 ID는 접근 권한(mode)에 문제가 없다면 다른 프로세스도 접근이 가능하다.  


```c++
// 파일경로와 101을 조합한 IPC Key 계산
key_shm = ftok("usr/local/bin/ipc_daemon",101);
int id_shm = shmget(key_shm, size_shm, flag_shm);

// IPC_PRIVATE
int id_shm = shmget(IPC_PRIVATE,size_shm, flag_shm);

// id_shm만 알고있다면 다시 shmget하지 않고, 접근이 가능하다(unique 기술자)
```

ipcs, ipcrm 명령어를 통해 생성된 IPC의 확인과 삭제가 가능하다.  
ipcs의 결과에 Key가 0x00000000이면, IPC_PRIVATE로 만들어져있는 경우이다.  

ipcrm으로도 삭제할 수 있고, 함수로 프로그램 내에서 각 ctl 함수를 사용하면 삭제가 가능하다.  

POSIX IPC는 ipcs같은 관리툴이 별도로 없다.  
그러한 단점들로 기존에는 덜 쓰였지만, pthread등 POSIX 인터페이스와 포팅하기에 좋아, 신규 프로젝트들에서 사용할 경우 POSIX를 쓰는것이 좋다.  

리얼타임 확장  라이브러리 librt(POSIX API)  
쓰레드 관련 라이브러리 pthread  
POSIX IPC는 리얼 타임의 확장 기능 컴파일에서 -lrt, -lpthread 처럼 링킹해줘야 함  


---
## 공유 메모리(shared memory)
**함수 콜 없이 주소 번지의 포인터 변수로 직접 접근하여 가장 빠른 성능을 갖는 IPC 메커니즘**  

코드 단계에서 배타적 접근을 보장해야 함  
=> Semaphore, MUTEX, rwlock, spinlock 등을 활용한 Lock 메커니즘  

POSIX는 정수형 기술자(descriptor)를 사용하는 형태  
=> 내부적으로는 별도의 식별자로 처리됨  
=> 식별자를 사용하는 코드는 대체로 비슷하여 쉽게 사용(소켓, 파이프 등)  

---
## XSI 공유 메모리
System V 의 공유 메모리 Key를 통한 접근방식 사용

- shmget = IPC ID 얻기 ( 없으면 생성 가능 )
- shmat = IPC ID를 통한 연결 (메모리 연결 주소 반환)
- shmdt = 연결 해제 - detach
- shmctl = 공유 메모리 조작 ( 제거, 메타 데이터 얻기 )

**사용중인 메모리를 삭제하려하면, 삭제 예약상태가 되어 detach되면 삭제됨**  
=> 삭제 예약은 리눅스의 기능, 유닉스는 안될 수 있음  


```c++
#include <errno.h>
#include <stdio.h>
#include <string.h>
#include <sys/ipc.h>
#include <sys/shm.h>

int sysv_shmget(void **ret, char *tok, key_t shm_fixkey, int size, int user_mode);
int sysv_shmrm(int shm_id);

/* Name : shm_init
 * Desc : init shared memory
 * Argv : tok(path), shm_fixkey(key), int size(byte), user_mode(octal notation)
 * Ret  : shm ID (if success), -1 (if fail)
 *        ret (return address)
 */
int sysv_shmget(void **ret, char *tok, key_t shm_fixkey, int size, int user_mode)
{
	key_t	shm_key;
	int		shm_id;
	char 	buf_err[128];

	if (ret == NULL || size < 0) { /* error */
		return -1;
	}

	if (tok != NULL) {
		if ((shm_key = ftok(tok, 1234)) == -1) {
			return -1;
		}
	} else {
		shm_key = shm_fixkey;  /* Fixed IPC key */
	}

	if ((shm_id = shmget(shm_key, size, IPC_CREAT|IPC_EXCL|user_mode)) == -1) {
		if (errno == EEXIST) { /* already exist shm, just get ipc id */
			shm_id = shmget(shm_key, 0, 0);
		}
	}
	if (shm_id == -1) {
		strerror_r(errno, buf_err, sizeof(buf_err));
		fprintf(stderr,"FAIL: shmget():%s [%d]\n", buf_err, __LINE__);
		return -1;
	}

	// shmat 인수로 Flag SHM_RND 등이 있음
	// SHM_RND는 페이지 경계 계산을 반내림으로 자동 처리해준다.
	if ((*ret = shmat(shm_id, 0, 0)) == (void *)-1) { /* error */
		strerror_r(errno, buf_err, sizeof(buf_err));
		fprintf(stderr,"FAIL: shmat():%s [%d]\n", buf_err, __LINE__);
		return -1;
	}

	return shm_id;
}

/* Name : sysv_shmrm
 * Desc : remove shared memory
 * Argv : int shm_id or key
 * Ret  : 0 (if success), -1 (fail)
 */
int sysv_shmrm(int shm_id)
{
	int		ret;
	char	buf_err[128];
	// IPC_STAT, IPC_SET, IPC_INFO 등이 있음
	if ((ret = shmctl(shm_id, IPC_RMID, NULL)) == -1) {
		strerror_r(errno, buf_err, sizeof(buf_err));
		fprintf(stderr,"FAIL: shmctl():%s [%d]\n", buf_err, __LINE__);
		return -1;
	}
	return ret;
}
```

---
### 공유 메모리 시스템 설정
공유 메모리 관련 시스템의 설정들이 있다.

- kernel.shmmni = 시스템 생성가능 공유 메모리 최대 개수
- kernel.shmmax = 공유 메모리 1개 최대 크기(bytes)
- kernel.shmall = 시스템 할당 가능 전체 공유 메모리 페이지 개수

```
sysctl -a |grep shm // 공유메모리 관련 정보 확인
```
몇몇 설정이 더 있었지만, 사라져가는 추세이다.  
위 코드들을 이용하여 아래와 같이 Make로 라이브러리 생성 가능  

```makefile
#----------------------------------------------------------------------------
# Prefix and Output target
#----------------------------------------------------------------------------
PREFIX_DIR  = ../../..

OUT		= libsysvipc.a \
			sysv_shm \
			sysv_sem sysv_sem_waitforzero \
			sysv_msg  sysv_msg_regex sysv_msg_info \
			sysv_sem_counting sysv_sem_graph

include $(PREFIX_DIR)/common_opt.mk

LDLIBS 		+= -lsysvipc
LOADLIBES 	+= -L$(PREFIX_DIR)/lib
#----------------------------------------------------------------------------
# Sources
#----------------------------------------------------------------------------
SYSVIPC_FILES = lib_sysv_sem lib_sysv_shm lib_sysv_msg

SYSVIPC_SRC      = $(addsuffix .c,       $(SYSVIPC_FILES))
SYSVIPC_OBJ      = $(addsuffix .o,       $(SYSVIPC_FILES))

#----------------------------------------------------------------------------
# Default target.
#----------------------------------------------------------------------------
.PHONY: all clean dep
all: $(OUT)

#----------------------------------------------------------------------------
# Suffix rules
#----------------------------------------------------------------------------
include $(PREFIX_DIR)/common_rule.mk

#----------------------------------------------------------------------------
# Static pattern rule
#----------------------------------------------------------------------------

# Library Compilation (Position Independent Code)
$(SYSVIPC_OBJ): %.o: %.c
	$(CC) -c $(CFLAGS) $(CPPFLAGS) $(CPICFLAGS) $< -o $@

#----------------------------------------------------------------------------
# Build library
#----------------------------------------------------------------------------

libsysvipc.a : $(SYSVIPC_OBJ)
	@echo "++ Building XSI(SysV) IPC Wrapper library... ($@)"
	$(AR) $(ARFLAGS) $@ $(SYSVIPC_OBJ)
	@ranlib $@
	@echo "++ install the archive to $(LIB_DIR)"
	@if [ ! -d $(LIB_DIR) ]; then \
		mkdir $(LIB_DIR); \
	fi
	@mv $@ $(LIB_DIR)
	@rm $^
	@echo ""

libsysvipc.so : $(SYSVIPC_OBJ)
	@echo "++ Building XSI(SysV) IPC Wrapper library... ($@)"
	$(CC) -shared -o $@ $(SYSVIPC_OBJ)
	@echo "++ install the shared object to $(LIB_DIR)"
	@mv $@ $(LIB_DIR)
	@echo ""

#----------------------------------------------------------------------------
# Build sample program
#----------------------------------------------------------------------------

#----------------------------------------------------------------------------
# others
#----------------------------------------------------------------------------
include $(PREFIX_DIR)/common_target.mk

```

