다른 챕터에서 다루지 않은 주제 위주로 정리

여러 부분에 미묘하고 헷갈리는 부분들 다룸


### **레퍼런스**
C++에서 레퍼런스란 일종의 변수에 대한 alias(별칭)임
=> 레퍼런스를 이용하여 레퍼런스가 가리키는 변수 값에 그대로 반영
(역참조 연산을 자동으로 처리해주는 특수 포인터)

**레퍼런스 변수**
생성하자마자 초기화 해야 함
```c++
int x =3;
int& xRef = x;

xRef = 10;
// x=10; 과 동일

int& yRef;  // 초기화 안하여 컴파일 에러

int& unnamedRef1 = 5; // 컴파일 에러 (리터럴은 사용 불가)
const int& unnamedRef2 = 5; // 정상 ( const라 어차피 수정 못함 )
```
데이터 멤버 초기화 - 생성자 이니셜라이저 이용

위를 이용하여 임시 객체에 대한 const 레퍼런스 만들기 가능
```c++
std::string getString() { return "Hello World!"; }

std::string& string1 = getString(); // 에러
const std::string& string2 = getString(); // 정상
```


**1. 레퍼런스 대상 변경하기**
본래 레퍼런스는 최초에 지정한 변수만 가리킴 ( 변경 불가 )
```c++
int x = 3, y =4;
int& xRef = x;
xRef = y; // y를 가리키는 것이 아닌, y의 값을 적용 => 4
xRef = &y; // 에러 => 포인터에 대한 레퍼런스가 아닌 int에 대한 레퍼런스라서
```


**2. 포인터에 대한 레퍼런스와 레퍼런스에 대한 포인터**
레퍼런스는 모든 타입에 대해 생성 가능
```c++
int* intP;
int*& ptrRef = intP;
ptrRef = new int;
*ptrRef = 5;
```

![image](https://user-images.githubusercontent.com/85000118/139620223-78309791-d53d-41d1-86bf-5e0d15053e7a.png)

xPtr == xRef  
=> 타입이 달라 컴파일 에러 ( 포인터 타입 vs 레퍼런스 타입 )
xPtr == &xRef,     xPtr == &x
=> 정상 (true)

레퍼런스에 대한 레퍼런스는 선언 불가


**레퍼런스 데이터 멤버**
생성자 본문이 아닌, 반드시 이니셜라이저 초기화
```c++
MyClass(int& ref) : mRef(ref) { }
```

**레퍼런스 매개변수**
흔히 C의 포인터 대신 사용하는 케이스 => pass by Reference
```c++
void swap(int& first, int& second);
swap(x,y);
```
위 처럼 인자로 받으면 대상을 가리켜 대상의 값을 직접 수정 가능
리터럴 상수를 쓰려면 const로 사용해야 함 ( 하지만, 해당 함수에서는 의미 없음 )
=> rvalue 레퍼런스 ( 이름 없는 변수나 상수 )

**1.포인터를 레퍼런스로 전달하기**
위 함수에서 포인터 타입을 전달하려면?
=> 역참조를 이용하면 레퍼런스 변환
```c++
int x=5, y=6;
int *xp = &x, *yp = &y;
swap(*xp,*yp);
```

**2.레퍼런스 전달 방식과 값 전달 방식**
레퍼런스 이용하면 값을 복제하지 않아, 장점이 있음
1. 효율성
=> 복제 오버헤드 줄임 - 레퍼런스만 전달
2. 정확성
=> 일부 객체 복제가 제대로 지원되지 않는다면? 문제 있을 수 있음

수정불가로 const 까지 사용하면 수정방지 효과도 있음


**레퍼런스 리턴 값**
리턴 타입도 레퍼런스로 사용 가능
=> 복제 연산 줄임 ( 단, 함수 종료 후에도 계속 남아 있는 객체 )

**rvalue 레퍼런스**
lvalue = rvalue  형태의 구조에서 rvalue는 대표적인 예로 상수, 임시 객체 등이 있음

복습차원에서 다루면,
레퍼런스 string& 를 받는 함수에  리터럴 "Hello"를 보내면 함수를 수행 못함
=> string&&를 받도록 하면 수행 가능함
=> 2개 구현 stirng& 버전도


**레퍼런스와 포인터 선택 기준**
swap 함수는 포인터로도 구현 가능하지만, 코드가 복잡해짐
=> 역참조도 해야하고, nullptr도 걱정해야 함
=> 포인터를 없다는 가정임
```c++
void refcall(int& t ) { ++t; };

int* ptr = (int*)8;
refcall(*ptr);
```
만약 위처럼 포인터를 받아, 사용하면 예측 불가(프로그램 죽을 수도)

객체 레퍼런스는 포인터처럼 다형성도 지원함
그러나, 포인터를 사용해야 하는 경우가 있음
 - 가리키는 위치 변경하기(레퍼런스는 변경 불가)
 - 동적할당 메모리 주소는 포인터
 - 주소 값이 nullptr가 될 수 있는 optional 타입 ( 반드시 포이너 사용 )
 - 컨테이너에 다형성 타입 저장할 때


포인터?? 레퍼런스??
=> 메모리 소유권이 어디에 있는지로 결정
1. 메모리 소유권이 변수를 받는 코드에 있다
=> 메모리 해제 책임은 그 코드에 있음 = 스마트 포인터 사용
2. 메모리 해제할 필요가 없다
=> 레퍼런스로 전달

어지간해서는 레퍼런스 사용 => 꼭 필요한 부분만 포인터

예를들어, 여러 인자를 전달하고 그 결과 값을 받아야 하기 때문에, 관련 레퍼런스 인자 여러개를 전달할 때,  ex. 동적할당 할 포인터, 동적할당 갯수 등
=> C스타일 포인터는 받고나서 역참조 등을 수행하며 더러워짐
=> 레퍼런스는 그냥 전달하고 * 쓸 필요 없이 사용 가능

물론, 동적할당보다 컨테이너 이용하는게 좋음

추가로, 예시는 매개변수를 이용한 값의 반환인데,
=> 가급적 사용하지 말고 리턴문을 이용하는게 좋음

C++11부터 추가된 이동 의미론과 C++17에 도입된 구조적 바인딩을 이용하면 간단하게 구현 가능
=> Pair나 vector 등 여러 목록을 담는 구조를 이용

```c++
pair<vector<int>, vector<int>> Func(const vector<int>& arr);

vector<int> vec = {1,2,3,4,5,6,7,8,9,10};
auto[odds, evens] = Func(vec);
```
위 함수에서 vector에 담긴 홀수와 짝수를 나눠 각각 리턴하는 함수라하면
=> auto[odds, evens] 를 이용하여 return 값을 각각의 vector에 담음


### **키워드 혼동**
const?? static??

**const 키워드**
변경 금지

**1.const 변수와 매개변수**
값이 변하지 않도록 보호 = #define 같은 상수
매개변수들도 const 가능

**const 포인터**
2가지 가능성
 - 가리키는 값이 변하지 않게
```c++
const int* ip;
```
 - 가리키는 대상이 변하지 않게
```c++
int* const ip = nullptr;  // new 할당하거나 변경하려하면 오류
// 선언과 동시 초기화
```

둘 모두는 const를 각각 붙여서 사용
**const 규칙은 오른쪽에서 왼쪽으로**
int* const ip; <-> int const* ip;
int에 대한 포인터에 const 적용 <-> const가 적용된 int에 대한 포인터

**const 레퍼런스**
레퍼런스는 기본적으로 const 속성을 가짐
레퍼런스는 &에 대한 &를 만들 수 없어 한단계( 포인터는 이중 삼중 가능 )

const 레퍼런스는 대부분 매개변수에 적용하며 아래와 같음
```c++
int z;
const int& zRef = z;
zRef = 4; //에러
```

매개변수 대상이 객체면 기본적으로 const 레퍼런스 (변경필요 시, const 생략)

**2.const 메서드**
const 메서드에서는 데이터 멤버 변경 불가 ( mutable 선언한 멤버만 가능 )

**3.constexpr 키워드**
상수표현식이란 개념
![image](https://user-images.githubusercontent.com/85000118/140251528-850e027c-9396-470f-8696-6eafbc19a222.png)

const 대신 constexpr키워드 사용하면 컴파일 시간에 계산되어 적용 가능

constexpr 적용 함수의 제약사항 (부작용-side effect 발생 방지)
![image](https://user-images.githubusercontent.com/85000118/140251708-df85eba6-b8b3-4069-b864-1a43d9712ec2.png)

상수표현식 변수 만들 때 constexpr생성자 정의
똑같이 제약사항이 있음
![image](https://user-images.githubusercontent.com/85000118/140251827-e27dc702-7416-45f4-8503-6cf10384bf4d.png)

클래스에서 다음과 같이 사용이 가능하다.
![image](https://user-images.githubusercontent.com/85000118/140251930-075d675c-1970-46fa-8d86-7e6cbfd9605d.png)

고정 상수 값 전달을 위한 메서드를 컴파일 시간에 활용 가능함


**static 키워드**
다양한 용도 => 키워드 늘어나는 것 피하기 위함

**1.  static 데이터 멤버와 메서드**
객체에 속하지 않는 공용 멤버로 볼 수 있음

**2.static 링크**
C++은 소스파일 단위로 컴파일 => 오브젝트 파일
=> 링크단계에서 서로 연결
각 소스파일은 외부링크나 내부 링크로 연결(외부는 다른 소스파일에서 사용)
함수나 전역변수는 기본적으로 외부링크
=> static을 붙이면 내부링크(정적링크)가 적용됨

ex. 2가지 소스파일이 있고, 한 곳에는 프로토타입만 선언했을 때, 외부링크로 다른 곳에서 구현부를 찾음
=> 구현부 있는 소스파일에서 static을 선언하면 찾지 못하게되어 에러 발생

익명 네임스페이스를 통해 동일하게 내부링크만 적용 가능(이게 더 바람직)
![image](https://user-images.githubusercontent.com/85000118/140252698-7a0727c2-260a-44e6-8f40-ea68f1bab537.png)

**extern 키워드**
static 관련 키워드 => 외부링크 적용 키워드

extern을 붙이면 정의가 아닌 선언문 따로 정의해야함(메모리 할당 안됨)
```c++
extern int x;
int x = 3;
// extern int x = 3;  이렇게도 가능함
```
1개의 소스파일에서 extern int x를 선언 및 초기화 하고,
다른 소스파일에서 extern int x; 를 선언하면 사용이 가능함
=> 다른 파일에서 extern int x;가 아닌, int x를 전역으로 선언하면 에러가 발생함
=> 전역 스코프에 x 변수가 2개이기 때문

전역변수 사용은 좋지 않다. 굳이 사용한다면 주의

**3.함수 안의 static 변수**
scope 안에서만 변수의 값을 유지하는 로컬 변수
=> 되도록 클래스로 만들어 객체내에서 관리하도록 하는게 좋음


**비로컬 변수의 초기화 순서**
여러 비로컬 변수, 전역 변수가 어떤 순서로 초기화 되는지 알 수 없다.
C++표준이 아님, 하지만 전역변수를 사용한다면 중요하다. (다른 전역변수에서 사용하게 될 수도)


### **타입과 캐스팅**

**타입앨리어스**
기존 선언된 타입에 다른 이름 붙이기
```c++
using IntPtr = int*;
int* p1;
IntPtr p2;
```
동일한 것으로 보기 떄문에 완벽하게 호환됨
간결하게 보기 위하여 주로 사용
ex. using namespace std;
ex. using StringVector = std::vector<std::string>;
실제로 string은  using string = basic_string<char> 이다


**함수 포인터에 대한 타입 앨리어스**
함수도 내부적으로 주소가 있음
이를 데이터로 취급이 가능함

![image](https://user-images.githubusercontent.com/85000118/140259670-f4772dd5-ee1b-4d3c-85c5-98688b1cfcf5.png)


![image](https://user-images.githubusercontent.com/85000118/140259684-3d06539f-1013-401c-a188-ec57f97febe3.png)

위처럼 변수처럼 매개변수로 넘겨, 일반 함수처럼 사용이 가능함

실제 함수를 아래 처럼 구현하고,
![image](https://user-images.githubusercontent.com/85000118/140259742-ae365a56-5752-491b-9354-46d9e1e58eb3.png)

![image](https://user-images.githubusercontent.com/85000118/140259758-f9bd88ae-0305-47a1-b2ba-a637d436437e.png)

인자로 함수의 주소값을 넘겨주면 사용이 가능하다. (&를 뺴도 주소라고 판단함)
// 보통은  bool(*funcName)(int,int,int) 로 전달
// using은 이름을 빼고 변수 타입처럼 사용 가능

함수 포인터대신 std::function도 있는데, 나중에 설명

대표적으로 함수포인터 쓰는 경우, DLL
```c++
HMODULE lib = ::LoadLibrary("hardware.dll"); // 커널 함수 loadLibrary
// 출력 결과 = 라이브러리 핸들에 담음
// 함수를 불러오려면 프로토타입을 알아야함
int __stdcall Connect(bool b,int n, const char* p);
// __stdcall은 MS에서 정의한 지시자
// => 함수 매개변수 전달 방식 및 메모리 해제 방법 지정

using ConnectFunction = int(__stdcall*)(bool,int,const char*);
// DLL 정상 로드, 포인터 정의 후
ConnectFunction connect = (ConnectFunction)::GetProcAddress(lib,"Connect");
// 에러면 nullptr 발생
connect(true,3,"HelloWorld");
// 본래라면 (c언어) 앞에 역참조 구문을 적어야 함 => 요즘 컴파일러가 대신 해줌
(*connect)(true,3,"HelloWorld");
```

**메서드와 데이터 멤버를 가리키는 포인터에 대한 타입 앨리어스**
메서드 내의 함수를 함수포인터로 호출하기
![image](https://user-images.githubusercontent.com/85000118/140262052-35b5bb9d-ac8e-4e1a-a016-69ba89e6b4ee.png)

위를 타입 앨리어스를 활용하여
![image](https://user-images.githubusercontent.com/85000118/140262406-ceffdad6-c231-4beb-9207-f8a2bf435072.png)

이렇게 작성이 가능함

auto까지 활용하면 더욱 간결하게 표현이 가능하다.
![image](https://user-images.githubusercontent.com/85000118/140262441-5f1720f1-4346-4c8d-9adf-0cdc7452f973.png)

std::men_fn()을 이용하면 (.*)같은 문법 사용 필요가 없음 => 추후 설명

non-static 멤버는 객체를 거쳐야 역참조 할 수 있음을 명심


**typedef**
타입 앨리어스는 C++11부터,
그전에는 typedef로 구현
```c++
typedef int* IntPtr;
```

![image](https://user-images.githubusercontent.com/85000118/140262830-c094cb8f-2f44-45ca-b4ce-98868aefa36f.png)

typedef보다는 타입 앨리어스 사용!


**캐스팅**
C++ 4가지의 캐스팅 방법이 있음 ( 기존의 괄호 캐스팅도 계속 지원 )

**1.const_cast()**
가장 이해하기 쉬움
변수에 const 속성을 추가하거나 제거할 때 사용
=> 보통은 그럴일이 없어야지만, 일시적이거나 부득이하게 사용

![image](https://user-images.githubusercontent.com/85000118/140263186-81f9107e-bc4d-4ca7-bd5a-45bd5f25baff.png)

C++17부터는 std::as_const() 헬퍼 메서드 추가됨 ( <utility> 헤더 )
=> 레퍼런스 매개변수를 const 레퍼런스 버전으로 변환
=> as_const(obj)  = const_cast<const T&>(obj)와 같음    (T는 타입)

non-const => const에는 as_const가 간결
```c++
std::string str = "C++";
const std::string& constStr = std::as_const(str);
auto result = std::as_const(str);
// auto는 const를 지워버림 ( 주의 )
```

**2.static_cast()**
명시적 변환 기능
예를 들어 정수를 이용한 나눗셈
```c++
int i = 3;
int j = 4;
double result = static_cast<double>(i) / j;
// 둘 중 하나만 적용해도 C++은 부동소수점 나눗셈적용
```

클래스 간에도 사용 가능
ex. A 생성자의 인자로 B 객체를 받을 때, static_cast로 처리

![image](https://user-images.githubusercontent.com/85000118/140265444-ddc4ae11-6c8f-43a3-9cd2-00242a3d0f18.png)
![image](https://user-images.githubusercontent.com/85000118/140265467-2815e2a8-4424-4f9b-8195-ebd4881de875.png)

-----여기서부턴 적당적당히 작성--------- 531p

reinterpret_cast()
=> 거의 모든 형변환을 억지로 시킴
dynamic_cast()
=> 보통 상속관계에 사용, 포인터 오류는 nullptr, 레퍼런스 오류는 bad_cast익셉션

![image](https://user-images.githubusercontent.com/85000118/140269496-06bd1f6c-4d4f-4c3b-b144-261b12976a9b.png)

### 스코프
namespace, if의 이니셜라이저, 전역변수 등
익명 네임스페이스는 전역수준으로 처리됨

### 어트리뷰트
C++ 표준 추가 전에는 __atribute__.__declspec같은 속성이 있었음
표준은 6개 지원
 - [[carries_dependency]]
=> 설명 스킵
 - [[noreturn]]
=> 함수 앞에 [[noreturn]] void ~~ 쓰면 리턴 안함
=> 보통 프로세스나 스레드 종료에 사용
 - [[deprecated]]
=> 지원하지 않는 대상 지정  ( 컴파일 에러 또는 경고 메시지 출력 )
![image](https://user-images.githubusercontent.com/85000118/140269809-eec933c9-7d09-40ff-89d5-454c605a1d59.png)

 - [[fallthrough]]
=> C++17부터 추가됨
=> case문에서 의도적으로 아래 case문까지 break없이 진입하려 할 때 사용

 - [[nodiscard]]
=> 리턴 함수 앞에 적으면, 리턴 값을 사용하지 않는 호출 발생 시, 경고 출력
=> 보통 에러 코드를 무시하지 못하도록 에러 함수에 적용

 - [[maybe_unused]]
=> C++17에서 추가됨
=> 사용하지 않는 코드를 발견해도 경고 메시지 출력하지 말라
=> 매개변수 등에 적용 가능

벤더마다 충돌할 수 있어서, [[clang::noduplicate]] 처럼 벤더 이름 붙이는 게 좋음


### 사용자 정의 리터럴
C++ 표준 리터럴이 있음
 - 'a' : 문자
 - 'character array" : 0으로 끝나는 문자 배열
 - 3.14f : 부동소수점(float)
 - 0xabc : 16진수

이런 것 말고 직접 정의 ( 반드시 _ 언더스코어로 시작해야 함, 첫 문자 반드시 소문자 )
리터럴 연산자 정의는 미가공 모드(raw mode) or 가공 모드(cooked mode)

미가공 모드에서는 단순히 문자 나열된 것
가공 모드는 특정 타입의 값

가공모드의 조건
![image](https://user-images.githubusercontent.com/85000118/140271793-63e5fb53-314e-4635-b7e7-06953068d1af.png)

![image](https://user-images.githubusercontent.com/85000118/140271880-1fbb4e8a-b450-43f5-97f4-8aae5e35f658.png)

![image](https://user-images.githubusercontent.com/85000118/140271976-708885b8-6399-475e-ac52-c738e6883ecd.png)

**표준 사용자 정의 리터럴**
표준은 언더스코어로 시작하지 않음
![image](https://user-images.githubusercontent.com/85000118/140272066-0fbb8f02-6909-4bb6-bf7f-14dc2b3ff5a9.png)
![image](https://user-images.githubusercontent.com/85000118/140272113-57b40b50-5454-4762-b39a-3d5ee73c5d6a.png)


### **헤더 파일**
