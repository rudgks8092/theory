클래스와 객체 마스터하기

### **friend**
클래스 내에 다른 클래스의 멤버 함수 or 비멤버 함수를 friend로 선언하는 기능
=> friend로 지정된 대상은 protected or private 데이터 멤버와 메서드에 접근 가능

사용방법은 다른 클래스의 메서드 또는 일반 함수 앞에 friend를 붙인다.
```c++
friend class Bar;
// Bar 클래스의 모든 것
friend void Bar::processFoo(const Foo& foo);
// Bar 내의 클래스 멤버함수
friend void dumpFoo(const Foo& foo);
// 스탠드 얼론 함수 ( 클래스 멤버함수 아닌 독립 함수 ) 사용가능

void dumpFoo(const& Foo& foo)
{
 // Foo 클래스 멤버의 protected, private에 접근 후 콘솔 출력
}
```
정의도 동일하게 하면 된다.
=> 다만 friend 선언한 클래스의 모든 데이터 멤버에 접근 가능하다는 점이 다름
=> 클래스 안에서 friend 지정해야만 함 ( 밖에서 지정하면 의미 없음 )
=> 너무 많이 사용하면 내부가 외부에 드러나 캡슐화 원칙이 깨짐 ( 필요할 떄만 사용 )


### **객체에 동적 메모리 할당하기**
동적 메모리에 관련된 까다로운 문제 다뤄보기

특정 멤버 변수가 2중 포인터로 있다면, new를 이용하여 동적 할당을 수행한다.
=> C++에서는 vector 같은 표준 컨테이너를 사용하는 것이 좋다.
```c++
// 생성자에서 가로 세로 에 맞춰 동적할당
Spreadsheet::Spreadsheet(size_t width, size_t height)
	: mWidth(width)
	, mHeight(height)
{
	mCells = new SpreadsheetCell*[mWidth];
	for (size_t i = 0; i < mWidth; i++) {
		mCells[i] = new SpreadsheetCell[mHeight];
	}
}
```
스프레드 시트 클래스 내에 각 Cell에 대한 동적 할당을 수행한다.
(자바는 new SpreadsheetCell[mWidth][mHeight] 같이 간단하게 가능하지만, C++은 불가능 )

각 셀에 대한 값을 Get, Set 하는 메서드를 구현하면,
실제 존재하는 좌표인지 확인하는 작업이 필요하다.

```c++
void Spreadsheet::verifyCoordinate(size_t x, size_t y) const
{
	if (x >= mWidth || y >= mHeight) {
		throw std::out_of_range("");
	}
}

void Spreadsheet::setCellAt(size_t x, size_t y, const SpreadsheetCell& cell)
{
	verifyCoordinate(x, y);
	mCells[x][y] = cell;
}

SpreadsheetCell& Spreadsheet::getCellAt(size_t x, size_t y)
{
	verifyCoordinate(x, y);
	return mCells[x][y];
}
```
코드 중복을 피하는 원칙을 따르기 위하여, 각 get과 set에 동일한 부분을 메서드화 시켰고, exception을 이용하여 구현


**소멸자로 메모리 해제**
메모리를 다 썼다면 소멸자에서 해제하는 것이 바람직함 ( 컴파일러가 자동으로 호출 )

```c++
// 소멸자의 이름은 클래스 이름 앞에 ~(틸드) 를 붙인 것
Spreadsheet::~Spreadsheet()
{
	for (size_t i = 0; i < mWidth; i++) {
		delete[] mCells[i];
	}
	delete[] mCells;
	mCells = nullptr;
}
```
소멸자는 인수를 받지 않고 단 하나만 존재함
소멸자는 noexcept가 적용됨 ( 익셉션 발생하지 않는 함수 )
ex. void myNonThrowingFunction() noexcept { ... }
=> 이 키워드로 지정한 함수에서 except발생하면 프로그램이 중단됨 ( 추후 설명 )

**복제와 대입 처리**
복제생성자 or 대입 연산자는 직접 작성하지 않으면 컴파일러가 자동으로 만들어줌
=> 그렇게 만들어진 메서드는 각 멤버의 복제 생성자나 대입 연산자를 재귀 호출함
=> int, double 같은 기본 타입에 대해서는 **비트 단위 복제(bitwise copy)**, **얕은 복제(shallow copy)** or 대입이 적용됨

**위 처럼 단순 복제는 동적 할당 객체 처리에 문제가 발생함**
=> 앞의 2차원 Cell 동적할당한 멤버 변수를 단순 복제하면 포인터 주소 값만 대입하게 됨
=> 복제한 객체가 소멸되면, 비어있는 포인터를 가리킴
=> 댕글링 포인터가 됨 ( 가리키던 메모리가 해제되어 올바르지 않은 메모리의 포인터 )

**대입은 더 심각할 수 있음**
2개의 시트를 생성함을 가정
![image](https://user-images.githubusercontent.com/85000118/138012418-06d20860-f995-41ee-8b2f-8b00c5128e59.png)

s1 = s2를 수행하여 대입하게 되면,
![image](https://user-images.githubusercontent.com/85000118/138012452-1e34503a-37c9-4733-a3d4-5a5b8813254e.png)

이처럼 메모리가 미아(orphan)가 되어 메모리 누수가 발생하게 된다.

따라서, 복제 및 대입 연산자에서는 반드시 깊은 복제 deep copy를 적용해야 함
=> 동적 할당이 있다면 반드시 깊은 복제로 처리하도록 직접 정의 필요


**복제 생성자(deep copy ver)**
```c++
Spreadsheet::Spreadsheet(const Spreadsheet& src)
	: Spreadsheet(src.mWidth, src.mHeight)
{
	// 이 생성자의 ctor-이니셜라이저는 먼저 적절한 양의 메모리를 할당하는 작업을
	// 비복제 버전의 생성자에게 위임한다.

	// 그러고 나서 실제로 값을 복제하는 작업을 수행한다.
	for (size_t i = 0; i < mWidth; i++) {
		for (size_t j = 0; j < mHeight; j++) {
			mCells[i][j] = src.mCells[i][j];
		}
	}
}
```
위 처럼 깊은 복제를 수행할 수 있음
=> 복제 생성자 이기 때문에 기존 mCells는 없다 => 따라서, 삭제는 불필요

**대입 연산자**
단순히 구현하면 아래 순서로 동작한다.
1. 자신 대입 확인 ( if(this == &rhs) )
2. 기존 메모리를 해제
3. 메모리 새로 할당
4. 데이터 복제
5. *this 리턴

위 처럼 구현하면 단계가 많은 만큼 문제 발생할 여지도 많음
ex. 메모리 할당 루프에서 익셉션 발생 => 나머지 코드 건너뛰고 리턴
=> 인스턴스 손상되어 원하는 결과 반영 안됨

이런 문제를 해결하기 위하여 **복제 후 맞바꾸기(copy and swap)** 패턴을 적용

1. 비멤버 swap 메서드 생성 ( 다양한 STL 알고리즘을 활용하기 좋아서 )
2. 사용 클래스의 friend로 지정하여 사용
```c++
friend void swap(Spreadsheet& first, Spreadsheet& second) noexcept;
// 를 클래스 내부에 선언
```
안전하게 구현하려면 절대로 except를 던지면 안되기 떄문에 noexcept 지정
```c++
void swap(Spreadsheet& first, Spreadsheet& second) noexcept
{
	using std::swap;

	swap(first.mWidth, second.mWidth);
	swap(first.mHeight, second.mHeight);
	swap(first.mCells, second.mCells);
}
```
각 멤버 교체작업을 <utility> 헤더의 std::swap으로 처리
```c++
Spreadsheet& Spreadsheet::operator=(const Spreadsheet& rhs)
{
	// 자신을 대입하는지 확인한다.
	if (this == &rhs) {
		return *this;
	}

	// 복제 후 맞바꾸기(copy-and-swap) 패턴 적용
	Spreadsheet temp(rhs); // 모든 작업을 임시 인스턴스에서 처리한다.
	swap(*this, temp); // 예외를 발생하지 않는 연산으로만 처리한다.
	return *this;
}
```
**복제본을 이용하여 익셉션에 대한 안정성을 높일 수 있음**

구현 과정은 3가지로 나눌 수 있다.
1. 임시 복제본을 생성 ( 현 객체의 상태를 변경하지 않아, 익셉션 발생해도 문제 X )
2. swap 함수를 이용하여 복제본과 교체 ( 익셉션 발생하면 절대 안됨 )
3. 임시 객체를 제거


**대입과 값 전달 방식 금지**
동적으로 할당할 때, 아예 그냥 복제나 대입을 못하게 만드는 방법이 간편할 수 있음
=>복제 생성자, 복제 대입 연산자의 선언에 = delete를 하여 삭제하면 됨
=> 삭제된 연산자나 생성자를 하려하면, 삭제된 함수 사용하려 한다는 오류 발생함
=> 만약 컴파일러가 명시적으로 삭제하는 기능을 지원하지 않는다면 private으로 지정하여 작성 안하는 방식 사용


### **이동 의미론으로 이동 처리하기**

**객체에 이동 의미론을 적용하려면 이동 생성자와 이동대입 연산자 정의 필요**
=> 컴파일러가 원본을 임시로 만들어 대입 연산 수행한 후, 객체를 제거함
=> 원본 객체의 멤버를 새로운 객체의 멤버로 이동 시킴 ( 그 후, 데이터 초기화 )

메모리를 비롯한 리소스 소유권을 다른 객체로 이동 시킴 ( 얇은 복제와 비슷 )
=> 소유권을 전환하여 댕글링 포인터 or 메모리 누수를 방지

구현하기 전에 우측값(rvalue), 우측값 레퍼런스(rvalue reference)를 알아둬야 함

**우측값 레퍼런스**
C++에서의 lvalue(좌측값)은 변수처럼 이름과 주소를 가진 대상 (대입문 왼쪽)
우측 값은 리터럴, 임시 객체, 값처럼 좌측값이 아닌 모든 대상
```c++
int a = 4 * 2;
```
lvalue = a  ( &a로 주소를 가져올 수 있음)
rvalue = 4*2 ( 임시 값 -> 실행하고 제거 )

우측 값 레퍼런스
=> 우측 값이 임시 객체일 떄 적용되는 개념
=> 임시 객체에 대해 적용할 함수를 컴파일러가 선택하기 위한 용도로 사용

이 **우측 값 레퍼런스** 개념을 이용하여 크기가 큰 객체 복사 연산이 나와도
컴파일러는 이 값이 나중에 삭제될 임시 객체라는 점을 이용하여 우측 값에 대한 포인터를 복사하는 방식으로 실행 가능

함수 매개변수에 &&를 붙여서 우측 값 레퍼런스 만들기 가능 (ex. type&& name)

일반적인 임시 객체는 => const type&로 취급
오버로딩 버전 중 &&를 사용하는 것이 있으면 그 버전으로 임시 객체 처리

```c++
// 좌측값 레퍼런스 매개변수
void helper(std::string&& message)
{
}
void handleMessage(std::string& message)
{
	cout << "handleMessage with lvalue reference: " << message << endl;
}

// 우측값 레퍼런스 매개변수
void handleMessage(std::string&& message)
{
	cout << "handleMessage with rvalue reference: " << message << endl;
	helper(std::move(message));
// helper는 && 로 우측 값 레퍼런스를 받기 때문에, 그냥 이름 있는 변수(좌측 값)인 message를 보내면 컴파일 오류 발생함
// 우측 값 매개변수로 받았어도, 현재는 이 함수에서는 이름 있는 변수로 좌측 값임
}

int main()
{
	std::string a = "Hello ";
	std::string b = "World";

	// 이름 있는 변수를 처리한다.
	handleMessage(a);             // handleMessage(string& value)를 호출한다.

	// 표현식을 처리한다.
	handleMessage(a + b);         // handleMessage(string&& value)를 호출한다.

	// 리터럴을 처리한다.
	handleMessage("Hello World"); // handleMessage(string&& value)을 호출한다.

	// 이름 있는 변수를 처리하고 우측값 레퍼런스 메서드를 사용하도록 설정한다.
	handleMessage(std::move(b));  // handleMessage(string&& value)를 호출한다.

	return 0;
}
```

좌측 값 레퍼런스를 받는 handleMessage 함수 제거하고 handleMessage(b)를 수행하면 컴파일 오류 발생
=> 우측 값 레퍼런스 타입의 매개변수(string&& message)를 좌측 값(b)인수에 바인딩할 수 없기 때문
=> 맨 아래 사용 예시처럼 좌측값을 우측값으로 캐스팅하는 std::move를 사용하면 호출이 가능해진다.

### 이름있는 변수는 좌측값이다.
=> a+b 같은 임시 변수는 우측 값이다.
=> 좌측 값을 우측값으로 전달하려면 std::move를 이용하여 캐스팅이 필요하다.

비슷한 예시
```c++
int& i = 2;  // 상수에 대한 레퍼런스 - 에러
// => int &&i = 2; 
int a=2,b=3;
int& j = a+b; // 임시 객체에 대한 레퍼런스 - 에러
// => int &&j = a+b;

// && 우측 값 레퍼런스를 이용하면 오류는 없지만 사용할 일이 없다.
```


**이동 의미론 구현 방법**
우측값 레퍼런스로 구현한다.
클래스에 이동 의미론 추가 => 이동 생성자, 이동 대입 연산자 구현 (noexcept 필수)
=> STL 호환성 유지를 위해서 객체를 이동만 시키고 except는 발생하지 않도록 구현

```c++
// 헤더
class Spreadsheet
{
public:
	Spreadsheet(Spreadsheet&& src) noexcept; // 이동 생성자

	Spreadsheet& operator=(const Spreadsheet& rhs); // 복제 대입 연산자
	Spreadsheet& operator=(Spreadsheet&& rhs) noexcept;  // 이동 대입 연산자

private:
	void cleanup() noexcept;
	void moveFrom(Spreadsheet& src) noexcept;
};

// 소스파일
void Spreadsheet::cleanup() noexcept
{
	for (size_t i = 0; i < mWidth; i++) {
		delete[] mCells[i];
	}
	delete[] mCells;
	mCells = nullptr;
	mWidth = mHeight = 0;
}
void Spreadsheet::moveFrom(Spreadsheet& src) noexcept
{
	// 얕은 복제 적용
	mWidth = src.mWidth;
	mHeight = src.mHeight;
	mCells = src.mCells;

	// 원본 객체를 리셋한다. 소유권이 이동했기 때문이다.
	src.mWidth = 0;
	src.mHeight = 0;
	src.mCells = nullptr;
}
// 이동 생성자
Spreadsheet::Spreadsheet(Spreadsheet&& src) noexcept
{
	cout << "Move constructor" << endl;

	moveFrom(src);
}
// 이동 대입 연산자
Spreadsheet& Spreadsheet::operator=(Spreadsheet&& rhs) noexcept
{
	cout << "Move assignment operator" << endl;

	// 자기 자신 대입 여부 검사
	if (this == &rhs) {
		return *this;
	}

	// 기존 메모리 해제
	cleanup();

	moveFrom(rhs);

	return *this;
}
```
비교를 위한 복제 대입 연산자를 제외하고 추가된 코드들이다.
원본 객체를 전달하고 삭제하는 개념이기 때문에 전달 후에는 nullptr로 변경함

일반 생성자, 복제대입 연산자와 마찬가지로 default, delete가 가능함

복제 생성자 때와 동일하게,
복제 생성자, 복제대입 연산자, 이동대입 연산자, 소멸자를 직접 선언하지 않았다면,
컴파일러가 디폴트 이동 생성자, 이동 대입 연산자를 생성해줌

**동작할당 메모리 사용 코드가 있다면 반드시 구현할 5가지 (5의 규칙-Rule of Five)**
 - 소멸자
 - 복제 생성자
 - 이동 생성자
 - 복제 대입 연산자
 - 이동 대입 연산자


**객체 데이터 멤버 이동하기**
만약 멤버가 객체가 존재한다면, 위의 moveFrom 에서 std::move를 이용한다.
string 타입의 mName의 이동
=> mName = std::move(src.mName);



**swap()함수로 구현한 이동 생성자와 이동 대입 연산자**
moveFrom 헬퍼메서드에서는 얕은 복사로 이동을 처리하였다.
=> 멤버 변수가 추가되면 swap, moveFrom 동시에 수정해야 함
=> 이동 생성자, 대입 연산자 또한, swap함수로 구현

```c++
// 이동 생성자
Spreadsheet::Spreadsheet(Spreadsheet&& src) noexcept
	: Spreadsheet()
{
	cout << "Move constructor" << endl;

	swap(*this, src);
}

// 이동 대입 연산자
Spreadsheet& Spreadsheet::operator=(Spreadsheet&& rhs) noexcept
{
	cout << "Move assignment operator" << endl;

	Spreadsheet temp(std::move(rhs));
	swap(*this, temp);
	return *this;
}
```

임시 객체를 생성하기 때문에, 효율성이 살짝 떨어질 수 있으나,
=> 코드가 줄고 멤버 추가 시 swap만 수정하면 되기 떄문에 버그 발생 확률을 낮춤



**이동 연산자 테스트**

```c++
Spreadsheet createObject()
{
	return Spreadsheet(3, 2);
}

int main()
{
	vector<Spreadsheet> vec;
	for (int i = 0; i < 2; ++i) {
		cout << "Iteration " << i << endl;
		vec.push_back(Spreadsheet(100, 100));
		cout << endl;
	}

	Spreadsheet s(2,3);
	s = createObject();

	Spreadsheet s2(5,6);
	s2 = s;

	return 0;
}
```
vector 동적 할당 가능 이유
=> 필요에 따라 더 큰 덩어리의 메모리를 할당하여 복제 및 이동함
=> 이동 생성자가 있으면, 복제하지 않고 이동시켜 효율적

![image](https://user-images.githubusercontent.com/85000118/138194147-b666c8bb-0ae7-4dbd-8efe-48bd9a589479.png)
![image](https://user-images.githubusercontent.com/85000118/138194231-3039b8ef-6b44-476b-ac82-edc8068ceff2.png)

위 출력 (1~5) 처럼 반복문의 수행순서가 다음과 같다.
1. 일반 생성자 호출
2. vector 공간 조절 후, 이동 생성자 호출하여 이동
3. 새로운 객체 생성
4. vector에 담긴 객체 및 새로운 객체 이동 생성자

다음의 객체 생성 단계들에서는,
1. s 객체 생성
2. createObject함수에서 생성자로 생성 후, 리턴
3. createObject로 생성한 객체는 임시 객체로 사라질 예정이기에, 복제 대입이 아닌, 이동대입 연산자 호출
4. s2 객체 생성
5. s2에 s를 대입하며 복제 대입 연산자 호출 (우변의 객체가 임시가 아닌, 이름있는 lvalue 객체이기 때문)
6. 복제 대입 연산자는 임시 복제본을 생성하므로, 복제 생성자가 호출되는데 복제 생성자는 일반 생성자를 한번 호출함


이동 의미론을 구현하지 않으면 이동 관련 부분은 모두 복제 생성자, 복제 대입 연산자로 대체됨
=> 앞의 예시에서는 이동 의미론을 통하면 메모리 할당이 필요 없지만, 복제 생성자 및 복제 대입 생성자는 101개를 할당함 ( 특정 상황에 성능을 크게 높일 수 있음 )


**이동 의미론으로 swap함수 구현하기**
![image](https://user-images.githubusercontent.com/85000118/138214268-337529a3-a3ea-44fc-8b34-12693b436404.png)

std::swap이 위 처럼 복제가 발생하지 않도록 구현하였다.


### **영의 규칙**
5의 규칙 외에 최신 C++에는 영의 규칙(Rule of Zero)를 추가했다.

앞의 예제는 SpreadsheetCell**를 멤버를 사용함으로써 동적할당이 필요하여 5의 규칙 적용이 필요했다.
=> vector<vector<SpreadsheetCell>>을 사용하여 구현 할 필요가 없도록 작성하자


### **메서드의 종류**

**static 메서드**
객체 단위가 아닌 클래스 전체 단위로 적용되는 메서드
SpreadsheetCell 예제에서는 doubleToString, stringToDouble으로
각 객체의 정보에 접근하지 않는 헬퍼 메서드들은 static으로 정의할 수 있다.
```c++
static std::string doubleToString(double inValue);
static double stringToDouble(std::string_view inString);
// 선언부에만 적고 구현부에는 안적어도 됨
```

static 정의하면 this나 non-static 멤버에 접근이 불가능함
=> 일반함수와 비슷하지만, 클래스의 private static이나 protected static 멤버만 접근할 수 있다는 점이 다름
=> 같은 타입의 객체를 포인터나 레퍼런스로 전달하면 non-static도 접근 가능

클래스 내부에서는 일반함수처럼 호출 가능함
public으로 외부에서 호출할 떄는 스코프 지정 연산자 사용( :: )


**const 메서드**
객체 멤버 변경 불가를 보장하는 메서드 ( 보통 get에 사용 )
=> 변경 시도하면 컴파일 에러
```c++
// 선언부
double getValue() const;

// 구현부
double SpreadsheetCell::getValue() const
{
}
```
const는 메서드 프로토타입의 일부라서 구현부에도 적어야 함
static 메서드는 기본적으로 객체 접근을 하지않아, const는 의미 없음

객체를 const로 선언하면 const 메서드만 접근 가능 (set 계열 함수 사용 불가)


**mutable 데이터 멤버**
const 메서드에서 객체의 데이터 멤버를 변경하는 경우가 있음
(ex. get 을 호출한 횟수를 count하는 멤버 변수 )
=> mutable 로 선언하면 가능함

```c++
mutable size_t mNumAccesses = 0;
```
이러면 const 메서드에서도 변경이 가능함


### **메서드 오버로딩**
동일한 이름으로 매개변수 타입이나 개수가 다르게 여러 개 정의 하는 것
( 리턴 타입은 컴파일러가 정확히 결정하기 어려워, 지원하지 않음 )
```c++
void set(double inValue);
void set(std::string_view inString);
```

컴파일러가 set을 호출할 때 매개변수를 보고 어떤 함수를 호출할지 결정하는 것
=> 오버로딩 결정(Overload resolution)

**const 기반 오버로딩**
const 여부로 오버로딩될 수 있다. 모두 동일하지만, const 여부만 다름

const만 동일할 경우 코드가 동일하다면, 코드 중복을 피하기 위해
const_cast()을 이용한다.
```c++
const SpreadsheetCell& Spreadsheet::getCellAt(size_t x, size_t y) const
{
	verifyCoordinate(x, y);
	return mCells[x][y];
}

SpreadsheetCell& Spreadsheet::getCellAt(size_t x, size_t y)
{
	return const_cast<SpreadsheetCell&>(std::as_const(*this).getCellAt(x, y));
	//return const_cast<SpreadsheetCell&>(static_cast<const Spreadsheet&>(*this).getCellAt(x, y)); // C++17 이전 문법
}
```
const는 그대로 구현, non-const 버전은 const 버전을 적절히 캐스팅하여 호출하는 방식

```c++
	Spreadsheet sheet1(5, 6);
	SpreadsheetCell& cell1 = sheet1.getCellAt(1, 1);

	const Spreadsheet sheet2(5, 6);
	const SpreadsheetCell& cell2 = sheet2.getCellAt(1, 1);
```

본 코드에서는 const 버전의 코드가 짧아 크게 차이는 없지만,
코드가 길어지면 코드 중복을 많이 줄일 수 있어짐


**명시적으로 오버로딩 제거하기**
특정 인수에 대해서 메서드 호출하지 못하게 하기

```c++
class cl {
public:
	void foo(int i);
};

cl c;
c.foo(123);
c.foo(1.23);
```
위 코드는 컴파일러 마음대로 double을 정수로 변환하여 호출된다.   (경고는 해줌)
void foo(double d) = delete;
위 코드로 double에 대한 인수를 삭제하면 컴파일 에러로 처리 됨


### **인라인 메서드**
별도의 코드 블록에 구현해 호출하지 않고, 호출부분에서 바로 구현하는 방식
=> 인라이닝(inlining)  - 구현된 메서드 = inline 메서드
#define 매크로보다 인라인 메서드가 안전함


구현부에서 함수 앞에 inline을 붙이는 걸로 만들 수 있다.
```c++
inline double SpreadsheetCell::getValue() const
{
 // 코드 구현
}
```

컴파일러는 해당 함수의 호출 부분을 호출로 처리하는게 아닌, 함수의 본문을 바로 넣는다.
=> 함수 호출에 발생하는 오버헤드를 줄일 수 있다.
**컴파일러는 inline 키워드를 참고만 함 - 성능에 문제 있을 수 있다면 그냥 무시 할수도**

제약사항은 호출 코드에서 함수의 정의부분을 확인할 수 있어야 함
=>프로토타입과 구현 코드를 헤더 파일에 작성해야 함
=> 고급 C++ 컴파일러는 헤더파일에 작성하지 않아도 됨
ex. Visual C++은 링크 타임 코드 생성 기능 지원  (GCC Clang도 비슷한 기능 제공 )
=> inline 선언 하지 않거나, 헤더파일에 정의 하지 않아도 크기가 작으면 자동으로 inline처리

```c++

class SpreadsheetCell
{
public:
	std::string getString() const
	{
		mNumAccesses++;
		return doubleToString(mValue);
	}
};
```
헤더에 위와 같은 형태로 정의한다. ( 고급 C++ 컴파일러가 아니라면 )
( 일부 디버거는 싱글스텝에서 함수 본문을 건너 뛸 수 있음 )

간단한 함수에 대해서만 inline으로 선언해야 한다.
(최신 컴파일러는 코드 비대화 기준에 따라, 인라인으로 처리하지 않기도 함)


### **디폴트 인수**
default argument = 함수 프로토타입(선언)에서 매개변수의 기본값을 지정할 수 있음 (구현부에서는 안됨)
=> 사용자가 직접 지정하여 호출하면 디폴트 값을 무시함

작성 조건 : 디폴트 인수를 사용하는 매개변수는 가장 끝 오른쪽부터 비는 곳 없이 작성해야함
ex. void ABC(int a = 10, int b, int c=10);
위 처럼 중간에 끊기면 안된다.

디폴트 생성자가 있으면서,
모든 매개변수에 디폴트 값이 지정된 생성자가 있으면
컴파일 에러 발생(컴파일러는 어떤 것을 호출하는지 알 수 없음)

각 상황에 맞게 오버로딩을 할지, 디폴트 인수를 지정할지 선택해서 사용


### **데이터 멤버의 종류**
멤버 변수 또한, static, const, reference, const reference 등 지정이 가능함

**static 데이터 멤버**
클래스가 공용으로 갖는 멤버 변수
=> 전역과 비슷하지만 클래스 범위를 벗나알 수 없음
기본적으로 0이나 nullptr로 초기화 됨
```c++
// 선언방법(헤더파일)
static size_t sCounter;

// 초기화 방법(소스파일)
size_t Spreadsheet::sCounter = 0;
```

c++17부터 static 멤버를 inline으로 선언할 수 있음
=> 위의 초기화 과정을 헤더에 작성 가능

```c++
class SpreadsheetCell
{
private:
	static inline size_t sCounter = 0;
};
```

**클래스 메서드에서 static 데이터 멤버 접근하기**
예를 들어 static으로 ID 값을 가지고 있고, 생성자에서 증가시키며 사용한다고 하면,

```c++
Spreadsheet::Spreadsheet(size_t width, size_t height,
	const SpreadsheetApplication& theApp)
	: mId(sCounter++)
	, mWidth(width)
	, mHeight(height)
{
	cout << mId << endl;
	mCells = new SpreadsheetCell*[mWidth];
	for (size_t i = 0; i < mWidth; i++) {
		mCells[i] = new SpreadsheetCell[mHeight];
	}
}
```

위처럼 사용할 수 있다. ( 일반 멤버인 것 처럼 )
=> 단, 복제 대입 연산자에서 mId를 복제할 수 있으니 const로 선언한다.


**메서드 밖에서 static 데이터 멤버 접근**
static 멤버가 public이라면 스코프 지정 연산자 (::)로 접근이 가능하다.
=> 하지만, 멤버를 public 사용하는 것은 바람직하지 않다. 
=> const를 이용하거나, static set/get 메서드를 이용한다.


**const static 데이터 멤버**
클래스 상수(class constant)를 정의할 때 사용해야 한다.
static const는 헤더에 선언과 동시에 초기화 가능

```c++
static const size_t kMaxHeight = 100;
static const size_t kMaxWeight = 100;
```
위 처럼 클래스 전용의 변치 않는 상수 지정에 사용한다.
```c++
Spreadsheet::Spreadsheet(size_t width, size_t height,
	const SpreadsheetApplication& theApp)
	: mId(sCounter++)
	, mWidth(std::min(width, kMaxWidth))
	, mHeight(std::min(height, kMaxHeight))
{
}
```
<algorithm>의 std::min을 사용하여 위 처럼 활용이 가능하다.
=> exception을 던져도 되지만, 생성자에서 exception 수행하면 소멸자가 호출되지 않는다. ( 중단만하고, 소멸이 안됨 )

mId를 const로 선언하게 되면 반드시 생성자 이니셜라이저에서 초기화 해야 한다.
=> const를 생성자에서 처리하지 않으면 컴파일 에러

위 예제 대신에 const static 변수를 생성자에서 디폴트 인자로 사용해도 된다.


**레퍼런스 데이터 멤버**
스프레드 시트 클래스와 이를 사용할 Application 클래스를 작성하기 전에,
둘은 서로를 참조하여 정보를 필요로 한다.
둘은 순환참조가 발생하여 #include 만으로는 해결이 불가능하다.
이를 위해 포워드 선언 (forward declaration) - 전방 선언을 하여 클래스에 대해 알린다. 
=> 포워드 선언은 컴파일과 링크속도를 높일 수 있는데 (추후 소개)

```c++
class SpreadsheetApplication; // forward declaration

class Spreadsheet
{
public:
	Spreadsheet(size_t width, size_t height,
		const SpreadsheetApplication& theApp);
private:
	SpreadsheetApplication& mTheApp;
};
```
포인터보다 레퍼런스 사용이 바람직하여 레퍼런스 데이터 멤버 추가
=> 시트는 항상 어플을 참조하기 때문인데, 포인터는 관계를 보장하지 못함

레퍼런스는 const와 마찬가지로 가리키는 대상 없이는 존재가 불가능함
=> 생성자 이니셜라이저에서 반드시 지정해야 함
=> 레퍼런스 초기화 이후에는 객체를 변경할 수 없음 (대입연산자로 값을 대입할 수 없음)
=> 대입연산자가 제공할 수 없게되면 deleted로 지정됨

**이런식의 사용보다 MVC 패러다임에 따라 구성하는게 바람직**


**const 레퍼런스 데이터 멤버**
const 객체를 가리키는 레퍼런스 멤버

위의 Application 멤버나 인자에 const를 붙임으로써 사용 가능함
=> const 레퍼런스는 const 메서드만 호출 가능함 ( non-const 호출 시 컴파일 에러 )


### **중첩 클래스**
클래스 내에 중첩클래스, 타입 앨리어스(typedef), struct, enum도 선언 가능함
단, 스코프는 클래스 내로 제한됨 ( ::를 이용 )
구현할 때 ::로 ( Main::Sub::함수명(){ } ) 식으로 구현하여야 한다.

선언부를 class 안에 또 class를 지정하면 코드가 길어지기 때문에,
class 내부에 class Sub; 선언만 하고,
class Main::Sub { } 식으로 별도로 선언하는게 좋다.

접근 제어 규칙은
1. Main클래스는 Sub 클래스의 Public만 접근 가능
2. Sub 클래스는 상위 클래스의 모든 protected, private 멤버 접근이 가능함


### **클래스에 열거타입 정의하기**
#define 이용 대신, 열거 타입을 활용
```c++
// 클래스 코드 내부
enum class Color { Red = 1, Green, Blue, Yellow };


//멤버
Color mColor = Color::Red;
```
이 외에 setColor 같은 메서드를 구현하고 인자나 리턴 타입을 Color color 형태로 지정 가능하다.
사용은 MyClass::Color::Blue 형태로 넘기면 된다.


### **연산자 오버로딩**
객체에 대한 연산을 지원하기 위함

**시트 셀 덧셈 구현**
객체 지향 방식의 구현
Cell1 + Cell2 = Cell3  ( 기존 셀 불변 )
```c++
// 헤더
SpreadsheetCell add(const SpreadsheetCell& cell) const;
// 불변이기에 const

// 소스파일
SpreadsheetCell SpreadsheetCell::add(const SpreadsheetCell& cell) const
{
	return SpreadsheetCell(getValue() + cell.getValue());
}

// 사용
SpreadsheetCell myCell(4), anotherCell(5);
SpreadsheetCell aThirdCell = myCell.add(anotherCell);

```

**2번째 버전 Operator+ 오버로딩**

```c++
// 헤더
SpreadsheetCell operator+(const SpreadsheetCell& cell) const;

// 소스코드
SpreadsheetCell SpreadsheetCell::operator+(const SpreadsheetCell& cell) const
{
	return SpreadsheetCell(getValue() + cell.getValue());
}
```

Operator + 사이에 공백있어도 괜찮음
이제 +으로 두 개의 객체를 add할 수 있게 되었다.
operator 방식
1. 컴파일러가 프로그램 파싱에서 연산자를 확인
2. operator 함수 확인 (객체의 operator+ 아니면, 객체 둘을 인수로 받는 operator+전역함수)

```c++
SpreadsheetCell aThirdCell = myCell.add(anotherCell);
SpreadsheetCell aThirdCell = myCell + anotherCell;
SpreadsheetCell aThirdCell = myCell.operator+(anotherCell);
```
위의 3개는 동일하다.


**묵시적 변환**
위 처럼 작성하면 셀에 여러 타입의 값을 더할 수 있다.

```c++
SpreadsheetCell myCell(4), anotherCell;
string str = "Hello";
anotherCell = myCell + string_view(str);
anotherCell = myCell + 5.6;
anotherCell = myCell + 4;
```

컴파일러가 operator+를 찾고, 타입 변환 방법도 찾기 때문
 - double + 하는 Cell을 발견하면 double 인수로 받는 생성자로 생성한 뒤, 전달
 - string_view도 마찬가지로 임시객체 생성 전달

string_view를 더하는 것은 상식적으로 맞지 않음
=> 묵시적 변환을 없애기 위하여, string_view 인자 생성자 앞에 **explicit** 키워드
```c++
// 헤더에서만 가능 ( 정의부분 ) + 인수 하나만 지정 호출하는 생성자에 적합
// 여러개 받는 생성자는 디폴트 값이 지정된 생성자
explicit SpreadsheetCell(std::string_view initialValue);
```

묵시적 변환의 생성자 선택은 성능 저하를 초래할 수 있음
따라서, operator+의 double 버전도 만들어주면 됨
```c++
SpreadsheetCell SpreadsheetCell::operator+(double rhs) const
{
	return SpreadsheetCell(getValue() + rhs);
}
```


**3번째 operator+ 전역함수 구현**
묵시적 변환의 연산은 교환법칙이 성립하지 않음
ex. 5.6 + myCell;

묵시적 변환 => 객체가 연산자의 좌변
=> 객체에 대해서 호출하는 것이기 때문에 바꿀 수 없음

Operator+를 전역함수로 만들면 가능해짐 ( 특정 객체에 종속되지 않기 떄문 )
```c++
// 헤더파일에 선언
class SpreadsheetCell
{
};
// 클래스 정의 밖에 선언
SpreadsheetCell operator+(const SpreadsheetCell& lhs, const SpreadsheetCell& rhs);
```
```c++
Cell = 4 + myCell; // 정상 동작
Cell = 4.5 + 5.5;
// 위 연산은 operator+가 아닌, double 연산이 적용되어 아래로 변환
Cell = 10;
// 위 연산은 우변이 객체이어야 하기 때문에, explicit이 아니면서 double 인수 받는 생성자로 변환하여 대입 연산자 호출
```

**산술 연산자 오버로딩**
operator+와 비슷함 
기존 연산자와 비슷한 의미일 떄만 사용하는 것이 좋다.


**축약형 산술 연산자의 오버로딩**

+= -= 등에 대한 연산자 오버로딩 제공
축약형은 **반드시 좌변에 객체가 있어야 한다.**
=> 전역함수가 아닌 메서드로 구현

구현방법은 동일하지만, 코드 중복을 막기위해 += 버전을 +버전에서 불러오는 식 같이 변경하는 것이 좋다


**비교 연산자 오버로딩**
> < == 등의 오버로딩
좌변 우변 모두 묵시적 변환 가능
리턴은 **bool* 타입 리턴 ( 바꿀 수 있지만 바람직하지 않음 )

```c++
bool operator==(const SpreadsheetCell& lhs, const SpreadsheetCell& rhs)
{
	return (lhs.getValue() == rhs.getValue());
}
```
getValue는 double을 리턴하는데, 부동소수점 크기 대소 비교는 **입실론 테스트(Epsilon test)를 적용하는게 바람직

구현한 것을 바탕으로 활용도 가능해진다.
```c++
bool operator>=(const SpreadsheetCell& lhs, const SpreadsheetCell& rhs)
{
	return !(lhs < rhs);
}
```

**연산자 오버로딩을 지원하는 타입 정의하기**
중요한 점 = 클래스를 최대한 int나 double같은 기본 타입에 가깝게 정의 하는 것
(사용자들에게 서비스를 제공한다는 생각으로 구현)

스트림 추가(stream insertion) 및 추출 (stream extraction) 연산자 오버로딩하면 편하다
다른 연산자 오버로딩은 추후 설명


### **안정적인 인터페이스 만들기**
추상화 원칙을 고려해 구현과 인터페이스를 최대한 분리하기
ex. 멤버변수 모두 private -> 게터와 세터 이용

**인터페이스 클래스와 구현 클래스**
스프레드시트 구현은 데이터 멤버 및 메서드 전부를 클래스 정의에 작성하여 구현사항이 어느정도 노출되고, 업데이트마다 재컴파일 해야함 (프로젝트가 크면 부담)

**해결책 : 인터페이스 클래스 + 구현 클래스 별도 정의**
구현 클래스 = 일반 클래스
인터페이스 클래스 = public 메서드를 제공하며 구현 클래스 객체에 대한 포인터 데이터 멤버 하나만 가진 클래스

핌플 이디엄(pimple idiom, private implementation idiom) - 핌플구문 또는 브릿지 패턴(bridge pattern)이라 부른다.

인터페이스 클래스
=> 단순히 구현 클래스 객체에 있는 동일한 메서드를 호출하도록 구현
=> 결과 : 구현 코드가 변해도 인터페이스 클래스는 영향 받지 않음
- 컴파일할 일이 줄어듬
주의할 점 : 인터페이스 클래스에 존재하는 유일 데이터 멤버를 구현 클래스에 대한 포인터로 정의해야 함 (포인터가 아닌 값이면 변경될 떄마다 다시 컴파일 해야 함 )

책의 내용은 좀 번잡하고, 실무에서 안쓸 내용이라 스킵함(검색 추천)

내부에 중첩 클래스로 분리하지 않고,
추상 인터페이스로 가상 메서드 형태로 구현해도 된다.(추후 설명)
