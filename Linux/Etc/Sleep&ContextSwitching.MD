# Sleep과 Context Switching 관계

보통 Sleep을 수행하면 다른 프로세스(스레드)에 CPU를 점유를 넘긴다.  
Sleep(0)도 그냥 넘기는 것 같음  

Sleep은 2가지 종류로 나뉜다.
1. Sleep
2. Busy Wait

보통 Sleep을 통해 넘기는 과정(Context Switching)보다 그냥 CPU를 점유하고 기다리는 것이 더 효율적인 경우  
=> 커널에서 처리 방법에 따라 다를 수 있음  


https://m.blog.naver.com/PostView.naver?isHttpsRedirect=true&blogId=kimbeast&logNo=60052829761
https://groups.google.com/g/comp.programming.threads/c/VhY8N-F_1k8

위 글을 참고하였을 때, usleep으로 10ms 이하의 sleep은 Context Switching 과정에 소요되는 시간이 많아 실질적으로는 Busy Wait이 될 가능성이 높다.  

따라서, 의도랑 다르게 해당 프로세스가 CPU를 점유하여 다른 스레드가 작업을 하지 못하게 Block하게 될 수 있다.

