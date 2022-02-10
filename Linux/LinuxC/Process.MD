### LinuxC Process 관련 API

1. fork
**fork = 자식 Process를 낳는?? 함수**  
```c++
pid_t fork(void);
// ret 0 => 자식 프로세스가 리턴받음
// ret > 0 => 자식 프로세스의 pid
// ret -1 => 에러
```
호출된 후, 자식과 부모는 그 시점에서 각각의 메모리를 가지고 동작함 ( 메모리 그대로 복사 ) 
따라서, 리턴 값에 따라 다르게 분기함  

2. vfork  
**fork 이후 exec함수로 다른 프로그램을 실행하는 과정을 간략화한 함수**
fork의 메모리 복제 과정을 가볍게 하기 위함  
다만, 요즘의 fork는 copy-on-write 도입으로 메모리를 사용할 때 복사가 되어 성능차이가 없음

3. exec   
**exec는 현재 실행 중인 프로세스 이미지를 다른 이미지로 대체**  
실행코드는 교체, PID,PPID, FD 등의 프로세스 정보는 유지 

execl, execlp, execle, execv, execvp, execve 등 인자에 따라 여러 종류가 있음  
```c++
if(execl("/bin/ls","ls","-al", NULL)==-1){
  perror("execl");
}
// 정상 동작하면 아래는 실행 될리가 없다.
cout << "EXECL EXECUTE" <<endl;
return 0;
```

fork-exec 과정에서 보통은 자식이 부모의 FD를 복제함  
그러나, fcntl로 FD_CLOEXEC 플래그 지정하면 exec에서 FD가 닫히게 됨(close-on-exec)   

--- 

```c++
//Parent Sample
#define APPLY_FD_CLOEXEC

	pid_t pid_child;
	int fd = open("Text.txt", O_WRONLY | O_CREAT | O_APPEND, 0644);
	if (fd == -1) {
		perror("FAIL:OPEN");
		exit(EXIT_FAILURE);
	}
	dprintf(fd, "Parent[%d]:Open log file(fd=%d)\n", getpid(), fd);
	/*
		dprintf(fd, format, ...);
		유닉스에서 High-Level, Low-Level 파일처리가 있음
		고수준 (fpintf, fscanf) :효율좋음, 응답속도안좋음, 버퍼 사용
		저수준(read, write) : 효율안좋음, 응답속도빠름, 버퍼 사용 안해서 CPU 사용률 증가
		write를 이용할 경우, 포맷팅이 안됨 - sprintf + write 형태
		이를 위한 저수준 파일 입출력
		dprintf
	*/
#ifdef APPLY_FD_CLOEXEC
	int ret_fcntl;
	if ((ret_fcntl = fcntl(fd, F_SETFD, FD_CLOEXEC)) == -1) {
		perror("FAIL : fcntl(F_SETFD, FD_CLOEXEC)");
		exit(EXIT_FAILURE);
	}
#endif
	// fork-exec
	char *argv_exec[] = { "forkexec_childE", (char*)NULL };
	switch ((pid_child = fork()))
	{
	case 0:
		//Child
		execv(argv_exec[0], argv_exec);
		break;
	case -1:
		//Error
		perror("FAIL : FORK");
		break;
	default:
		//Parent
		wait();
		break;
	}
	printf("Parent[%d]:Exit\n", getpid());
```
---
```c++
//Child Sample
dprintf(STDOUT_FILENO, "Child = %d\n", getpid());
dprintf(3, "CHILD fd3 %d\n", getpid());
close(3);
dprintf(STDOUT_FILENO, "CHILD EXIT\n", getpid());
```
---

Child에서 fd 3으로 dprintf를 하였을 때, FD_CLOEXEC가 없다면 열리지 않음   
파일 복제 오버헤드를 피하려면 FD_CLOEXEC를 쓰면 되지만 **posix_spawn**을 사용하는게 더 좋음   


**system 함수는 셸을 시켜 명령어 실행 (fork-exec의 간단 구현 버전)**	  
중요 차이   
1. 실행 중 부모프로세스 정지
2. 자식 프로세스의 시그널 SIGCHLD, SIGINT, SIGQUIT가 무시됨

