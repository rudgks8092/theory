**윈도우 특징**
1. GUI
2. 메시지 구동 구조 ( 각 응용은 독립적인 메시지 큐를 갖고, 변화를 감지하여 운영체제가 각각의 메시지 큐에 전달 함 )
3. 멀티태스킹

**윈도우 응용 특징**
1. API 호출문 집합 ( 유닉스 계열은 System Call 이라고 함 )
2. 메시지 핸들러 집합 ( 메시지에 따라 결과가 달라지는 메시지 구동 구조 - 핸들러 기반 ) - 메시지 핸들러 집합을 윈도우 프로시저라 부름
3. 실행파일 + DLL 집합
4. 장치 독립성 ( 직접 외부 장치들을 다루지 않고, API를 통해서 제어함 )

**윈도우 응용 개발 방식**
1. SDK ( WIndows 10 SDK같은 개발 도구 이용 - API를 직접 다뤄 세부 제어와 모든 기능 이용이 가능하지만, 생산성이 낮아짐 )
2. RAD (Rapid Application Development)
 - VB나 델파이 등을 이용함 - 빠른 개발이 가능하지만, 클래스나 SDK에 비해 속도도 느리고 기능에 제한이 걸림
3. 클래스 라이브러리
 - C++을 이용해 윈도우즈 기능들을 클래스화 한 것들 MFC나 OWL
 - RAD와 SDK 중간급, 속도도 빠르고 많은 기능 이용이 가능하며 상대적으로 편리하게 개발 가능
4. .NET 프레임 워크
 - 윈도우에서 설치 가능한 소프트웨어 개발 및 실행 환경
 - 공용언어런타임( Common Language Runtime) 가상머신을 사용하여 **장치독립성**
 - 윈도우 API에 버금가는 방대한 라이브러리 ( 언어에 상관없이 라이브러리 사용 가능 ) **언어 독립성**
 - MS의 차세대 소프트웨어 개발 및 실행 플랫폼으로 볼 수 있다.

SDK = C언어
MFC = C++ ( SDK 활용하여 객체지향처리 )

2개를 합쳐서 필요한 기능을 사용하는 편

