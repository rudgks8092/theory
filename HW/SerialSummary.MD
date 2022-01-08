**시리얼 통신**

- 직렬 통신 ( 시리얼 )  과거 외부 통신
ex. COM Port 마우스, 조이스틱, 외장형 모뎀 등

- 병렬 통신 ( 페러렐 )  과거 내부 통신
ex. ISA, EISA, PCI 내부 버스들

![image](https://user-images.githubusercontent.com/85000118/136369162-d731dea7-a8b3-466d-a266-2f8579692049.png)

RS-485 통신은 산업용 필드버스 디바이스 네트워크에서 여전히 사용됨


**동기, 비동기 전송**
클럭과 같은 동기 신호의 유무

*클럭 신호 없이 RZ(Return to zero) 형태의 데이터 파형과 같이 임베디드 클럭이나 프리앰블(Preamble)과 같은 신호로 동기를 맞춰 통신하는 동시 전송 방식도 있음

*비동기 통신은 각각 로컬 소스를 가지고 시작 구별자( Start Delimiter )를 인지하여 데이터를 송수신
ex. RS-232, RS-422, RS-485 - 1970년대 후반에 개발되어 아직까지 널리 쓰임

비동기
1. 통신 속도, 데이터 프레임 포맷 미리 정의 ( 양쪽 모두 )
2. 문자단위 전송 ( 1번에 1글자)
3. START비트와 STOP 비트로 하나의 문자씩 동기를 맞춤

동기

1. 동기 유지를 위해 클럭같은 타이밍 신호를 계속 공급 or 동기 유지를 위한 동기 문자를 전송
2. 한번에 데이터와 제어 정보를 포함한 큰 크기의 텍스트 단위 문자 프레임 전송하는 고속 전송 방식
3. 문자의 비트 수와 같은 동기 문자를 보냄

![image](https://user-images.githubusercontent.com/85000118/136370468-0372532d-b08d-4afd-b292-af353d38d870.png)

![image](https://user-images.githubusercontent.com/85000118/136370626-afb9394e-f430-40bd-beae-192fc28ee268.png)

DTE = Data Terminal Equipment 데이터 단말 장치
 - 데이터 생성 및 처리하는 단말 장치 ( ex.  컴퓨터 )
DCE = Data Communication Equipment 데이터 통신 장비
 - 데이터 전송 관련 장비 (ex. 신호변환기 )
 - 다른 관점으로는 데이터 회선 종단 장치 (Data Circuit_Terminating Equipment) - 데이터 회선 끝에 존재하는 장치로 아날로그 통신 회선의 경우 변복조기로서 모뎀 같은 경우,
 - 디지털 전송 회선을 이용하는 경우는 DSU(Digital Service Unit)

**비트 단위 데이터 전송**
현재 컴퓨터는 8비트에서 64비트까지 발전한 이후, 지터 등 여러가지 기술 문제로 한계에 다달해 전송 클럭을 높이는 방식으로 성능 개선 중임

시리얼 통신은 8비트 시절 만들어져 8비트 단위의 데이터 전송을 함
8비트 병렬 데이터를 시리얼로 변환하고 전송하는데,
페러렐 -> 시리얼 변환 통신 컨트롤러가 UART ( Universal Asynchronous Receiver and Transmitter )

시리얼 통신은 오래되어 거의 모든 UART 컨트롤러는 8bit data width

시리얼 통신
**송신**
- 1바이트 병렬 데이터 -> 보관 레지스터(Holding Register) -> 이동 레지스터(Shift Register)로 옮겨 한 비트씩 출력 핀으로 내보낸다.
**수신**
 - 1비트씩 수신하여 -> 이동 레지스터(Shift Register)에서 1바이트 되면 -> 보관 레지스터(Holding Register)에 옮겨 1바이트 정보 전달

비동기 필수
1. 통신 속도
 - ex. 9600bps ( bit per second ) = 1비트 전송 104마이크로초 유지 
 - 1비트 폭 유지 시간 = 1비트 전송 시간  ---> 104 마이크로초 간 비트 폭 유지 1 or 0
 - 즉, 1비트 폭 유지 시간 = 1 / baud rate
2. 통신 데이터 프레임
 - 아무 데이터를 보내지 않는 상태 = IDLE ( 비트 1로 유지 )
 - 전송 데이터 앞 뒤에 제어 비트를 붙여 비트 나열을 약속 ( 시리얼 데이터 프레임 )
 - 제어비트로 START BIt ( 로직 0 ), 바이트 끝난 후 Parity 비트, STOP Bit(로직 1)
 - STOP BIT이후 다음 데이터가 있으면 START Bit 0 으로 확인

**비트**
1. Data Bit
 - 5비트 ~ 8비트 중 선택, 일반적으로 1바이트(8비트) 사용 - 특정 분야에서 9비트 사용하기도 함
 - START - DATA ( LSB - ~ - MSB ) - ParityBit - STOP Bit
2. Parity Bit
 - 가장 단순한 에러 검출 방식
 - Harmming Distance = 2 ( 에러가 2개의 비트 이상 나면 검출 불가능 )
 - Even Parity, Odd Parity, None Parity 선택 가능
 - 위에 나왔다시피 통신 규칙을 서로 동일하게 정해야 함
 - 짝수 패리티 = Data 각 Bit + 패리티 가 짝수 ex. 1010 0101  일때 패리티는 0 이어야함
 - 홀수 패리티는 반대
 - NONE 패리티는 패리티 비트가 아예 존재하지 않음
 - Mark Parity = 무조건 1
 - Space Parity = 무조건 0

**UART 에러**
1. Parity Error
 - UART 레지스터 세트 중 LSR(Line Status Register)의 Bit2에 에러여부 표기
2. Overrun Error
 - 수신 장비가 어떠한 사유로 데이터 Buffer Memory의 수신된 데이터를 처리하지 못하면, 그로 인한 데이터 손실 발생 (Overwrite 되거나 버려지거나 )
3. Framing Error
 - 수신 데이터 프레임에 STOP 비트 검출 불가 시 발생
 - 일반적으로 각각의 장치가 로컬 클럭을 계산하여 속도를 조절하는데, 당연히 오차가 생길 수 있다. 그러면서 비트가 밀리는 현상 발생함 ( 이를 프레이밍 에러라 함 ) ex. 104us가 오차로 인해 100us를 26번 반복하면 104us 오차가 발생하여 비트가 밀림
 - 해결책으로 STOP Bit를 2비트로 두는 식으로 전송 중간에 IDLE 기간을 두어 타이밍 계산을 리셋 시킨다.
4. Break Signal
 - 일정 시간 이상 로직 0으로 유지되는 상황
 - 전체 비트 11 비트에서 항상 1을 유지할 수 없으나, 이러한 상황이 발생했을 때 이를 브레이크 시그널이라 부름
 - 프레이밍 에러로 인하여 1바이트 정도는 그럴 가능성이 있으나, 2~3바이트가 발생하면 확정적
 - 오류라기보단 고장 표시에 가까움

**비동기 시리얼 통신 시스템**

![image](https://user-images.githubusercontent.com/85000118/136394941-288de1c8-da9e-49fa-ae87-eb288775bb84.png)
데이터 처리부(CPU) + UART + Line Transceiver + Connector
위 처럼 구성 됨
UART : 시리얼 <-> 페러렐  데이터 변환
라인 트랜시버 : UART 출력신호를 케이블 통해 멀리 전송하기 적합한 신호 레벨로 증폭

1. 데이터 처리부
 - MCU가 사용되며, CPU 역할
 - UART Channel당 1개의 Chip Select 신호 및 IRQ(Interrupt Request) 신호가 할당 필요
 - 여러 채널을 운용하는 시스템에선 두개를 위한 디코딩 회로 필요
2. UART 컨트롤러
 - 기본 데이터 변환 ( 시리얼 <-> 페러렐 )
 - 그 외 모뎀 신호 ( DSR, CTS,DCD, RI같은 입력신호로 모뎀 모니터링, DTR, RTS같은 출력 신호를 통해 모뎀을 제어
3. LIne Transceiver
 - UART 출력 신호는 TTL or CMOS 레벨의 신호로 5V나 3.3V정도의 크기로 PCB(Printed Circuit Board) 상 돌아다닐 수 있는 수준의 신호
 - UART 신호를 그대로 보내면 감쇄되어 멀리 전송 불가하여 이를 적합하게 증폭하고 다시 낮은 레벨로 조정해주는 역할
 - Transceiver = Tranmitter + Receiver
 - Line Driver라고도 부름 ( 골프 멀리 날리는 골프채 드라이버 )
 - 아래 RS 신호들은 OSI 물리계층으로 신호 방식 및 크기가 조금씩 다름
![image](https://user-images.githubusercontent.com/85000118/136395050-7c14fe40-2c73-4487-83a6-aa4570b6abeb.png)

4. Connector & Cable
 - 현장 설치 시, 장착, 노이즈 등 다양한 문제 발생 ( 커넥터로 조임 나사를 이용 )
 - 보통 UTP 케이블을 사용하지만, 노이즈가 심한 환경 또는 장거리 전송을 위해 STP(Shielded Twisted Pair)같은 차폐 케이블 사용
 
통신 종류 비교 마인드맵
![image](https://user-images.githubusercontent.com/85000118/136396271-74df84eb-5547-4ddb-ab5a-a4ac5c64a165.png)

**시리얼 통신 표준**
RS232, 422, 485 ( Recommended Standard 권고 표준안 )

1. RS-232
 - MASTER - MASTER 통신 전이중(Full Duplex)
 - 1:1 통신만 가능한 단점
2. RS-422
 - Master - Slave 통신
 - 데이터를 동시에 송수신하면 Data collision 발생함
 - 해결하기 위하여 번갈아가면서 사용권한을 부여함  ( Mutex + Round Robin 같은 느낌 ) - Polling 방식이라 불림
 - 급한 Slave에 대한 권한을 주지 못해 비효율적임
3. RS-458
 - Master - Master - Master 방식 - 모두가 Master
 - 모두가 Master라 데이터 충돌 발생이 필연적
 - 해결 방법은 데이터 충돌 여부를 확인하고 재전송
 - 확인 가능 방법 1 : 전송된 데이터를 버스 상에서 전달 여부 관찰
 - 확인 가능 방법 2 : 수신자로부터 답신 받기


RS-232는 그라운드를 기준으로 신호가 큰가 작은가에 따라 데이터가 전송되는 Unbalanced 통신 방식으로 통달거리 15m가량
( 보통 PC 주변에 바로 시리얼 통신 기기 두는 경우 - 장비 관리 콘솔 많이 사용됨 )
RS-422, RS-485는 그라운드 없이 +, - 신호 2가닥을 가지고 그 신호의 크기 차이에 따라 전송되는 Balanced 방식 노이즈에 강하고 더 멀리 전송 가능 최대 1.2km

그리고 케이블 선택에 따른 비용도 필요

장비가 여러대면 RS-232는 케이블을 각각 사용
RS-422는 4가닥(TXD+,TXD-,RXD+,RXD-)으로 장치마다 중간중간 따주는 데이지 체인(Daisy Chain)으로 연결하여 케이블 중복이 없음  ( 전이중으로 송수신 케이블 각각 2개씩 존재 )
RS-485는 반이중으로 (TRXD+, TRXD-) 2가닥 필요 N*N 통신 가능


**시리얼 네트워크의 구성**
![image](https://user-images.githubusercontent.com/85000118/136401870-328035b8-d4ab-4679-af78-197d8c84b66a.png)

RS-232 = P2P
 - 가장 간단한 방식, 네트워크 온전히 사용 가능하여 고속 전송이 가능 + 유지 보수 쉬움; But, 추가 연결 불가
RS-422, RS-485 = Bus
 - 레이아웃이 간단, 소규모 네트워크에 적합, 새로운 노드 쉽게 연결; But, 케이블 문제 시 복원 시간 및 비용 많이 듬, 속도 느려짐

![image](https://user-images.githubusercontent.com/85000118/136402902-2ed44308-dddb-484c-bc86-9900a2266601.png)

