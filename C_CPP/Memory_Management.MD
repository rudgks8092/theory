스마트포인터나 vector 같은 자동 메모리 관리 도구들을 이용하여도, 내부 처리 과정을 알아둘 필요가 있음

### **동적메모리 다루기**

로컬에 선언하는 변수 int i = 7;
=> 자동 변수(automatic variable)로 스택 메모리 이용 스코프 벗어나면 자동 해제

new 키워드 이용
```c++
int *ptr = new int;
// 할당 안하면 nullptr로 초기화 필수
```
=> ptr 변수 스택 생성하고, ptr는 힙에 동적 생성하고 가리킴
![image](https://user-images.githubusercontent.com/85000118/137139797-eb18435c-0814-4706-aa76-d0881157e551.png)

포인터 변수가 스택에 있으려면,
```c++
int** handle = nullptr;
handle = new int*;
*handle = new int;
```
![image](https://user-images.githubusercontent.com/85000118/137140255-ee6f3ded-decf-451f-bbc4-a84ce762f533.png)


**메모리 할당과 해제**
new 키워드로 생성, delete 키워드로 해제

**1. new & delete 사용법**
new에 타입 지정하여 호출하면, 메모리 할당 후 그 메모리 주소를 반환함(포인터)
=> 이 리턴 값을 잃어버리면 접근이 불가능하고, 이를 메모리 누수(Memory Leak)이라 함
=> 메모리 누수가 발생하면 그만큼의 메모리가 할당된 채로 사용이 불가하여 꼭 해제 필요

```c++
int* ptr = new int;
delete ptr;
ptr = nullptr;
// 반드시 위 루틴이 필요함
```

**2. malloc()**
malloc은 인수로 지정한 바이트 수만큼 메모리에 할당하는 함수
=> new는 메모리 할당뿐만 아니라 객체까지 만들어 바람직 ( 생성자 )
=> free도 delete와는 다르게 소멸자를 호출하지 않는다.
= 메모리 할당만 받고, 사용만 할 뿐이다.

**3. 메모리 할당에 실패한 경우**
메모리가 부족하면 new가 실패하기도 함
 - 기본적으로 new가 실패하면 exception 발생- 프로그램이 종료됨 (추후 해결법 설명)

exception 발생하지 않는 new도 있음 (대신 nullptr 리턴)
```c++
int * ptr = new(nothrow) int;
```
=> 종료는 안되더라도 상황을 처리해줘야 함
=> 표준 new보다 버그 발생률이 높음

### **배열**

**기본 타입 배열**
메모리 할당하면, 연속된 공간에 할당 ( 한 원소 담는 크기 * 배열 크기 )
크기는 컴파일 시간에 결정되어 상수값으로 지정 (드물게 컴파일러가 가변을 지원하기도 하는데 비표준)

배열 포인터
```c++
int* myArrayPtr = new int[5];

delete[] myArrayPtr;
myArrayPtr = nullptr;
```
![image](https://user-images.githubusercontent.com/85000118/137145236-6e2d7e8e-1b99-44ce-8149-e223da62eea9.png)

동적으로 할당된 배열(dynamically allocated Array) != 동적 배열(dynamic array)
* STL 컨테이너처럼 크기 동적 조절 뿐만 아니라 실제 크기도 알아낼 수 있는 데이터 구조도 있음

C로부터 받은 realloc이란 함수도 있음 (새로 지정한 크기에 맞게 메모리 블록을 재할당)
=> 기존 데이터를 새 블록에 복사, 원래 데이터 삭제
=> C++에서는 위험함 =사용자 정의 객체는 비트단위 복사 작업에 맞지 않음
* C++에서 절대 사용 하지 말 것

**객체 배열**
기본 타입과 비슷, new 객체[N]을 하면 제로인수 디폴트 생성자가 호출 됨

**배열 삭제하기**
new[]를 사용하면 delete[]를 사용하여 해제 ( delete는 소멸자도 호출해줌 )
delete[]가 아닌 delete하면 배열의 포인터를 삭제할 수도, 맨 처음 객체만 삭제할 수도 있음
=> 메모리 손상(memory corruption) 발생 - 원소 접근 불가

포인터 배열에 대해서는 각각 해제해줘야 함
```c++
const size_t size = 4;
int** myPtrArray = new int*[size];

//포인터마다 객체 할당
for (size_t i = 0; i < size; i++) {
myPtrArray[i] = new int();
}

//객체 삭제
for (size_t i = 0; i < size; i++) {
delete myPtrArray[i];
}

delete[] myPtrArray;
myPtrArray = nullptr;
```


### **다차원 배열**
1차원 배열을 확장한 것
char board[y][x]   x,y 좌표 평면 기준 왼쪽 코드와 같다.

**다차원 스택 배열**
![image](https://user-images.githubusercontent.com/85000118/137251209-6de0d96e-7e9a-4f62-9a35-35ee948c5e86.png)
=> 너무 기초 개념이라 설명 SKIP ( 검색 )

**다차원 힙 배열**
동적 할당을 이용한 다차원 배열
```c++
char** board = new char[i][j]; 
// 스택 배열 생성과 동일하게 하면 컴파일 에러가 발생한다.
```

![image](https://user-images.githubusercontent.com/85000118/137251491-b22ebc6a-6379-4a85-8e44-1afe4eb018c9.png)

그림과 같이, 2차원 포인터에 1차원 배열을 먼저 할당하고, 각각의 포인터 인덱스들에 다시 y만큼의 동적 할당을 수행해야 함
=> delete 또한 각각 생성한 포인터에서 delete[]를 다수 수행해야 한다.


C Style 동적배열은 안전성이 떨어져 가급적 사용 금지
N차원 동적배열이 필요하면 C++ 헬퍼 클래스인
=> Matrix<T> or Table<T> 클래스 템플릿 이용 (내부적으로 vector<vector<T>> 사용)

### **포인터 다루기**
포인터는 메모리 주소를 의미하는 숫자
정상적이지 않은 다양한 방식으로 이용이 가능
ex. char* scaryPointer = (char*)7; 

**포인터의 작동 방식**
![image](https://user-images.githubusercontent.com/85000118/137253397-c1bf3297-3447-45c1-9472-985d1b3833c0.png)

[*] 연산자 = 역참조로 메모리 주소에 해당하는 내용물 확인 가능
[&] 연산자 = 특정 지점에 대한 주소 확인 가능

**포인터에 대한 타입 캐스팅**
포인터 = 메모리 주소(를 가리키는)
=> 타입을 엄격하게 따지지 않음  (ex. XML문서를 가리키는 포인터 = 정수 포인터
=> C스타일 캐스팅으로 마음대로 변환 가능
```c++
Document* documentPtr = getDocument();
char* myCharPtr = (char*)documentPtr;

// 정적 캐스팅(static cast) 이용하면 조금 더 안전
// 관련없는 데이터 타입으로 포인터를 캐스팅하면 컴파일 에러
Document* documentPtr = getDocument();
char* myCharPtr = static_cast<char*>(documentPtr); // 컴파일 오류
```

정적 캐스팅은 객체가 상속 관계라면 오류가 발생하지 않는다.
하지만, 상속 관계는 동적 캐스팅이 더 안전하다 ( 추후 설명 )

### **배열과 포인터의 두 얼굴**
배열과 포인터는 비슷하다. 자세하게 알아보자

**배열 = 포인터**
힙에만 포인터를 쓰는게 아닌, 스택 배열 접근에도 사용함

**배열 주소는 사실 첫번째 원소의 주소**
```c++
int myIntArray[10];
int* myIntPtr = myIntArray;

// 똑같이 접근 가능
myIntPtr[5] = 5;
```

함수의 인자로 int* arr 형태로 받을 때,
스택 배열도 힙 배열도 모두 가능하다.
![image](https://user-images.githubusercontent.com/85000118/137264700-58103996-f2bb-491f-adaf-b3c932cbc3fd.png)

![image](https://user-images.githubusercontent.com/85000118/137264660-7a8ee83c-64b4-4525-ab5a-e332597e5719.png)

포인터(배열)을 전달하면 복사본이 아닌 원본을 직접 수정
=> 레퍼런스(참조) 전달 방식의 효과 
=> 전달 값이 원본 배열의 주소이기 때문 (포인터 or 배열 0번인덱스 주소)

```c++
void doubleInts(int theArray[], size_t size);
```
위 형태의 배열로 받아도, 원본의 값을 변경한다.

컴파일러는 배열 뒤 [] 안의 숫자를 무시하여 아래의 코드는 모두 동일하다.
![image](https://user-images.githubusercontent.com/85000118/137265400-10909b03-9867-4259-88d2-e0b7d4a95edd.png)

### **템플릿 공부하고 확인**
![image](https://user-images.githubusercontent.com/85000118/137269582-ffb778a3-c411-4a03-8f17-c17901536869.png)

**포인터가 모두 배열은 아니다!**
객체를 가리킬 수도, 하나의 타입만 가리킬 수 도 있다.
=> 모든 배열은 포인터로 참조 가능 - but, 모든 포인터 != 배열


### **로우레벨 메모리 연산**
C++은 메모리 관리를 클래스 생성자 소멸자를 통해 내부적으로 관리하고, 컴파일러가 도와준다.
=> 그러나 저수준 관리 테크닉을 알아두면 좋음

**포인터 연산**
포인터가 정수라서 가능한 연산
ex. ptrArray[2] 나 *(ptrArray+2) 같은 방식으로 값 접근이 가능함

추후에 설명할 wide string
wchar_t 타입 => 유니코드 문자 지원 문자 타입 보통 char보다 크다.
```c++
const wchar_t* myString = L"Hello, World";  
// L을 통해 wide string 명시해줌
```
위의 wide string 전달받아, 특정 위치부터 대문자로 변경한다고 할 때,
=> myString+7 을 인자로 넘겨주면 L"World"로 시작한다.


**커스텀 메모리 관리**
C++ 기본 메모리 할당만으로 대부분의 일 처리 가능함
new와 delete는 내부적으로 적절하게 처리, 관리함

리소스가 상당히 부족할 때나 세심한 메모리 관리가 필요할 때
=> 메모리를 크게 할당하고 필요할 때 잘라쓰는 방식
=> 객체의 수가 많을 때 오버헤드를 줄이기에 유리함
구체적인 방법은 추후에 설명

### **가비지 컬렉션**
메모리를 정상 상태로 유지하기 위한 최후의 보루 Garbage Collection
=> 가비지 컬렉션 제공 환경은 프로그래머가 직접 메모리 해제할 일이 거의 없음
=> 레퍼런스 카운터라 하여 참조하는 수가 0이면 일정 시점 자동 해제

C++은 자바나 C#과 달리 가비지 컬렉션 제공 안됨
=> 최근에는 그나마 스마트 포인터로 조금 나아짐
=> shared_ptr가 가비지 컬렉션과 상당히 비슷한 방식의 관리법
 = 어떤 리소스 참조하든 shared_ptr가 삭제되면 일정 시간 안 가리키던 리소스도 제거

가비지 컬렉션 구현 기법 **표시 후 쓸기(Mark and Sweep)** 알고리즘
= 가비지 컬렉터가 프로그램의 모든 포인터 주기적으로 검사하고, 계속 사용 여부 표시, 표시되지 않은 메모리는 해제

C++에서 구현한다면,
1. 모든 포인터를 쉽게 탐색하도록 포인터를 가비지 컬렉터에 리스트 형태로 등록
2. 가비지 컬렉터가 상태 표시할 수 있도록 모든 객체가 GarbageCollectible같은 믹스인 클래스를 상속
3. 객체에 동시 접근 못하도록 가비지 컬렉터 동작 중 포인터 변경 불가

구현할 수는 있으나 불편하고, 모든 애플리케이션에 적합하지 않음 + 오류가 많이 날 수도

**GC 단점**
1. 가비지 컬렉터 동작 중 프로그램 중지 가능성
2. 소멸자가 비결정적(non-dterministically)으로 호출됨 = 객체는 가비지 컬렉터가 처리해야해서 스코프를 벗어나도 소멸자가 즉시 실행되지 않음
=> 즉, 소멸자가 처리해야 할 리소스 정리작업이 언제 수행될지 알 수 없음

가비지 컬렉션 메커니즘 구현은 상당히 어려움
=> 다양한 에러, 느린 속도 가능성 등

**객체 풀**
가비지 컬렉션?  => 뷔페식당 다 쓴 접시 테이블 위에 두고 치워주는 느낌

좀 더 효율적인 방법 = 객체 풀
=> 접시를 재사용하는 느낌

사용할 객체 수 미리 지정 => 사용 끝난 객체를 다시 사용
= 객체를 자주 생성하는 상황에 오버헤드를 줄이기 좋음 (추후 자세히 설명)

### **스마트 포인터**
delete를 호출하지 못하는 실수 방지를 위함
 - 여러번 해제하면 메모리 손상 or 치명적인 런타임 오류
 - 해제하지 않으면 메모리 누수 현상

**스코프를 벗어나면 동적으로 할당된 리소스 자동 해제**

C++의 스마트 포인터
1. 템플릿을 이용하면 모든 포인터 타입에 대해 타입 세이프 포인터 클래스 작성 가능
2. 연산자 오버로딩을 이용하여 스마트 포인터 객체를 일반 포인터처럼 활용 가능
두 가지 정보 관련하여 모두 추후 보충 설명

리소스에 대한 고유 소유권을 갖는다 = std::unique_ptr

포인터 복사본을 여러 객체가 가지고 있는 상황 = 앨리어싱(aliasing)
=> 마지막 포인터가 해제해야 함
=> 리소스 소유자 추적하는 레퍼러스 카운팅(Reference Counting) - 참조 횟수 계산을 구현 = shared_ptr

복사한 스마트포인터 인스턴스가 스코프 벗어나면 레퍼런스 카운트 감소
=> 레퍼런스 카운트가 0이되면 스마트 포인터에 의해 해제
**공유 소유권 방식(Shared Ownership)**
**shared_ptr는 스레드에 안전함, but 리소스도 안전한건 아님**
**<memory>** 헤더 사용

_리소스 할당 결과를 절대 일반 포인터로 표현하면 안됨_
=> unique, shared나 RAII(Resource Acquisition Is Initialization 리소스 획득 = 초기화) 클래스 사용해야 함 (RAII는 리소스 소유권을 받아 적절한 시점에 해제 - 디자인 기법 중 하나)


### **unique_ptr**
메모리 누수 발생 상황
 - new 하고 delete를 안함
 - new하고 함수 실행 -> delete인데, exception이 발생하여 delete 수행 안됨

**생성 방법**
```c++
auto mySimpleSmartPtr = make_unique<Simple>();
// C++14의 make_unique와 auto 동시 적용
// 생성자가 매개변수를 받는 경우,
auto mySimpleSmartPtr = make_unique<Simple>(a,b);

// make_unique를 지원하지 않는 경우
unique_ptr<Simple> mySimpleSmartPtr(new Simple());


// C++17 이전에는 안전을 위해 make_unique 사용했었음 
foo(unique_ptr<Simple>(new simple()), unique_ptr<Bar>(new Bar(data())));
// 이 경우,  Simple이나 Bar의 생성자 data()함수에서 
// 익셉션이 발생하면 메모리 누수 발생 가능성 높음
foo(make_unique<Simple>(), make_unique<Bar>(data()));

// C++17부터는 앞에 나온 두 코드 중 어느 방식으로 작성해도 안전
// 가독성 감안하면 make_unique가 낫다.
```

**unique_ptr 사용 방법**
스마트 포인터는  일반 포인터와 동일하게 * 또는 -> 연산자를 통해 역참조 함
객체에 대한 함수 호출이나 참조는 동일

단, 함수의 인자가 해당 객체의 일반 포인터를 받을 때,
```c++
auto mySimpleSmartPtr = make_unique<Simple>();
processData(mySimpleSmartPtr.get());
// get을 통해 내부의 일반 포인터를 리턴받을 수 있음


mySimpleSmartPtr.reset() // 리소스 해제 + nullptr 초기화
mySimpleSmartPtr.reset(new Simple()); // 리소스 해제 + 재할당

Simple* pSimple = mySimpleSmartPtr.release(); // 일반 포인터 반환 후, 소유권 해제

delete pSimple;
simple = nullptr;

// unique_ptr는 컨셉 자체가 단독 소유권이기에 복사는 불가함
// 유틸리티 중 std::move()를 이용하면 소유권 이동 가능 ( 추후 추가 설명 )
class Foo
{
public:
    Foo(unique_ptr<int> data) : mData(move(data)) {}
    unique_ptr<int> mData;
};

auto myIntSmartPtr = make_unique<int>(42);
Foo f(move(myIntSmartPtr));

```

**unique_ptr와 C 스타일 배열**
unique_ptr는 기존 C스타일 동적할당 배열 저장에 적합하다.
```c++
auto myVariableSIzedArray = make_unique<int[]>(10);
```
하지만, array, vector같은 표준 컨테이너 사용하는 것이 바람직

**커스텀 제거자**
기본적으로 unique_ptr은 new와 delete로 메모리 할당 해제
=> 다른방식 (생성자 , 소멸자)  
![image](https://user-images.githubusercontent.com/85000118/137414847-93cb51e1-096a-4102-89f5-429dd39f9e0f.png)


**shared_ptr**
unique_ptr와 비슷한 생성
```c++
auto mySimpleSmartPtr = make_shared<Simple>();
```
C++17부터는 shared_ptr도 unique_ptr처럼 동적할당 배열의 포인터 저장 가능
이전에는 불가능
=> 지원해도 C배열보다 STL 컨테이너 사용이 바람직하다.

shared_ptr도  **get**, **reset** 메서드를 제공함
다른 점은 reset이 레퍼런스 카운팅 메커니즘에 따라 **마지막 shared_ptr가 제거되거나 리셋될 때 리소스가 해제됨**
**release**메서드는 지원하지 않음
**레퍼런스 카운터는 use_count() 메서드로 확인 가능**


C스타일 배열에는 new[]나 delete[]를 기본으로 사용하지만 아래처럼 사용도 가능
```c++
shared_ptr<int> myIntSmartPtr(malloc_int(42),free);
// 커스텀 제거자의 타입을 unique랑 다르게 지정하지 않아도 됨(decltype(free)*)
```

**shared_ptr 예시**
=> 파일 포인터 저장 예시
```c++
void CloseFile(FILE* filePtr)
{
	if (filePtr == nullptr) return;
	fclose(filePtr);
	cout << "FILE CLOSED" << endl;
}
void Tester(shared_ptr<FILE> &filePtr)
{
	shared_ptr<FILE> tester = filePtr;
	cout << filePtr.use_count() << endl;
}

int main()
{
	FILE* f = fopen("data.txt", "w");
	// f 포인터에 대한 스마트포인터 사용, 제거자는 CloseFile
	shared_ptr<FILE> filePtr(f, CloseFile);
	// 인자는 아마 해당 객체 포인터일 듯??
	cout << filePtr.use_count() << endl;
	Tester(filePtr);
	cout << filePtr.use_count() << endl;
}
```
결과 1 - 2 - 1
그냥 테스트 해봤음

**테스트 도중 중요한 사실**
=> 이전에 나온 설명이지만 인자에 &연산자를 붙이지 않으면, 1-3-1 이 출력됨. 즉, 복제 과정을 거쳐서 원본 1개 + 복제본 1개 + 함수 내부 1개 사용


**shared_ptr 캐스팅하기**
shared_ptr 캐스팅을 위한 함수
 - const_pointer_cast()
 - dynamic_pointer_cast()
 - static_pointer_cast()
 - reinterpret_pointer_cast()  => C++17 추가
추후 설명

**레퍼런스 카운팅 필요 이유**
현재 사용중인 특정 객체 추적하는 메커니즘
=> 이를 이용하여 스마트포인터의 중복삭제를 막음

```c++
Simple* mySimple = new Simple();
shared_ptr<Simple> smartPtr1(mySimple);
shared_ptr<Simple> smartPtr2(mySimple);
```
위 코드는 참조 횟수가 0이 되면서 두 스마트포인터가 서로 객체 해제를 시도한다.
**컴파일러나 상황에 따라 프로그램이 죽을 수도 있음**
C++ 표준을 따른 정상 동작

포인터 2개로 가리키지 말고, 복사본을 사용해야함
```c++
auto smartPtr1 = make_shared<Simple>();
shared_ptr<Simple> smartPtr2(smartPtr1);
```
=> 일반 포인터가 아닌 make_shared로 제작함
=> shared_ptr의 생성자에 일반 포인터면 아마 가리키고, 스마트포인터면 복사인 듯?
unique는 컨셉자체가 복제 생성자를 지원하지 않음

**앨리어싱**
shared_ptr는 aliasing을 지원
=> 소유한 포인터를 다른 shared_ptr와 공유하면서 다른 객체를 가리킬 수 있음
```c++
class Foo {
public:
	Foo(int value) : mData(value) {}
	int mData;
};

auto foo = make_shared<Foo>(42);
auto aliasing = shared_ptr<int>(foo, &foo->mData);
// foo 인스턴스의 mData
```
둘 모두 삭제되어야 Foo 객체가 삭제됨
=> shared를 안쓰고 별도로 그 객체의 멤버를 가져다 쓰면 객체가 삭제될 때 같이 삭제됨

소유한 포인터에 대한 비교 연산은 owner_before() 메서드 또는
std::owner_less 클래스 이용
=> 이런 기능은 set에 저장할 때 같은 특수 상황에 유용

### **week_ptr**
weak_ptr는 shared_ptr가 가리키는 리소스의 레퍼런스 관리에 사용
=> weak_ptr는리소스를 직접 소유하지 않기 때문에 shared_ptr가 해당 리소스를 해제하는데 아무런 영향을 안미침
 
 - weak_ptr는 스코프 벗어날 때, 리소스를 삭제하지 않음
 - shared_ptr가 리소스를 해제했는지 알 수 있음
 - 생성자로 weak_ptr나 shared_ptr를 인수로 받음
 - weak_ptr에 저장된 포인터 접근하려면 shared_ptr로 변환 필요 ( 2가지 방법 )
=> weak_ptr의 lock() 메서드를 이용해 shared_ptr 리턴받음  **이떄 shared_ptr에 연결된 weak_ptr가 해제되면 shared_ptr 값은 nullptr가 됨**
=> shared_ptr 생성자에 weak_ptr를 인수로 전달하여 shared_ptr 새로 생성 **이때 shared_ptr에 연결된 weak_ptr가 해제되면 std::bad_weak_ptr 익셉션 발생

```c++
#include <memory>
#include <iostream>

using namespace std;

class Simple
{
public:
	Simple() { cout << "Simple constructor called!" << endl; }
	~Simple() { cout << "Simple destructor called!" << endl; }
};

void useResource(weak_ptr<Simple>& weakSimple)
{
	auto resource = weakSimple.lock();
	if (resource) {
		cout << "Resource still alive." << endl;
	} else {
		cout << "Resource has been freed!" << endl;
	}
}

int main()
{
	auto sharedSimple = make_shared<Simple>();
	weak_ptr<Simple> weakSimple(sharedSimple);

	// weak_ptr를 사용한다.
	useResource(weakSimple);

	// shared_ptr를 리셋한다.
	// Simple 리소스에는 shared_ptr가 하나 뿐이므로
	// weak_ptr가 남아 있더라도 이렇게 하면 리소스를 해제한다. 
	sharedSimple.reset();

	// weak_ptr를 다시 한 번 사용한다.
	useResource(weakSimple);

    return 0;
}
```

**이동 의미론**
3 가지 스마트포인터는 성능향상을 위해 이동 의미론 지원 (추후 소개)
리턴하는 과정을 굉장히 효율적 처리가 가능
example Code
```c++
#include <iostream>
#include <memory>

using namespace std;

class Simple
{
public:
	Simple() { cout << "Simple constructor called!" << endl; }
	~Simple() { cout << "Simple destructor called!" << endl; }
};

unique_ptr<Simple> create()
{
	auto ptr = make_unique<Simple>();
	//  ptr을 사용하는 코드를 여기에 작성한다.
	return ptr;
}

int main()
{
	unique_ptr<Simple> mySmartPtr1 = create();
	auto mySmartPtr2 = create();
	return 0;
}
```
아마 스코프를 벗어나도 리턴받은거면 괜찮다가 아닐지?


**enalbe_shared_from_this**
믹스인 클래스인 이것을 이용하면 shared, weak을 안전하게 리턴 가능
 - shared_from_this() : 객체의 소유권을 공유하는 shared_ptr 리턴
 - weak_from_this() : 객체의 소유권을 추적하는 weak_ptr 리턴 ( C++17 추가 )
```c++
#include <memory>

using namespace std;

class Foo : public enable_shared_from_this<Foo>
{
public:
	shared_ptr<Foo> getPointer() {
		return shared_from_this();
// return shared_ptr<Foo>(this);
// 이렇게 구현하면 2개의 스마트포인터가 같은 객체를 가리켜 서로 삭제하려 함
	}
};

int main()
{
	auto ptr1 = make_shared<Foo>();
	auto ptr2 = ptr1->getPointer();
}
```

**폐기된 auto_ptr**
C++11이전에는 auto_ptr을 제공했는데 심각한 단점이 몇 개 있음
**그 중 하나가 vector같은 STL 컨테이너 안에서 제대로 동작하지 않음**
C++11, 14에서는 공식지원 안함
17에서 완전 폐기


### **흔히 발생하는 메모리 문제**
흔히 발생하는 유형들

**스트링 과소 할당 문제**
C스타일 스트링 가장 흔한 문제 과소 할당(underallocation)
=> string 끝에 '\0'문자 넣을 공간을 뺴놓고 할당한 경우가 많음
 
해결 방법 3가지
 1. C++ 스타일 스트링 사용 ( 알아서 관리 )
 2. 스택을 사용하지 않고, 힙을 사용 => 부족하면 추가 할당 및 복사
 3. 최대 문자 수(널 포함) 입력 받아서 그 길이를 넘어선 부분은 리턴 X  남은 공간과 위치를 추적


** 메모리 경계 침범**
흔히 발생하는 버퍼 오버플로우 에러 등이 대표적
ex. NULL이 있는 곳까지 데이터를 씌우는 함수가, NULL을 만나지 못하고 의도하지 않은 부분까지 덮어씌워버림
현재 메모리 검사 도구들은 이런 문제를 찾아내주고
C++ 기능들을 이용하면 이런 버그 방지 가능


**메모리 누수**
스마트 포인터가 필요한 이유
=> 해제 할 동적 할당 대상에 대한 주소를 잃거나 깜빡하고 처리하지 않아 발생

몇몇 툴들에서는 메모리 누수 감지 도구들도 있음
VC++ 디버그 라이브러리에 있으며, 기본 설정은 이 기능을 사용하지 않지만,
MFC프로젝트는 자동 활성화

```c++
#define _CRTDBG_MAP_ALLOC
#include <cstdlib>
#include <crtdbg.h>

// 그리고 new를 재정의
#ifdef _DEBUG
	#ifndef DBG_NEW
		#define DBG_NEW new ( _NOMAL_BLOCK, __FILE__, __LINE__ )
		#define new DBG_NEW
	#endif
#endif


// 마지막 main 함수 첫 부분에 삽입
_CrtSetDbgFlag(_CRTDBG_ALLOC_MEM_DF | _CRTDBG_LEAK_CHECK_DF);

```
기능 사용을 위해서는 위 코드 추가 ( 반드시 순서대로 )
디버그 모드로 컴파일 해야함

애플리케이션 종료할 때, CRT 라이브러리는 감지된 모든 메모리 누수 현상을 디버그 콘솔에 출력함

![image](https://user-images.githubusercontent.com/85000118/137445816-26479145-8abb-4ab8-b001-1d7aa55f4569.png)

메인 시작 부분 _CrtSetBreakAlloc(147) 함수 호출하면 메모리가 할당되는 순간 실행을 중단하고 디버거 구동하도록 설정 가능 ( 147번쨰 메모리 할당하는 순간 )

대부분 디버거가 지원하는 왓치 포인트(조사식) 기능 이용 - 문제 발생하는 순간 실행 중단

**밸그라인드를 이용한 리눅스 메모리 누수 탐지**
밸그라인드(valgrind)는 무료 리눅스용 오픈소스 도구로 할당한 객체를 해제하지 않는 지점을 정확히 찾아줌

![image](https://user-images.githubusercontent.com/85000118/137456407-e8ecb71d-7188-4ce2-882c-366437031f8f.png)
![image](https://user-images.githubusercontent.com/85000118/137456438-6a37cc7d-8f52-41b7-93ab-0967c2edcf8c.png)


**중복 삭제와 잘못된 포인터**
delete로 메모리 해제하면 그 메모리부분을 다른 곳에서 사용 가능함
하지만, 그 포인터를 계속 쓰는 것을 막을 수 없음
=> 댕글링 포인터(dangling pointer)라 부름
=> 중복 삭제하면 (delete2번) 이미 할당되어 있는 다른 객체를 해제해버리게 됨

중복 삭제 문제
or
해제한 메모리를 다시 사용하는 문제 찾기는 굉장히 힘듦

혹시라도, 삭제한 객체의 메모리 영역 데이터가 그대로 남아있어 제대로 작동할 순 있는데,
그럼에도 문제가 있어 바람직하지 못함

스마트포인터를 이용하지 않는다면,
반드시 해제 작업과 해제 후, nullptr 할당 작업을 필수로 해야한다.


결론 :
1. 포인터 내부 작동 방식을 이해해야 함 ( 주소와 화살표 )
2. 동적 메모리 관련 문제가 발생하지 않도록 C++ 컨테이너, 클래스, 스마트포인터를 사용

C스타일 보다  C++!
