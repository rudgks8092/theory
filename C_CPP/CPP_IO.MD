### C++ I/O

C 언어 - printf, scanf  
C++ - Stream  

**차이 ?**
1. 객체지향에 어울리는가?
2. 에러 처리 기능이 있는가?
3. 커스텀 데이터 타입을 다룰 수 있을 정도로 유연한가?

스트림 ?(cout) = 공장 컨베이어 벨트 ( cout의 c는 문자를 의미 )  
**콘솔 <-- 스트림 <-- 프로그램**  
스트림에 변수를 올리면 콘솔에 표시됨  

스트림마다 방향, 출발지src, 목적지dest 지정  

![image](https://user-images.githubusercontent.com/85000118/153137400-5a949705-be18-4588-ab36-681fd1ea0ef0.png)

**Buffered = 버퍼에 저장 후, 블록 단위 전송** => flush() 메서드로 데이터를 목적지로 전송  
**Unbuffered = 데이터 들어오자마자 전송**

Buffer 필요성 ? => 파일같은 대상에 입출력 수행할 때 블록 단위로 묶어서 보내는 것이 효율적


GUI는 콘솔 잘 사용 안하여 스트림 사용하지 않을 수 있음  
입력 스트림은 입력장치에 연결, 출력 스트림은 출력장치에 연결되어 있음  

스트림은 현재 위치정보를 담고 있음 (읽기나 쓰기를 수행할 위치)  





스트림의 개념  
데이터의 입력 or 출력 모두 적용 가능 ex. 네트워크 클래스, MIDI장치 접근 등  
File Stream, String Stream 등 ( C에서 fprintf, fscanf, sprintf 등 )  


### 출력 Stream 기초
<osstream> 헤더 (iostream헤더에는 입출력 모두 정의되어 있음).  

**cout = CPP 기본 built-in 내장 스트림으로 콘솔(표준출력)에 값을 씀**  
  ```c++
  // cout 기본 사용법
  int i = 7;
  cout << "Value i = " << i << endl;
  // 위 코드는 아래와 동일
  cout << "Value i = " << i << "\n";
  cout.flush()
  ```
  
  std::endl은 많으면 버퍼 비우는 연산을 자주 성능이 떨어질 수 있음
  
  
 ### 출력 Stream 제공 메서드
  

  
