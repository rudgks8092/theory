### **C++ 고급 기능**

C++의 String  <string> 헤더 사용 - 동적으로 이용 가능한 문자열

### **Stack과 Heap**

![image](https://user-images.githubusercontent.com/85000118/136684939-56054391-2945-46a3-a264-5a984f02f99a.png)

위 그림처럼 서로 반대 방향으로 메모리가 자라게 된다.
stack은 보통 지역 변수로 main에서 시작하여 지역변수나 함수 호출에 따라 Stack Frame이란 형태로 쌓음 ( Return되면 최근 Stack Frame을 제거하는 식 )
스택은 직접 deallocate 할 필요 없이 자동으로 처리 됨
Heap은 동적할당을 사용하면 생성되는데, 스마트포인터를 사용한 것이 아니라면 직접 해제 해야 한다. ( 비트 더미에서 원하면 직접 할당하고 사용 하는 느낌 )

### **포인터**
int* a; 형태의 선언
위 포인터변수는 초기화되지 않은 변수라 한다. (uninitialized variable)
포인터 변수를 선언하고 반드시 초기화 해야함
어떤 메모리를 가르키는지 모르고 사용하면, Crash로 프로그램이 뻗어버릴 확률이 매우 높음
그래서 선언하자마자 초기화하거나 Null pointer 사용 ( nullptr )
int* a = nullptr;

포인터에 할당된 주소가 아닌 가리키는 값을 접근 하기 위해서는
**역참조(dereference)로 int b = *a; 형태로 사용한다.
( 역참조는 초기화나 메모리 할당 후에만 사용 필요 )

new로 할당한 메모리는 반드시 해당하는 주소값을 delete a; 해야하고,
a = nullptr 널 포인터로 초기화 해주는 것이 좋다.

new 로 할당하는 것이 아닌 기존 전역 또는 지역 변수의 할당은
int i = 3;
int* j = &i;
형태로 한다. ( & 는 주소 참조 연산자로 해당 값에 대한 주소를 알려줌 )

구조체 포인터도 같은 방식
struct node* a = nullptr;
struct node b;
a = &b;
(*a).a  같은 형태로 가져올 수 있음 ( 포인터로 사용할 때는 a->a;


**단락 논리를 이용하면 잘못된 포인터 사용 방지 가능하다.**
```c++
bool isValidSalary = (anEmployee && anEmployee->salafy > 0);
or
bool isValidSalary = (anEmployee != nullptr && anEmployee->salafy > 0);
```

### **동적 할당**
```c++
int* arr = new int[8];
```
주소 값을 가리키는 포인터 변수에
new 연산자를 이용하여 Heap영역에 int형 배열 8사이즈만큼 할당함
할당한 배열은 동일하게 arr[7] 까지 접근이 가능하다.

할당받은 변수를 delete a or delete[] arr 로 메모리 해제 필수

**C는 malloc + free, C++은 new delete 쌍으로 사용**

**malloc, new 차이**
1. 함수 <-> 연산자 ( 라이브러리 필요 유무 )
2. malloc은 사이즈를 매개변수로 형변환 필수(void* 반환), new는 그냥 해당 객체 크기만큼 생성 반환
3. new는 Operator로 Overload 가능, 재정의 가능
4. 생성 실패 시, new는 throw, malloc은 nullptr
5. new는 생성자를 자동 호출해줌
등등 자세한 것은 검색 필


### **널 포인터 상수**
C++11 이전에는 nullptr이 아닌 NULL 이나 0을 사용했음
```c++
void func(char* str) { cout << 1; }
void func(int i) { cout << 2; }
int main()
{
	func(nullptr);
	func(NULL);
	return 0;
}
```
과거에는 동일한 느낌이었지만, 위 같은 경우 위는 1, 아래는 2가 출력 된다.


### **스마트 포인터**

C의 기본 포인터 대신 C++에서는 Smart Pointer를 사용한다.
스마트 포인터는 그 대상이 스코프를 벗어나면 ( 사용 중인 범위 - 함수 실행 끝 ) 자동으로 해제 됨

<memory> 헤더를 사용, std 네임스페이스 사용
 1. unique_ptr
 2. shared_ptr
 3. auto_ptr

현재 auto_ptr는 C++11부터 사용중지 권고, 17부터 삭제됨
이유는
```c++
~auto_ptr() {
    delete _Myptr;
}
```
auto_ptr에 대한 소멸자가 위와 같이 구현되어 있는데,
이는 배열에 대한 메모리 해제를 진행하지 않아서, unique_ptr로 대체하였다.

스마트포인터는 별도로 리소스 해제를 신경쓰지 않아도 되는 장점이 있다.
exception이나 return되어도 스스로 리소스를 해제한다.

```c++
// C++14 이전 방식 
unique_ptr<int> a(new int);
unique_ptr<int> a(new int[10]);
// C++14부터 추가된 make_unique<>(); 
unique_ptr<int> a = make_unique<int>();
unique_ptr<int[]> a = make_unique<int[]>(10);

// 간편하게 앞의 unique_ptr는 auto로 대체함
auto a = make_unique<int[]>(10);
```

**shared_ptr는 데이터 공유 목적의 스마트 포인터**

shared_ptr는 레퍼런스 카운터 (참조 횟수)가 있으며, 그 카운트가 0이 되면 해제한다.
동일하게 make_shared를 이용

원래는 배열 저장이 불가능하였지만, C++17부터 배열 저장 가능, 다만 make_shared<>() 불가
```c++
shared_ptr<int> a = make_shared<int>();
shared_ptr<int[]> a(new int[10]);
```


**1. 소유권 관련 없다 = 일반 포인터**
**2. 소유권이 유일 = unique_ptr**
**3. 소유권 공유 필요 = shared_ptr**

### **const**
1. const 상수
 - 일반적인 사용법 = 수정되지 않을 상수 선언에 사용 ex. MAX_COUNT
 - #define은 전처리기가 메타 수준 처리하여, 전처리기가 처리하므로 꼭 필요한게 아니라면 const를 쓰는게 좋다.
2. const 매개변수
 - 함수에 전달 할 떄 컴파일러를 이용한 변수 원본 보장용도
 - 변수를 함수에 전달 할 때, 함수에서 const int 이런식으로 받으면 함수 자체에서 원본을 보장해준다.

### **레퍼런스**

**함수의 매개변수 전달 방식**
1. Pass by value
 - 함수에 값만 복제하여 전달하는 방식
 - 직접적인 값 수정을 위해서 포인터 사용하여 복잡하게 수정하였었음
2. Pass by Reference
 - 함수에 참조 전달 방식
 - 복제하지 않고, 그 변수 그대로 전달하기 때문에 pass by value보다 조금 더 효율적
 - C++11 추가된 이동 의미론(Move Semantics) 덕에 복제하지 않고 구조체나 클래스를 직접 리턴 가능해짐 ( 추후 설명 )
3. Pass by const Reference
 - 정식으로 사용되는 명칭이 아닌 그냥 표현
 - 모순되지만, 성능 개선을 위함
 - const int& a 처럼 함수에서 참조 값을 받지만, 원본을 보장해주고 값을 복제하지 않고 사용하기에 성능을 조금이나마 개선할 수 있음
 
###**Exception**
C++은 굉장히 유연하지만 안정성은 낮은 편임 ( 메모리 공간 제어 )
안정성을 높이기 위한 방법 중 하나 Exception 클래스/객체
<strexcept> 헤더, std::ivalid_argument  -> 여러 exception 중 하나

```c++
double divideNumbers(double numberator, double denominator)
{
	if (denominator == 0) throw invalid_argument("Denominator cannot be 0.");
	
	return numberator / denominator;
}

int main()
{
	try { cout << divideNumbers(2.3, 0) << endl; }
	catch (const invalid_argument& exception) {
		cout << "Exception : " << exception.what() << endl;
	}
	return 0;
}
```
try 내부에서 오류가 발생하여 throw 하면 중단하고, catch로 exception을 잡아 예외처리 할 수 있음
std:invalid_argument 외 여러 상황에 맞는 익셉션 타입 사용

### **타입 추론(type inference)**
컴파일러가 스스로 타입을 알아내는 기능

1. auto
 - 함수 리턴 타입
 - 구조적 바인딩
 - 표현식 타입 추론
 - 탬플릿 매개변수 타입
 - 제네릭 람다
 - 다양한 상황에 사용
 - 핵심은 auto를 사용하였을 때, 리턴 타입이나 매개변수를 변경하면 일일히 수정해줘야 하는 번거로움을 없앨 수 있음
 - 단점 : const나 레퍼런스가 제거됨
```c++
// const string&을 리턴하는 함수 가정 foo();
auto f1 = foo();
// 를 하면 f1은 const string이 아닌, string 값이 그냥 복제 됨
const auto& f2 = foo();
// 를 해야 정상 동작
```
2. decltype
 - 특정 변수의 타입을 추론하여 타입을 지정
```c++
int x = 123;
decltype(x) y = 456;

decltype(foo()) f2 = foo(); 
// 위와 같은 사용이 가능하며, const와 &를 유지한다.
```

### **객체 지향**
Object-oriented Programming OOP  - 추후 설명

### **Class**
객체의 특성을 정의
 - C++은 주로 선언부를 헤더파일에, 구현부를 소스파일에 작성함
 - 데이터 멤버 + 메서드 작성
 - 접근 수준 지정 ( public, protected, private )
 - 보통 데이터 멤버는 public 함수 getter나 setter로 정의하거나 사용하는 구현을 많이 함
 - 멤버변수(데이터 멤버) 앞에는 m을 붙임 ex. mPassName 
 - 멤버함수는 멤버 변수를 변경할 필요가 없다면 const 사용 지향
 - const 지정함수를 inspector or accessor or getter 라고 함
 - non-const 멤버 함수는 mutator - 변경자라 부름
 -  자세한건 추후 작성

```c++
class AirlineTicket {
public:
	AirlineTicket();
	~AirlineTicket();

private:
// 데이터 멤버를 고정 값으로 초기화하면 여기서 작업해도 됨
	string mPassengerName;
	int mNumberOfMiles;
	bool mHasEliteSuperRewardsStatus;
};

// 생성자 - 뒤의 : 는 생성자 이니셜라이저 ctor이니셜라이저(생성 시 초기화)
AirlineTicket::AirlineTicket(): mPassengerName("ABC"), mNumberOfMiles(0), mHasEliteSuperRewardsStatus(false) {
	// 이부분에 작성하면 생성 후, 초기화
    // ctor 이니셜라이저를 권장함
}
//소멸자
AirlineTicket::~AirlineTicket() {

}
```


### **유니폼초기화**
C++11 이전에는 타입 초기화 방식이 일정하지 않았음
```c++
struct A = { 1 , 2 , 3};
class A(1,2,3);
```
구조체는 {...} 문법
클래스는 (...) 함수 표기법

C++11부터는 타입 초기화에 {...} 문법을 따르도록 통일됨 - 유니폼 초기화 (uniform initialization) = 균일초기화, 중괄호 초기화

```c++
class a = {1,2,3};
// 등호 생략도 가능
struct A{1,2,3};
class A{1,2,3};

int a{};  // 0이 됨 제로초기화


// narrowing (축소 변환 - 좁히기)를 방지할 수 있음 
int x{3.14};
func({3.14});
// 컴파일 오류 발생

// 동적할당 배열 초기화 가능
int* pArray = new int[4]{1,2,3,4};

// 배열 ctor initialize ( 생성자 이니셜라이저 ) 도 가능
mClass() : mArray{0,1,2,3,4} { }
```

### **리스트 초기화**
**1.  복제 리스트 초기화 (copy list initialize) : T obj = {arg1, arg2, ...};**
**2. 직접 리스트 초기화 (direct list initialize) : T obj{arg1,arg2,...};**

c++17부터 auto의 리스트 초기화가 바뀜
```c++
// 기존 복제 리스트 초기화
auto a = {1};
auto b = {11, 22};
// 기존 직접 리스트 초기화
auto c{1};
auto d{11,22};

// 모두 initializer_list<int> 로 선언됨

// C++17 복제 리스트 초기화
// 기존과 동일
auto a = {11};
auto c = {11,22};  

// 직접 리스트 초기화
auto c{1};  // int
auto d{11,22}; // 원소가 너무 많아 에러

// auto가 직접 리스트 초기화 값 1개에 대해서만 추론

// 복제 리스트는 중괄호 내 모든 타입 같아야 함
auto b = {11, 22.33 } // 에러
```
