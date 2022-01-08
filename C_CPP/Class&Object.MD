클래스와 객체 숙달하기

엑셀의 스프레드 시트 업그레이드 하는 방식으로 클래스 설명

클래스 구현은 보통
선언부, 구현부를 나눔 ( 헤더 + 소스 )

```c++
class SpreadsheetCell
{
public:
	void setValue(double inValue);
	double getValue() const;

private:
	double mValue;
};
```
클래스는 class 키워드와 이름, 내용, 세미콜론; 으로 선언

**클래스의 멤버들**
 - 메서드 (생성자, 소멸자, 일반 멤버함수)
 - 멤버 변수 - 열거형(enum), 타입 앨리어스, 중첩 클래스(클래스 내에 클래스)

get은 const로 선언하는 것이 바람직

각 멤버는 객체 단위로 적용됨 ( static 정적 멤버는 클래스 단위로 적용 )
멤버 함수의 구현 코드는 모든 객체가 공유,
멤버 변수는 각 객체가 개별로 소유

**접근 제어**
접근 제한자/지정자 access control/specifier
 - public
 - private
 - protected

![image](https://user-images.githubusercontent.com/85000118/137660057-d4c584bd-ae6f-4b23-a23f-f190a3bf47bf.png)

class는 접근제한자 지정하지 않으면 private,
struct  는 기본이 public임(C++)

선언에 순서는 따로 없으나, 가독성을 위해 따로따로 묶어서 정리

선언 내부에서 변수 기본 값을 설정 가능함
```c++
private:
   double mValue = 0;
```

**메서드 정의**
헤더파일의 선언부를 소스코드에서 #include로 불러와서 정의함

```c++
void SpreadSheetCell::setValue()
{
}
```

위와 같은 방식으로 소스코드에 정의함
:: 는 스코프 지정연산자로 해당 메서드가 앞의 클래스 또는 네임스페이스에 속함을 알려준다.

Tip : VC++에서 stdafx.h는  VC의 미리컴파일된 헤더로 가장 위에 작성되어야 함 ( 자세한건 MS 문서 참조 )

클래스 내부에서는 멤버 접근에 별도로 지정할 필요가 없음
ex. mValue = inValue;

클래스 메서드는 다른 클래스 메서드를 호출할 수 있음
setValue에서 getValue 호출 가능

엑셀 시트는 텍스트를 숫자로 변환하여 사용하기도 한다.
=> 기능 추가
```c++
#include <string>
#include <string_view>

class SpreadsheetCell
{
public:
	void setValue(double inValue);
	double getValue() const;

	void setString(std::string_view inString);
	std::string getString() const;

private:
	std::string doubleToString(double inValue) const;
	double stringToDouble(std::string_view inString) const;

	double mValue;
};
```
이전 내용 정리
 - C++17에 추가된 stringview const & string을 대신하여 사용함

```c++
#include "SpreadsheetCell.h"

using namespace std;

void SpreadsheetCell::setValue(double inValue)
{
	mValue = inValue;
}

double SpreadsheetCell::getValue() const
{
	return mValue;
}

void SpreadsheetCell::setString(string_view inString)
{
	mValue = stringToDouble(inString);
}

string SpreadsheetCell::getString() const
{
	return doubleToString(mValue);
}

string SpreadsheetCell::doubleToString(double inValue) const
{
	return to_string(inValue);
}

double SpreadsheetCell::stringToDouble(string_view inString) const
{
	return strtod(inString.data(), nullptr);
}
```

private인 헬퍼 메서드 2개 추가 ( 변환 함수 )
=> 헬퍼 메서드를 추가한 것으로, 클라이언트 코드 수정 필요 X

**this 포인터**
멤버를 구분짓기 위해 사용함
ex)
```c++
void SpreadsheetCell::setValue(double value)
{
	value = value; // 이름 중복
// => this 사용 구분
	this->value = value;
}
```
이 외에도 인자를 객체로써 받는 경우, this를 이용하여 넘겨주기도 한다.
```c++
void printCell(const SpreadsheetCell& cell)
{
	cout << cell.getString() << endl;
}

void SpreadsheetCell::setValue(double value)
{
	this->value = value;
	printCell(*this);
}
```

추후에 나올 연산자 오버로딩을 익히면 
cout << *this << endl; 형태로도 출력할 수 있다.

### **객체의 사용법**

**스택에서의 생성**
```c++
int main()
{
	SpreadsheetCell myCell, anotherCell;
	myCell.setValue(6);
	anotherCell.setString("3.2");

	cout << "cell 1: " << myCell.getValue() << endl;
	cout << "cell 2: " << anotherCell.getValue() << endl;

	cout << "cell 1: " << myCell.getString() << endl;
	cout << "cell 2: " << anotherCell.getString() << endl;

	return 0;
}
```

일반 변수 선언 방법과 동일함 ( 타입만 클래스 이름 )
dot 연산자 ( . )을 이용하여 객체에 속한 메서드 호출 및 멤버 접근이 가능
=> 멤버 접근하는 행동은 바람직하지 않음 (Private 적용)


**힙에서의 생성**
new를 이용하여 생성한다.
```c++
int main()
{
	SpreadsheetCell* myCellp = new SpreadsheetCell();

	myCellp->setValue(3.7);
	cout << "cell 1: " << myCellp->getValue() <<
		" " << myCellp->getString() << endl;
	delete myCellp;
	myCellp = nullptr;

	return 0;
}
```
힙에 생성한 객체는 화살표 연산자(->)로 멤버에 접근함 
=> 화살표는 역참조(*)와 멤버 접근 닷 연산자( . )를 합친 것
=> *와 .을 이용해도 되지만, 가독성이 떨어짐

delete를 해줘야하기 때문에 스마트포인터를 이용한다
```c++
int main()
{
	auto myCellp = make_unique<SpreadsheetCell>();
	// 다음과 같이 작성해도 된다.
	// unique_ptr<SpreadsheetCell> myCellp(new SpreadsheetCell());
	
	myCellp->setValue(3.7);
	cout << "cell 1: " << myCellp->getValue() <<
		" " << myCellp->getString() << endl;

	return 0;
}
```

### **객체의 라이프 사이클**
기본은 3 가지 단계 **생성(creation), 소멸(destruction-제거), 대입(assignment-할당)이다.
각각의 시점과 방법 그리고 원하는 방식으로 변경법 이해하기

**객체 생성**
스택 객체는 선언 시점 생성, 스마트 포인터, new 이용할 떄는 공간 할당 후 생성
=> 선언과 동시에 초기값 설정하는 것이 좋음  ( 생성자로 처리 - ctor )

**생성자 작성**
생성자는 리턴 값이 없고, 필요에 따라 매개변수 가능
=> 아무런 인수를 주지 않은 생성자 Default Constructor ( 작성하지 않으면 컴파일 에러 날 수 있음 )

```c++
SpreadsheetCell::SpreadsheetCell(double initialValue)
{
	setValue(initialValue);
}

SpreadsheetCell::SpreadsheetCell(string_view initialValue)
{
	setString(initialValue);
}
```

**생성자 사용법**

```c++
int main()
{
	//스택 객체
	
	SpreadsheetCell myCell(5), anotherCell(4);
	// SpreadsheetCell myCell.SpreadsheetCell(5); 컴파일 에러
	// SpreadsheetCell myCell
	// myCell.SpreadsheetCell(5); 컴파일 에러     선언과 동시 호출, 선언 이후 호출 모두 에러 발생함
	
	
	// 힙 객체
	
	auto smartCellp = make_unique<SpreadsheetCell>(4);
	// 스마트 포인터는 별도로 삭제할 필요가 없다.

	// 스마트 포인터 대신 일반 포인터를 사용할 수도 있지만 권장하지 않는다.
	SpreadsheetCell* myCellp = new SpreadsheetCell(5);
	SpreadsheetCell* anotherCellp = nullptr;
	anotherCellp = new SpreadsheetCell(4);
	// ... do something with the cells
	delete myCellp; 		myCellp = nullptr;
	delete anotherCellp; 	anotherCellp = nullptr;

	SpreadsheetCell aThirdCell("test");  // string 타입 인수를 받는 ctor
	SpreadsheetCell aFourthCell(4.4);    // double 타입 인수를 받는 ctor
	auto aFifthCellp = make_unique<SpreadsheetCell>("5.5"); // string 타입 인수를 받는 ctor
	cout << "aThirdCell: " << aThirdCell.getValue() << endl;
	cout << "aFourthCell: " << aFourthCell.getValue() << endl;
	cout << "aFifthCellp: " << aFifthCellp->getValue() << endl;

	return 0;
}
```

**생성자 여러 개 제공하기**
위에 구현과 사용 설명에 나오긴 했지만, C++은 같은 이름에 다른 매개변수를 갖는 구현 방법 오버로딩을 지원함 ( 추후 설명 )

생성자가 여러개면 하나의 생성자가 다른 생성자 호출 가능
=> 하지만 의도한 대로 실행되지 않을 수 있음(내부에서 호출한 생성자의 임시 객체가 생성되어 원래 초기화 대상인 객체의 생성자가 호출되지 않음)
=> C++에서 제공하는 위임 생성자(delegating constructor)를 이용하면 가능함 (추후 설명)

**디폴트 생성자**
아무런 인수를 받지 않는 생성자 (영인수 - 제로인수 생성자)

**디폴트 생성자가 필요한 경우**
객체 배열을 선언할 때

```c++
SpreadsheetCell cells[3]; // 디폴트 생성자가 없어서 컴파일 에러가 발생한다.
  SpreadsheetCell* myCellp = new SpreadsheetCell[10]; // 여기서도 마찬가지다.

 // 스택 배열은 이니셜라이저 제공하는 방식으로 가능하긴 하다.
  // 이 문장은 디폴트 생성자가 없어도 컴파일 된다.
  SpreadsheetCell cells[3] = {SpreadsheetCell(0), SpreadsheetCell(23),
			      SpreadsheetCell(41)};
```

vector나 배열 등을 사용하려면 디폴트 생성자가 있는 것이 편리하다.
=> 다른 객체에서 생성할 때도 편함 ( 생성자 이니셜라이저 )


**디폴트 생성자 작성 방법**
디폴트 생성자 생성은 그냥 제로 인수로 동일하게 만들면 됨

주의사항 :
SpreadsheetCell myCell(); 가 아닌 SpreadsheetCell myCell; 로 사용해야함

이는 컴파일 에러가 발생하는데, 선언 지점이 아닌 사용지점에서 발생한다.
=> 가장 짜증나는 파싱 문제(most vexing parse)라고 불림

1. 컴파일러가 SpreadsheetCell 를 리턴 타입으로 갖는 myCell 함수를 선언했다고 착각함.
2. 함수 이름을 객체 처럼 사용했다고 착각

힙에서 디폴트 생성자는 동일함
```c++
auto myCellp = make_unique<SpreadsheetCell>();
SpreadsheetCell *myCellp = new SpreadsheetCell();
SpreadsheetCell *myCellp = new SpreadsheetCell;
```
모두 사용 가능


**컴파일러에서 생성한 디폴트 생성자**
디폴트 생성자를 작성하지 않아도, 컴파일 에러가 발생하지 않음
=> 하지만, 생성자가 하나라도 있으면 별도로 만들어주지 않음


**명시적 디폴트 생성자**
C++ 03 이하 버전에서는 비어있는 디폴트 생성자를 반드시 정의 해줬어야 함
이런 수고를 덜기 위하여 **명시적 디폴트 생성자**를 제공해줌
```c++
class SpreadsheetCell
{
public:
	SpreadsheetCell() = default;
	SpreadsheetCell(double initialValue);
	SpreadsheetCell(std::string_view initialValue);

	// 생략
};
```
다른 생성자가 있음에도 default로 선언해줘서 디폴트 생성자가 생성됨

**명시적으로 삭제된 생성자**
C++은 명시적으로 삭제된 생성자 개념 지원
=> static 메서드로만 구성된 클래스를 정의하면 생성자를 만들면 안됨
=> 명시적으로 삭제 하기

= 위의 default 대신 delete를 넣어주면 됨


**생성자 이니셜라이저**
C++은 멤버 변수를 직접 대입하는 식으로 초기화 하는 방법 말고,
생성자 이니셜라이저 (ctor 이니셜라이저 or 멤버 이니셜라이저 리스트)를 제공

```c++
SpreadsheetCell::SpreadsheetCell(double initialValue)
	: mValue(initialValue) , ~~ , ~~
{
}

SpreadsheetCell::SpreadsheetCell(string_view initialValue)
	: mValue(stringToDouble(initialValue))
{
}
```
함수 이름과 본문 사이에 넣는다.
시작은 : 으로 시작하고 ( , ) 쉼표로 구분함
나올 수 있는 항목
 - 함수 호출 형식
 - 유니폼 초기화 문법으로 표현한 데이터 멤버의 이니셜라이저
 - 베이스 클래스 생성자 호출 구문(추후 설명)
 - 위임된 생성자 호출

**생성자 내부에서 초기화 하는 것과 다르다**
C++ 객체 생성은  **구성 멤버 생성** => **생성자 호출** 
=> 이 경우는 이미 객체가 생성된 상태 ( 값을 변경할 뿐 )

**생성자 이니셜라이저를 이용하면 생성 과정에서 초기값 설정**
=> 훨씬 효율적

멤버 변수가 클래스 객체이고, 디폴트 생성자가 정의되어 있다면,
=> 초기화 하지 않아도 됨 = string을 예로 들면, string 디폴트 생성자에서 공백으로 초기화 해줌 => 초기화를 한번 더 하면 2번 하는 셈

B객체 { 멤버변수 A 객체 } 가 있을 때,
=> A 객체에 Default 생성자가 없으면, ( 다른 생성자는 있음 )
=> B::B() { }  구현코드 작성 시, 컴파일 에러가 발생함 ( A객체의 디폴트 생성자가 없어서 )
=> 초기화 하려면 생성자 이니셜라이저 작성 ( 객체 생성하는 시점에 데이터 멤버 초기화 )
=> B::B() : mCell(1.0) { }
=> B객체의 멤버변수인 mCell의 double 생성자 호출

**생성자 이니셜라이저나 or 내부 초기화 타입들**
![image](https://user-images.githubusercontent.com/85000118/137823862-12f33cc5-a330-4078-8274-9ecca5d8967f.png)


**생성자 이니셜라이저 사용 주의점**
생성자 이니셜라이저에서 : x(x2), y(y2) 이런식으로 정의할 때, 쓴 순서대로 초기화 하는 것이 아님!
=> x -> y 순으로 초기화 하는 것이 아니라 클래스 정의의 선언 순서대로 초기화 함
=> 헤더파일에  Y y;  X x; 순으로 초기화 했다면, y먼저 초기화하고 x를 초기화 한다.
=> 혹시라도 x에 값을 입력 시키고, y에 값을 입력 시켰다면 순서문제로 의도치 않은 값이 들어갈 수 있다.

**복제 생성자**
C++ 복제 생성자라는 특수한 생성자를 제공함 (작성하지 않으면 컴파일러가 작성해줌)
=> 다른객체와 똑같은 객체 생성 목적
=> 컴파일러가 작성한 복제 생성자 : 멤버 변수가 기본타입이면 똑같이 복사, 객체면 그 객체의 복제 생성자 호출

일반적인 생성자와 특별히 다를게 없다.
인자를 const & 객체 대상 으로 받아서 생성자 이니셜라이저로 값을 복사한다.
=> 컴파일러에서 모든 멤버를 생성자 이니셜라이저로 초기화 해주므로 굳이 필요는 없음
![image](https://user-images.githubusercontent.com/85000118/137824919-7592acf7-6462-4ae5-8002-dfc63abe84af.png)

이런 방식으로 부족할 떄가 있는데, 추후 설명

**복제 생성자가 호출되는 경우**
C++에서 인자를 전달하면 보통 Pass by Value로 값을 복제하여 전달함
=> 인자로 string을 받는다고 하면, 이를 복제 생성자를 이용하여 초기화 하여 사용함
=> 이러한 복제 오버헤드를 줄이기위해 const & 를 사용함
=> 추가로 리턴값이 객체일때도 복제생성자가 호출됨

**복제 생성자 명시적 호출하기**
그냥 생성자 인자에 객체 넣어주면 됨
ex. AClass A;    AClass B(A);

**레퍼런스로 객체 전달하기**
객체를 레퍼런스로 전달하면 복제 연산 오버헤드를 줄일 수 있음
=> 내용 복사가 아닌 주소 복사라서
=>이 방식을 사용하면 객체의 동적 메모리 할당 관련 문제도 피할 수 있음
=> 단, 값이 변경되면 안된다 = const & 사용

string_view는 포인터와 길이만 가지고 있어, 복제 오버헤드가 적어서 그냥 사용
기본 타입은 const레퍼런스와 값 전달이 큰 차이가 없음

리턴에 객체를 쓸 때는 함수가 끝날 때 그 객체가 삭제되는지 확인해야 함

**명시적으로 만든 디폴트 복제 생성자와 명시적으로 삭제된 복제 생성자**
생성자와 동일하다
AClass(const AClass& src) = default;
AClass(const AClass& src) = delete;
=> 복제할 수 없어짐 => 객체를 값으로 전달 불가(&만 가능)

**이니셜라이저 리스트 생성자**
Initializer-list constructor = std::initilaizer_list<T>를 첫 매개변수로 받고, 다른 매개변수는 없거나 디폴트 값을 가진 매개변수를 추가로 받는 생성자
<initializer_list> 헤더 필요

```c++
#include <iostream>
#include <initializer_list>
#include <stdexcept>
#include <vector>

using namespace std;

class EvenSequence
{
public:
	EvenSequence(initializer_list<double> args)
	{
		if (args.size() % 2 != 0) {
			throw invalid_argument("initializer_list should "
				"contain even number of elements.");
		}
		mSequence.reserve(args.size());
		for (const auto& value : args) {
			mSequence.push_back(value);
		}
	}

	void dump() const
	{
		for (const auto& value : mSequence) {
			cout << value << ", ";
		}
		cout << endl;
	}

private:
	vector<double> mSequence;
};

int main()
{
	EvenSequence p1 = {1.0, 2.0, 3.0, 4.0, 5.0, 6.0};
	// EvenSequence p1{1.0, 2.0, 3.0, 4.0, 5.0, 6.0};  도 가능
	p1.dump();

	try {
		EvenSequence p2 = {1.0, 2.0, 3.0};
	} catch (const invalid_argument& e) {
		cout << e.what() << endl;
	}

	return 0;
}
```
위 코드는 짝수개 원소만 매개변수로 받고, 아니면 익셉션이 발생한다.
size 메서드로 원소 수를 확인,
범위 기반 for문으로 각 원소를 확인 가능함
mSequence.reserve(args.size());부터 push_back 하는 부분은
vector의 assign(args)로 대체 가능하다 ( 4줄 -> 1줄 )


STL은 모두 이니셜라이저 리스트 생성자 지원
ex. vector<string> v = {"string1", "string2", "string3"};


**위임 생성자**
생성자 안에서 동일 클래스의 다른 생성자를 호출할 수 있음
=> 생성자 이니셜라이저에서 호출해야 함 ( 리스트에 위임할 생성자만 적어야 함 )
```c++
SpreadsheetCell::SpreadsheetCell(string_view initialValue)
	: SpreadsheetCell(stringToDouble(initialValue))
{
}
```
위 처럼 생성자 이니셜라이저에서 호출되면 대상에 위임함
=> string view의 생성자가 리턴되면 위임 생성자의 코드가 실행 됨
=> 재귀 호출 되지 않도록 주의 ( 서로가 서로를 위임 생성자로 호출 )


**컴파일 생성하는 생성자에 대한 정리**
컴파일러 -> 디폴트 생성자, 복제 생성자 자동 생성
프로그래밍한 생성자에 따라 달라질 수 있음

![image](https://user-images.githubusercontent.com/85000118/137834859-d690ba4d-d4f8-4b39-ae7f-47e29af918e2.png)

둘은 default, delete를 이용하여 컴파일러의 생성 여부를 결정 지을 수 있음

이동생성자(move constructor)가 있는데, 이동 의미론 구현에 필요함
=> 특정한 상황에서 성능 높이기 위한 목적 ( 추후 설명 )


**객체 소멸**
객체의 제거 과정 2 단계
1. 객체의 소멸자를 호출
2. 할당받은 메모리 반환

소멸자에서 동적 메모리 해제나 파일 핸들 닫기 등을 처리함
없으면 컴파일러가 하나 만들어 줌

스택 객체는 스코프를 벗어날 때 자동으로 삭제
=> 코드 내부에 코드블록이 또 있다면 그 블록을 벗어날 때 삭제 (ex. if문, for문 등 )

스택 객체 삭제 순서는 선언 및 생성 순서와 반대
=> A 선언 후, B를 선언했다면  = B삭제 후, A 삭제
=> 멤버 변수(데이터 멤버) 또한, 선언 순서와 반대로 삭제 됨 ( 선언 순으로 생성자 이니셜라이저 실행되는 것은 앞에서 설명함)


힙 객체는 스마트 포인터는 자동 삭제, 나머지는 직접 삭제(delete)


**객체에 대입하기**
C++에서 복제(Copy)는 객체를 초기화 할때의 표현
값이 할당된 객체 덮어 쓰는 것은 대입(assign)이라 표현함

```c++
SpreadsheetCell myCell(5), anotherCell;
anotherCell = myCell;
```

복제 기능은 복제 생성자에서 제공함(생성에만 사용하고 대입에는 사용 불가 - 생성자라서)

C++은 클래스마다 대입을 수행하는 메서드를 따로 제공
=> 대입연산자(assignment operator) 
=> = 연산자를 오버로딩한 것 (이름 operator=) 
=> 복제 대입 연산자(copy assignment operator)라고도 부름
=> 좌변과 우변에 있는 객체가 대입 후에도 남아 있기 때문
=> 대입 후에 우변의 객체가 삭제되는 이동 대입 연산자(move assignment operator)가 따로 있음 (추후 설명)

직접 정의하지 않아도, 자동으로 만들어 줌 (각각의 디폴트 복제 동작과 거의 동일)


**대입 연산자 선언**
```c++
//선언부
SpreadsheetCell& operator=(const SpreadsheetCell& rhs);
```

복제 생성자처럼 보통 const &를 받을 때가 많음
위에 rhs는 right-hand side(우항)의 줄임말으로,
인자로 받는 것이 a = b 에서 b이다.

리턴으로 객체 레퍼런스을 준 이유
=> A = B = C 처럼 연달아 있을 수 있어서
=> 연산순서는 C에서 B로 대입 후, 리턴 받은 것을 다시 A로 대입

실제로 Operator= 을 간략히 표현한 것에 불과하기 때문에 꼭 리턴을 해줘야 함
=> 값을 복제할 목적이 아니라면 꼭 레퍼런스로 리턴하자

**대입 연산자 정의 방법**
복제 생성자와 비슷하지만, 대입 연산자는 자주 쓸 수 있음
=> 메모리를 동적으로 할당하지 않는 한 큰 차이는 없음 ( 추후 설명 )
=> C++ 객체는 객체에 자기 자신 대입이 가능함 (ex. cell = cell  가능)

따라서, 대입 연산자 구현에는 자기 자신 대입도 고려
=> 시작부분에 if(this == &rhs) 로 확인
=> 리턴 타입이 객체의 레퍼런스이기 때문에 리턴은 return *this;

다른 경우에, 각종 멤버에 대입을 수행하고, 동일하게 return *this;
```c++
SpreadsheetCell& SpreadsheetCell::operator=(SpreadsheetCell& rhs)
{
	if (this == &rhs)
		return *this;
	
	mValue = rhs.mValue;
	return *this;
}
```
이러한 대입 연산자는 단순 대입밖에 없어서 컴파일러가 생성해주는 것만으로도 충분하다.
특수한 케이스는 다음 Class 내용에서 다룸

**명시적으로 디폴트 or 삭제 대입 연산자**
기존 생성자들 처럼
SpreadsheetCell& SpreadsheetCell::operator=(SpreadsheetCell& rhs) = default or delete 하면 된다.


**컴파일러가 만들어주는 복제 생성자 및 복제 대입 연산자**
C++11부터 클래스에 
사용자가 선언한 복제 대입 연산자 or 소멸자 있으면 복제 생성자를 생성 기능 미지원
=> 사용하고 싶다면 MyClass(const MyClass& src) = default; 로 명시한다.

복제 생성자 or 소멸자 = 복제 대입 연산자 생성 미지원
=> 동일하게 MyClass& operator=(const MyClass& rhs) = default;

즉, 복제 생성자나 복제대입 연산자 둘 중 하나라도 선언하면 다른 것도 처리해줘야 함

**복제와 대입 구분**
선언처럼 생겼다면 복제 생성자
대입문처럼 생겼다면 대입연산자

```c++
SpreadsheetCell A = B;
// 위는 operator= (복제 대입 연산자)는 호출되지 않음
// SpreadsheetCell A(B)의 다른표현이라고 봄

A = B;
// 위는 이미 생성되었기 때문에 operator= 호출
```



**리턴값이 객체인 경우**
함수나 메서드에서 객체를 리턴받을 때, 복제가 될지 대입이 될지 해깔릴 수 있다.
예를 들어, getString 함수로 임시 string 값을 반환 받는 함수가 있다.
```c++
string s1;
s1 = myCell.getString();
// 이 경우,  복제 대입 연산자가 수행됨 
// 오버헤드가 크다면 **리턴값 최적화**, **복제 생략**을 적용하여 최적화

string s2 = myCell.getString();
// 이 경우에도  임시 string 객체를 리턴하지만 복제 생성자가 호출됨
```
이동 의미론에 따르면 string을 리턴할 때, 컴파일러는 복제 생성자 대신 **이동 생성자**를 사용한다. ( 추후 설명 )

잘 모르겠으면 디버깅 해보는게 좋음


**복제 생성자와 객체 멤버**
객체가 다른 객체를 가지고 있다면,
=> 컴파일러가 만들어준 복제 생성자는 해당 객체들의 복제 생성자를 재귀적으로 호출

복제 생성자 직접 정의하였고, 생성자 이니셜라이저를 이용해 직접 구현한다면
=> 생성자 이니셜라이저에서 생략한 멤버는 컴파일러가 그 멤버에 대한 디폴트 생성자를 호출해서 초기화 작업 처리해줌 ( 본문 실행할 시점에는 데이터 멤버 모두 초기화 상태 )

```c++
MyClass::MyClass(const MyClass& src)
  : mValue(src.mValue)
{
// 위의 생성자 이니셜라이저는 복제 생성자 사용
// 안쓴 것은 컴파일러가 디폴트 생성자
    mValue = src.mValue;
// 이건 복제가 아닌 대입  연산자
}


``
