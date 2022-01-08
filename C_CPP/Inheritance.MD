코드 재사용 및 확장

### **상속을 이용한 클래스 구현**

원본 클래스를 그대로 가져다 사용하는 경우
=> OOP 프로그래머 입장에서 성가신일 가능성
 - 원본 버그 수정 코드 반영안됨 ( 2개가 독립적 )
 - 컴파일러는 원본과의 관계를 모름 ( 다형성 구현 불가 )
 - 진정한 is-a 관계가 아님 ( 타입이 같은 것이 아님 - 가져다 썼을 뿐 )
 - 소스코드를 복사할 수 없을 수도 있음

**클래스 확장**
클래스 정의할 때, 컴파일러에 상속(inherit), 파생(derive), 확장(extend)한다고 선언할 수 있다.

 - 기존 클래스의 멤버들을 자동으로 가져옴
=> 원본 = 부모클래스(베이스, 슈퍼 클래스)
=> 확장 대상 = 자식 클래스(파생, 서브 클래스)

자식 클래스는 부모와 다른점만 구현하면 됨

```c++
class Base
{
// 이런저런 메서드, 멤버
}

class Derived : public Base
{
// 추가할 메서드
}
```

다른 클래스를 위 처럼 상속받아 필요한 부분만 추가 구현이 가능함


**클라이언트 입장의 상속**

Derived는 Base의 public 멤버 + Derived 멤버 사용이 가능하다.

상속은 단방향으로만 진행
=> Base는 Derived 존재를 모름, 따라서 메서드 사용 불가능
=> Derived = Base 타입   , but,  Base != Derived
**하위 클래스는 하위 클래스 타입 뿐 아니라, 베이스 클래스 타입이다.**

객체를 레퍼런스나 포인터로 가리킬 때, 파생 클래스도 가리킬 수 있음
```c++
Base* base = Derived(); //Derived 객체를 Base포인터에 저장

// 하지만, Derived의 멤버는 사용불가
base->derivedMethod();  // 오류
```

파생 클래스의 이용가능 내역
 - 부모의 Public
 - 부모의 Protected
 - 자신의 멤버
ex. 자신의 메서드안에서 부모의 데이터 멤버 이용 가능

=> Private는 오직 자신만, public은 모두가, protected 상속 관계만

=> 멤버는 기본적으로 private가 바람직 ( 캡슐화 수준을 최대로 끌어올리기 위함 )
=> 게터나 세터를 public으로 지정
=> 파생클래스 사용 목적은 protected

=> 위 처럼 구현하여, 인터페이스는 public, Protected로 내부 데이터를 마음껏 변경 가능
=> 멤버를 게터나 세터로하면 검사 작업 처리도 쉬워짐


**상속 방지**
final 키워드를 클래스 선언 뒤에 붙이면 다른 클래스가 상속이 불가능 해짐
```c++
class Base final
{
};
class Derived : public Base
{
 // 에러 발생
}; 
```
상속 받으면 컴파일 에러 발생


### **메서드 오버라이딩**
상속받은 메서드의 동작 변경을 위함 Method Overriding

**메서드 오버라이딩과 virtual 속성**
오버라이딩을 하려면 베이스 클래스에서 virtual 선언된 메서드만 가능(선언부)
```c++
class Base
{
public:
	virtual void canOverride();
	void cantOverride();
};
```
일반적으로 모든 메서드를 virtual로 하는 것이 바람직
=> 장점 : 오버라이드 과정에서 메서드 제대로 동작할 지 신경 안써도 됨
=> 단점 : 성능이 좀 떨어짐 (주의점은 뒤에)

**컴파일러가 자동 생성하는 소멸자는 virtual로 선언되지 않음**


**메서드 오버라이딩 문법**
헤더의 메서드 선언부에 함수들 뒤에 override 붙이기
```c++
class Derived : public Base
{
public:
    virtual void canOverride() override; // 적지 않아도 되긴 함
 //void canOverride(); // 이것도 오버라이드
};
```
되도록 override를 적어서 구분하는 것이 좋다. (이유는 뒤에)

부모의 virtual 메서드 & 소멸자
=> 자식은 virtual 기입 안해도 override 가능함


**클라이언트 관점의 오버라이드한 메서드**
오버라이드한 메서드는 각 객체에 해당하느 메서드 호출

```c++
Base base1;
base1.canOverride(); // Base의 함수

Derived derived1;
derived1.canOVerride(); // Derived의 함수
```
오버라이드 하지 않았다면, 둘다 Base함수

**레퍼런스나 포인터로 선언하면 파생클래스를 가리킬 수 있다.**
=> 해당하는 메서드를 호출 가능
```c++
Derived myDerived;
Base& ref = myDerived;
ref.canOverride(); // Derived의 메서드
ref.derivedMethod(); // Derived의 메서드 호출 시도 = 컴파일 오류
```
베이스 클래스에 virtual이 안적혀있다면, Base의 메서드가 호출됨

베이스 클래스 타입의 레퍼런스는 자신의 클래스에 정의되지 않은, 메서드를 호출할 수는 없다.

Derived 객체 => (캐스팅 or 대입) => Base 클래스
=> 가능은 하지만, Derived의 정보는 사라짐

Base 레퍼런스 or 포인터  => Derived 객체
=> 메모리는 그대로이며, 접근법만 달라짐 ( Derived 메서드 사용은 불가지만, 메모리 차지 )
Derived (캐스팅) => Base  
=> 메모리 영역 축소 Derived 정보 사라짐 ( 삭제되는 것을 슬라이싱(slicing)이라 함 )

**Override 키워드**
정의 뒤에 override를 쓰는 이유
=> 실수를 방지하기 위함
ex. 베이스 클래스에서 virtual 함수 인자를 수정하였을 때,
=> Base abc(int a)  => abc(double a);  
파생 클래스에서 override를 적지 않으면, 새로운 메서드 정의로 보고 처리 됨
(Override 쓰면 컴파일 오류 처리)


**virtual 메서드**
virtual 안한 메서드 오버라이드하면 문제 생김 ( 항상 virtual 선언이 좋음 )

**오버라이드하지 않고 숨기기**
Base에서 virtual 안쓰고 파생 클래스에서 그 메서드를 정의하면,
=> 오버라이드가 아닌 새로운 함수를 선언한 것

```c++
Derived myDerived;
Base& ref = myDerived;
ref.go();
```
Base의 go가 호출되는데, 오버라이드 됐다면 Derived의 함수가 출력되어야 정상
=> virtual 안한 오버라이드 함수는 베이스에서는 숨겨버림 ( 파생클래스에서만 사용 가능 )

**virtual 메서드의 내부 작동 방식**
C++ 클래스 컴파일하면
=> 클래스의 모든 메서드를 담은 바이너르 객체 생성
but, virtual로 선언되지 않은 메서드를 호출하는 부분
=> 컴파일 시간에 결정된 타입의 코드로 교체 ( 위 예제에서 Derived -> Base로 교체 )
 **정적 바인딩(static binding) or 이른 바인딩(early binding) 이라 함**

virtual로 선언하면 **vtable(가상테이블)** 를 이용 ( virtual function table 이라고도 함)
=> 특수한 메모리 영역 활용 가장 적합한 구현 코드 호출

virtual 메서드 하나 이상 => vtable 하나
인스턴스는 vtable에 대한 포인터를 갖게 됨 ( 안에 virtual 메서드 구현 코드 포인터 담김 )
따라서, 객체 메서드 호출 시, vtable 보고 적합한 버전의 메서드 실행
=> **동적 바인딩(dynamic binding) or 늦은 바인딩(late binding)**

![image](https://user-images.githubusercontent.com/85000118/138585422-26ee04f2-4fba-45ab-a898-4400327da249.png)

조사식을 보면 virtual한 함수들에 대한 table을 볼 수 있다.
b의 오버라이드한 함수는 Derived, 아닌 것은 Base를 가리킨다.
(virtual 선언안하면 vtable에 안보임)


**virtual 키워드가 필요한 이유**
모든 메서드를 virtual로 처리해야 한다는 사람이 많다. ( 자바는 실제로 그렇게 함 )
안된다는 사람도 있음 ( vtable 오버헤드 줄이기위해 만든 것이 virtual 키워드기 때문 )
=> virtual 메서드 호출에 적합한 코드 포인터를 역참조해야하는데 , 과거에는 이러한 오버헤드도 거슬렸었음
=> 따라서, virtual 여부에 따라 성능이 많이 떨어지는 App은 거의 없음
=> 결론 = 최신 성능 기준으로 모든 메서드, 소멸자는 virtual 선언하자

그러나 극단적으로 수백 수십억 개의 virtual 포인터를 저장하면 엄청난 오버헤드가 발생
+
객체마다 vtable 포인터를 가져야하여 메모리 공간 더 차지함

이를 고려하여 virtual 키워드 활용


**virtual 소멸자의 필요성**
소멸자의 virtual는 이견없이 모두 필요하다고 함 (메모리 잔류)
=> final 선언이 아닌 이상 항상 virtual이 좋음
ex. unique_ptr같은 자동 삭제 멤버는 소멸자가 호출되지 않으면 삭제되지 않고 남음


```c++
Base* ptr = new Derived();

delete ptr;
// Base의 소멸자 호출 (~Base)
// ~Derived는 호출되지 않음 => 소멸자를 virtual로 선언하지 않아서

//따로 처리 할일 없다면 Base 소멸자는 아래처럼
virtual ~Base() = default;
```
전에 설명처럼  C++11부터
사용자 선언 소멸자가 있으면,
=> 복제 생성자, 복제 대입 생성자 자동 생성 안함
=> 필요하다면 명시적으로 생성

**특별한 이유가 없거나 final로 선언한게 아니라면 소멸자를 포함, 모든 메서드를 virtual로 선언  ( 생성자는 그럴 수도 그럴 필요도 없음 )**


**오버라이딩 방지하기**
메서드 또한 final로 수정을 막을 수 있음
```c++
class Base
{
virtual void someMethod() final;
};
```
오버라이드 하려하면 컴파일 에러


### **코드 재사용을 위한 상속**
설명을 위한 간단한 일기예보 프로그램 제작(섭씨 화씨 모두 사용)
 - 서드파티 클래스 라이브러리 이용 ( 저작권으로 바이너리 형태 배포 )
 클래스 정의는 제공한다고 가정하고, 아래와 같다.

```c++
// 현재 온도, 목성과 화성 사이의 거리를 기반으로 날씨를 예측하는 검증된 최첨단 기술을 구현한다.
// 이런 값이 제공되지 않아도 날씨를 예측할 수 있지만 정확도는 99%에 불과하다.
class WeatherPrediction
{
public:
	// Virtual 소멸자
	virtual ~WeatherPrediction();

	// 현재 온도를 화씨 단위로 설정한다.
	virtual void setCurrentTempFahrenheit(int temp);

	// 목성과 화성 사이의 현재 거리를 설정한다.
	virtual void setPositionOfJupiter(int distanceFromMars);
	
	// 내일 온도에 대한 예측값을 가져온다.
	virtual int getTomorrowTempFahrenheit() const;
	
	// 내일 비가 올 확률을 가져온다. 값이 1이면 확실히 비가 오고, 
    // 0이면 비가 오지 않는다는 것을 의미한다.
	virtual double getChanceOfRain() const;
	
	// 사용자에게 다음과 같은 포맷으로 결과를 출력한다:
	// Result: x.xx chance. Temp. xx
	virtual void showResult() const;
	
	// 현재 온도를 스트링값으로 리턴한다.
	virtual std::string getTemperature() const;

private:
	int mCurrentTempFahrenheit;
	int mDistanceFromMars;
};
```

모두 virtual로 선언되어, 오버라이드한다고 가정
이를 아래 이유로 수정해야한다.
1. 온도 값이 모두 화씨 단위(섭씨 필)
2. showResult 메서드 출력 결과의 형식


**파생클래스에 기능 추가**
위의 클래스를 상속받아 사용할 클래스 새로 정의
```c++
#include "WeatherPrediction.h"
class MyWeatherPrediction : pubic WeatherPrediction
{
};
```
이 상태 만으로도 MyWeatherPrediction를 대신하여 사용 가능 ( 똑같고 차이 없음 )

섭씨를 추가하기 위한 방법 중 한 가지 방법은
=> 베이스와 파생 클래스 사이에 중계하는 인터페이스 추가

중간에서 화씨 섭씨 변환하는 헬퍼메서드를 추가하여 섭씨를 받는 메서드를 추가
```c++
class MyWeatherPrediction : public WeatherPrediction
{
public:
	virtual void setCurrentTempCelsius(int temp);
	virtual int getTomorrowTempCelsius() const;
private:
	static int convertCelsiusToFahrenheit(int celsius);
	static int convertFahrenheitToCelsius(int fahrenheit);
};
```
모든 객체가 동일한 기준을 갖기 때문에 static 사용
그리고 보통 부모 클래스의 명명 규칙을 그대로 따라, 작성한다. (일관성 유지)

위 처럼
1. set 함수를 호출하면
2. get메서드로 부모 클래스로 화씨를 받고,
3. 내부에서 변환하여 사용
위 처럼 중간에서 값을 주고 받아 처리하는 인터페이스 형태로 구현 가능


**파생 클래스에서 기존 기능 변경**
showResult를 원하는 형태로 바꾸기 위해 override를 이용

```c++
virtual void showResult() const override; // 헤더에 정의

//소스 삽입
void MyWeatherPrediction::showResult() const
{
	cout << "Tomorrow's temperature will be " <<
		getTomorrowTempCelsius() << " degrees Celsius (" <<
		getTomorrowTempFahrenheit() << " degrees Fahrenheit)" << endl;

	cout << "The chance of rain is " << (getChanceOfRain() * 100) << " percent"
		<< endl;

	if (getChanceOfRain() > 0.5) {
		cout << "Bring an umbrella!" << endl;
	}
}
```
showResult 대신 원하는 형태로 출력할 수 있도록
=> 기존 것을 덮어버리고 새롭게 쓰는 override


### **부모를 공경하라**
파생 클래스 작성에 부모 <-> 자식 연동 방식 주의
1. 생성순서
2. 생성자 체이닝
3. 캐스팅
등 여러 버그가 발생 가능함


**부모 클래스의 생성자**
객체는 한번에 생성 안됨, 
=> 부모 것 + 추가한 것 모두 담아 생성

C++의 객체 생성 과정 정의
1. 베이스 클래스면 디폴트 생성자 실행, 생성자 이니셜라이저가 있으면 이니셜라이저 먼저 호출
2. static이 아닌 데이터 멤버를 코드 순서로 생성
3. 클래스 생성자 본문 실행

위를 재귀적으로 적용
=> 부모가 있으면 가장 위 부모부터 현재 클래스까지 순차 초기화
정확하게 위 3가지 순서로 출력된다.
```c++
class Something
{
public:
	Something() { cout << "2"; }
// 5. 동일하게 클래스 생성 과정 수행
	virtual ~Something() { cout << "2"; }
};

class Base
{
public:
	Base() { cout << "1"; }  
// 2. 상위 클래스 없고, 디폴트 수행자가 있으므로 수행 
	virtual ~Base() { cout << "1"; }
};

class Derived : public Base
{
public:
	Derived() { cout << "3"; }  
// 1. 실행 전 부모 클래스 확인
// 3. 부모 생성자 수행하고, 각 클래스 멤버들 구성
// 6. 마지막으로 Derived 생성자 내부 수행함
	virtual ~Derived() { cout << "3"; }

private:
	Something mDataMember;
// 4. 멤버 구성 과정에서 생성자 호출됨
};

int main()
{
	Derived myDerived;  // 위의 Derived 클래스로 진입
	return 0;
}
```

위 순서처럼 1-2-3 이 출력된다.

생성자의 연쇄적인 호출을 체인으로 엮을 수 있음
=> 부모의 디폴트 생성자가 없으면, 컴파일 에러가 발생함
```c++
Derived::Derived(int i) : Base(i)
{
}
```
위처럼  값 i를 Base 생성자에 전달하며 처리하여도 된다.
=> 단, 멤버는 부모 생성자 이후에 초기화 되기 떄문에, 전달이 불가함

생성자 내부에서는 virtual 메서드의 동작이 다름
=>  파생 오버라이드 함수가 아닌 Base의 메서드가 호출됨


**부모 클래스의 소멸자**
생성자와 반대 순서
1. 현재 클래스 소멸자 호출
2. 데이터 멤버 생성과는 반대로 삭제
3. 부모가 있으면 부모 소멸자 호출

재귀적으로 동작 

생성자 코드에서 virtual 소멸자도 선언되어 있는데, 
=> 정상적인 호출은 123321 순서
=> virtual 없애고 베이스 클래스 포인터에 delete를 실행하면 실행순서가 뒤 바뀜
```c++
Base* ptr = new Derived();
delete ptr;
```
=> virtual 삭제하면 1231이 나옴

**virtual 지정하지 않아, Base의 소멸자만 호출 됨**
=> 생성자 앞에 virtual만 해도 해결하지만, 모든 소멸자는 virtual로 하여 방지하는 것이 바람직

컴파일러 생성 소멸자는 virtual 아님 
=> virtual 소멸자 정의나 default 지정 필요

virtual 소멸자 또한, 오버라이드한 내부 메서드 사용은 Base 메서드를 호출


**부모 클래스 참조하기**
오버라이드를 하여도 부모 버전을 얼마든지 실행할 수 있음
=> 조심할 사항 스코프가 기본적으로 현재 클래스이기에 오버라이드한 메서드에서  부모 클래스를 부를 떄 ::를 사용해야함
```c++
string MyWeatherPrediction::getTemperature() const
{
return getTemperature() + "\u00B0F";
}
```
위의 경우 무한 재귀 호출이 됨
앞에 베이스 클래스 WeatherPrediction::을 붙여줘야함

**VC++에서 제공하는 __super(언더스코어 2개) 키워드를 사용하여 작성해도 됨**
__super::~~~

 부모 메서드에서 특정 요건을 추가하거나, 그 함수를 토대로 작성할 때
부모 메서드를 가져다 쓰면 편리하다.
=> 2단 상속을 하였는데, 2번째에서 override를 하지 않았으면, 타고 올라가서 그 위의 메서드를 수행함


**업캐스팅과 다운캐스팅**
객체는 부모 클래스 타입으로 캐스팅 or 대입이 가능함
=> 이 경우, 자식 클래스 특성이 사라지는 슬라이싱 발생 ( 독립적인 메서드 삭제 )

그러나, 포인터나 레퍼런스 대입은 슬라이싱 발생 안함
```c++
Base& myBase = myDerived; // 슬라이싱 발생 안함
```

위와 같이 베이스 = 파생 참조하는 경우를 **업캐스팅(upcasting)** 이라 함
=> 이렇게 슬라이싱 없이 파생클래스 전달을 위해 레퍼런스를 활용함

반대인 경우는 **다운캐스팅(downcasting)**이라 함
=> 객체가 파생 클래스에 속한다고 보장할 수 없고,
=> 다운캐스팅 있단 것 자체가 디자인이 잘못된 것을 의미

```c++
void Presumptuous(Base* base)
{
   Derived* myDerived = static_cast<Derived*>(base);
   // 기타 데이터 다루는 작업
}
```
위 경우, 함수 제작자는 Derived타입을 넘겨야한단 사실을 알고 사용하지만,
다른 사용자가 Base를 넘겨야한다 생각하고 넘긴다면 문제 발생 가능성이 있음

**다운캐스팅이 필요할 때도 있지만, 완벽히 통제된 상황에서만**
=> 반드시 dynamic_cast() 호출
( 객체 정보에 캐스팅이 잘못됐다면 처리하지 않음 => 타입정보는 vtable 참조 )
=> vtable참조하기 떄문에 virtual 멤버가 하나 이상이어야  사용 가능

dynamic_cast가 실패하면 nullptr가 됨
=> 추가로, std::bad_cast 익셉션 발생함
```c++
// 앞의 예제 수정
Derived* myDerived = dynamic_cast<Derived*>(base);
if(myDerived != nullptr){
// 기타 소스코드
}
```

디자인이 잘못되면 다운캐스팅 코드가 나타날 때가 있음
=> 다시 디자인하는게 좋다.
ex. 위 함수는 Derived 객체만 다루니 Base가 아닌 Derived* 만 받도록 수정

Base를 상속한 다른 파생 클래스를 사용해야 한다면 다형성을 활용하자


### **다형성을 위한 상속**
상속의 가장 강력한 기능

**스프레드시트 예제**
다른 장에서 설명했던 스프레드 시트에서는 double과 string 2가지 타입만 내부적으로 다뤘음
=> 수식, 날짜 등 다양한 타입을 다루기 위해서는??

**다형성을 지원하는 스프레드시트 셀 디자인하기**
구현한 스프레드 시트셀은 비대해서 계층화 할 필요가 있음
=> 한 가지 타입만 담을 수 있도록 축소
ex. StringSpreadsheetCell, DoubleSpreadsheetCell

공통 분모를 기준으로 상속을 디자인하여 아래처럼 변경하였다.
![image](https://user-images.githubusercontent.com/85000118/138864290-2cbe2b8e-7d5c-4886-adc1-e9ee831b73a3.png)

반드시 필요한 객체의 서로 is-a 관계로 생각하지말고,
공통부모를 가진 관계도 고려

위 그림은 계층 구조를 다형성을 활용하도록 디자인함
=> 둘 모두 같은 부몰를 상속하여 얼마든지 교체항여 사용 가능함

디자인 특성
 - 두 파생 클래스 모두 베이스 클래스에서 정의한 인터페이스를 똑같이 제공
 - 현재 셀 타입을 신경 쓸 필요 없이 상위 인터페이스만 맘껏 사용
 - virtual 메커니즘을 통해 인터페이스의 메서드 중에서 가장 적합한 버전 호출
 - 부모를 참조하는 방식으로 여러 타입으로 된 셀을 하나로 묶을 수 있음

**스프레드시트셀 베이스 클래스**
공통된 기능들을 모아 정의하는 베이스 클래스를 제작

**첫번째 버전**
공통된 동작을 정의해야함
=> 파생 클래스 구현 전에, 동작을 베이스에서 선언할 방법? ( 멤버가 현재 없음 ex. String을 쓸지 double을 쓸지 모름 )

1. 아무일도 하지 않도록 구현하는 것
2. 인스턴스를 만들 수 없게하는 것


**순수 가상 메서드와 추상 베이스 클래스**

순수 가상 메서드(pure virtual method)
=> 정의 코드에서 명시적으로 정의하지 않는 메서드
추상 클래스(abstract class)
=> 순수 가상 메서드가 최소 하나라도 정의된 클래스

추상 클래스는 인스턴스를 만들 수 없음
=> 구현방법 : 메서드 선언 뒤 = 0을 붙임 + 구현 코드 작성 X

```c++
class SpreadsheetCell
{
virtual ~SpreadsheetCell() = default;
virtual void set(std::string_view inString) = 0;
virtual std::string getString() const = 0;
};

SpreadsheetCell cell; // 추상 클래스 인스턴스 생성 => 컴파일 에러

// 파생 클래스 타입으로 생성하여 가리키는 것은 가능해진다.
std::unique_ptr<SpreadsheetCell> cell(new StringSpreadsheetCell());
```

**핵심**
1. 추상클래스는 인스턴스 생성이 불가능하다.
2. 추상클래스는 소스파일 작성할 필요가 없다.


**파생클래스 구현하기**
파생클래스는 객체를 생성해야 한다.
=> 상속받은 순수가상메서드를 모두 구현해야함
=> 모두 구현하지 않으면, 이 또한 추상 클래스

파생클래스는 각 멤버를 추가하는데, (private)
C++17에 추가된 std::optional을 사용한다  <optional> 헤더

optional은 클래스 템플릿 (추후 자세한 설명)
=> optional<string>같이 구체적인 타입 지정 필요
=> 셀에 값이 설정됐는지 쉽게 알아낼 수 있음

optional을 이용하여 내부 함수 구현할 때, 실제 값이 있는지 여부를 체크 할 수 있다.
ex. getString메서드에서 return mValue.value_or("");  식의 사용이 가능

string도 그냥 구현하면 되고,
double도 optional<double>에 static private로 내부 전용 변환 메서드를 만드는 등의 개별 구현을 하면 된다.

optional 기능 중 하나인 mValue.has_value로 값을 확인하고 변환하는 등의 작업을 할 수도 있다.
ex. return (mValue.has_value() ? doubleToString(mValue.value()) : "");

이런식으로 계층화하면 코드가 훨씬 간결해진다. 각 타입의 기능만 집중 구현한다.


**다형성 최대로 활용하기**
활용법을 보이기 위하여 vector를 이용한다.
```c++
// 객체 생성 전 원소 타입으로 베이스 클래스 지정
vecctor<unique_ptr<SpreadsheetCell>> cellArray;

// 여러 타입의 객체 인스턴스를 생성
cellArray.push_back(make_unique<StringSpreadsheetCell>());
cellArray.push_back(make_unique<StringSpreadsheetCell>());
cellArray.push_back(make_unique<DoubleSpreadsheetCell>());

cellArray[0]->set("hello");
cellArray[1]->set("10");
cellArray[2]->set("18");
```

위 같은 사용은 컴파일 시간에 어떤 객체인지 알 수 없다.
다만 런타임에는 가리키는 객체에 대해 메서드를 수행할 수 있다.
=> SpreadsheetCell에 정의된 메서드는 확실히 사용을 보장함
=> 독립적으로 구현된 파생 클래스 메서드는 사용 불가 ( 베이스 클래스 타입이라서 )


**나중에 대비하기**
계층화하여 객체지향 디자인 관점에서 많이 나아졌지만, 셀 타입 상호변환 기능이 빠져있다.
=> 변환생성자(converting constructor) = 타입 생성자(typed constructor)
을 추가하여 구현

복제 생성자와 비슷하지만, 형제 클래스를 인수로 받음
=> 디폴트 생성자 반드시 선언 필요 or default ( 컴파일러가 자동으로 안만들어줌 )

 ```c++
StringSpreadsheetCell(const DoubleSpreadsheetCell& inDoubleCell);

StringSpreadsheetCell::StringSpreadsheetCell(const DoubleSpreadsheetCell& inDoubleCell)
{
mValue = inDoubleCell.getString();
}
```
주의 : 포인터 or 레퍼런스 캐스팅때와는 방식이 다름
=> cast 연산자를 오버로드 하지 않으면 포인터나 레퍼런스 타입의 형제 클래스끼리 캐스팅 불가

cast 연산자 동작 변경 or reinterpret_cast()를 이용하면 계층을 건너뛰면서 캐스팅 가능 = but, 바람직하지 않음


생성자와 더불어 두번째로 셀마다 연산자를 오버로드 해야 함
방식은 여러가지
1. 모든 셀 타입 조합에 대해 연산자 일일히 구현하기
2. 공통 표현 방식의 규정
=> 앞의 예제는 double도 string도 결국 getstring을 통해 string이라는 공통 표현 방식이 있다.

 따라서, operator+를 이용하여 모든 조합의 연산이 가능하다.
(doube + string , double + double, string + string)
```c++
StringSpreadsheetCell operator+(const StringSpreadsheetCell& lhs,
	const StringSpreadsheetCell& rhs)
{
	StringSpreadsheetCell newCell;
	newCell.set(lhs.getString() + rhs.getString());
	return newCell;
}


// 모두 String 타입으로 변환되게 만들어 결과 얻기
DoubleSpreadsheetCell myDbl;
myDbl.set(8.4);
StringSpreadsheetCell result = myDbl + myDbl;
```

### **다중 상속**
좋다 나쁘다 논란이 많음 

**여러 클래스 상속하기**
```c++
class Baz : pulbic Foo, public Bar
{
};
```

동시 상속 시, 속성
 - Baz = Foo, Bar 멤버 + public 메서드
 - Baz = Foo, Bar Protected 멤버, 메서드 접근
 - Baz => Foo or Bar 업캐스팅
 - Baz 생성 => Foo, Bar 디폴트 생성자 호출 - 호출 순서 : 첫줄에 정의한 순서
 - Baz 삭제 => Foo, Bar 소멸자 자동 호출 - 호출 순서 : 생성자와 반대

예를 들어 Dog와 Bird를 상속받는 DogBird
=> 각자의 public 메서드를 제공 받음

**이름 충돌과 모호한 베이스 클래스**
다중 상속 문제 사례

**1.모호한 이름**
Dog와 Bird 모두 eat() 메서드를 가지고 있다면,
=> 컴파일러가 어떤 것을 호출해야 하는지 몰라 에러 발생

해결방법으로 4가지가 있음
1. dynamic_cast<Dog&>(myConfusedAnimal).eat();
=> 업캐스팅으로 특정 상위버전 객체 확인
2. myConfusedAnimal.Dog::eat();
=> 스코프 지정 연산자 사용
3. 오버라이드하여 독립적인 eat() 함수 생성
4. 클래스 내에서 using을 이용하여 버전 지정
```c++
class DogBird : public Dog, public Bird
{
using Dog::eat;
};
```

**2.모호한 베이스 클래스**
Bird가 Dog를 상속받고,
DogBird가 둘을 상속받으면 => 에러 발생
=> 베이스 클래스가 모호하기 떄문에 계층이 정리 안됨
![image](https://user-images.githubusercontent.com/85000118/139165073-b70df575-1748-4319-a330-c6077f5aa84f.png)

데이터 멤버가 겹칠 때 도 모호함
부모가 겹친다면??
![image](https://user-images.githubusercontent.com/85000118/139165162-1777df57-80fc-420c-aed2-95610cf272e5.png)
=> Animal의 메서드를 호출한다면, Dog인지 Bird인지 어디 메서드인지 구분 불가
=> 해결법은 최상단을 순수 가상 메서드 구성  추상 클래스로 구현
(베이스 클래스에서 호출할 것이 없음)
=> 하지만 근본적으로 메서드 이름들이 같아, 앞의 해결책을 또 사용해야 함
=> 이런 형태는 최상단을 가상 베이스 클래스(virtual base class)로 구성 (추후 설명)


**3.다중 상속 활용법**
왜 다중 상속을 쓰는가?
=> 가장 간단한 예 : is-a관계 맺는 대상이 하나 이상인 객체 정의
=> 가장 적합 : 믹스인 클래스
+
=> 컴포넌트 기반으로 클래스 모델링할 떄
ex. 비행기 부품 상위 ~~ 있다. has-a 관계가 복잡해져 좋지 않다.
 = AirPlane(Engine, Fuselage, Controls ... ) 데이터 멤버 정의가 좋다.


### **상속에 관련된 미묘하면서 흥미로운 문제들**
다양한 이슈를 고려해야 함
 - 클래스 속성 변경 가능한 것은 무엇인가
 - non-public 멤버는 어떻게 되는가
 - 가상 베이스 클래스가 무엇인가

**오버라이드한 메서드의 속성 변경하기**
오버라이드 과정에서 메서드 속성 변경하고 싶을 때?

**1. 리턴 타입 변경하기**
베이스 클래스의 메서드 프로토타입(method prototype)과 똑같이 작성하는 것이 원칙이지만,
=> 변경할 수도 있다.
ex. 베이스 리턴 타입이 다른 클래스 포인터or레퍼런스 타입이면 오버라이드할 때 그 클래스의 파생클래스에 대한 포인터or레퍼런스로 변경 가능
=> 이런 타입 = 공변 리턴 타입(convariant return type)이라 함

베이스 클래스 <-> 파생 클래스가 병렬 계층(parallel hierarchy)를 이룰 때, 두 계층이 따로 존재하지만, 어느 한쪽에 관련 있을 때 이 기능이 유용

예를 들어, 체리 과수원의 객체
![image](https://user-images.githubusercontent.com/85000118/139185643-532cbadd-fe9e-45f0-a73f-a4dce3943c39.png)

이렇게 있을 때,
![image](https://user-images.githubusercontent.com/85000118/139185866-f235229e-2006-4d35-bbab-bc24f33f820f.png)
=> 체리 나무로 부터 따는 메서드를 하위 클래스에서 오버라이드 할 때
![image](https://user-images.githubusercontent.com/85000118/139185909-e2d89451-1f7d-41ac-b06a-1d8490fe972b.png)
=> BingCherry를 리턴하여도 그 상위 객체인 Cherry로 캐스팅 되어 문제없이 수행됨
![image](https://user-images.githubusercontent.com/85000118/139185969-7af12ced-eeae-4952-8102-bb9e3a98e846.png)
=> 따라서, 실제로 사용자들에게 알려주기 위해 오버라이드하며 교체할 수 잇음

![image](https://user-images.githubusercontent.com/85000118/139186004-b7adb1e3-0bd4-4edf-9f98-d1994c0d0531.png)

**리턴 타입을 void*처럼 전혀 관련 없는 타입은 변경이 불가능함**

오버라이드 타입 변경해도 되는지 알아보기 위한 좋은 방법은
=> 변경해도 기존 코드가 제대로 동작하는지 확인하기
=> 리스코프 치환 원칙(Riskov Substitution Principle, LSP)

위의 예제에서 일반 포인터를 사용한 이유
=> unique_ptr<Cherry>는 unique_ptr<BingCherry>로 변경이 불가능 함
=> unique_ptr가 클래스 템플릿이기 때문에 두 가지는 전혀 다르고 관련이 없음
(내용물만 상속관계일 뿐)


**2.메서드 매개변수 변경하기**
virtual 메서드를 인수 다르게 오버라이드하면, (뒤에 override 키워드 안붙이고)
=> 수행은 되지만, 오버라이드는 아님
=> C++ 표준으로 인수를 다르게 선언하면 기존 부모 메서드를 가림
(ex. 인자 없는 부모 메서드를 오버라이드하여 int 인자를 받는다고 선언하면 객체에서 인자 없는 메서드 호출이 막힘)
![image](https://user-images.githubusercontent.com/85000118/139187003-02c8a0c7-1566-49cd-89ed-bc188709c978.png)

이러한 실수 방지를 위하여 뒤에 override 키워드를 붙이자

새로운 인자의 메서드도 필요하다면?
![image](https://user-images.githubusercontent.com/85000118/139187150-c1f2fa89-cddb-4160-9817-4e13c614375f.png)

**using**키워드로 부모 메서드 사용을 명시하자


**생성자 상속**
using 키워드는 생성자에도 적용이 가능함

ex. Base = 디폴트 생성자 + string_veiw 인자 생성자
Derived = int 생성자

=> string_view 인자를 받는 객체를 Derived는 생성 못함
```c++
class Derived : public Base
{
using Base::Base;
Derived(int i);
};
```
위처럼 using 키워드 사용하면 Base 디폴트 생성자 제외 모든 생성자를 상속함
=> Dervied d("Hello");   가 가능해짐
=> 동일한 인자의 생성자를 선언하면 오버라이드 됨

**using 상속 제약사항**이 있음
 - 생성자 일부만 상속받을 수는 없음(모두 받아야 함)
 - 여러 베이스 클래스에서 인자 목록이 똑같은 생성자 상속 불가(어느쪽인지 구분 불가)
```c++
class Derived : public Base, public Base2
{
using Base::Base;
using Base2::Base2;
Derived(int i); 
// 만약 2가지 모두 float 타입 인자를 받는 생성자가 있다면
// override로 명시함
Derived(float f);
};

// 원한다면 둘 모두 생성자 이니셜라이저를 이용
Derived::Derived(float f) : Base(f), Base(f) { }
```

상속한 생성자 활용할 때는 **모든 변수가 제대로 초기화 됐는지 확인 필요**
=> 상속받은 생성자를 활용하면 파생 클래스 초기화를 안할 수 도 있음
=> 선언할 때 내부 멤버 이니셜라이저로 처리
```c++
using Base::Base;
private:
int mInt = 0;
```

**메서드 오버라이딩의 특수한 경우**
메서드 오버라이드 주의를 기울여야 할 특수한 상황
흔한 몇 가지

**1. 베이스 클래스가 static인 경우**
static 메서드는 오버라이드 불가
static virtual 동시 지정 불가
=> 동일 함수명으로 오버라이드? 하면 별개의 메서드로 생성됨
=> 스코프 지정하지 않으면, 객체의 참조 타입으로 사용이 결정됨
ex. Base& ref = myDerived;
=> myDerived.methods(); 와 ref.methods() 는 다르게 호출됨


**2. 베이스 클래스 메서드가 오버로드된 경우**
여러 오버로드된 메서드가 있을 때, 한 버전만 오버로드?
=> 다른 버전 가려버림
=> 컴파일러가 에러 발생을 방지해 메서드를 지워줌 = C++이 편의상 제공
( 일부만 수정할일은 거의 없기 때문 )

그러나 접근할 방법은 있음
=> Derived 객체를 Base 포인터나 레퍼런스로 가리킴
=> 가려진 메서드여도 베이스로 가리키면 접근 가능

**이것또한 using 키워드를 이용하여 사용가능**
=> using Base::method;
=> 필요한 부분만 virtual void ~~ override 하면 됨
**using 사용 주의 사항**
=> 베이스에 오버로드 메서드가 추가가 되었을 때, 이를 수정해야 하는데 그냥 사용해버리면 문제 발생됨 ( 애초에 이를 막으려고 가려주던 것 )

버그 방지를 위하여 **모든 메서드 오버로드** 하거나 **using을 적절히 사용**


**3.private나 protected로 선언된 베이스 클래스 메서드**
둘 다 오버라이드 가능 함 ( 접근 지정자는 호출 대상만 제한할 뿐 )
=> 부모 클래스의 private를 호출만 못할 뿐 오버라이드는 가능
=> 파생클래스만의 버전 정의 
(참고 = C#, JAVA는 private 오버라이드 불가)

예를 들어, 각 차종 별로 연비 호출을 하는 메서드를 사용할 때 등
=> 골격 유지하며 private, protected로 특정 기능만 변경을 위해 오버라이드

**4.베이스 클래스 메서드에 디폴트 인수가 지정된 경우**
디폴트 인수가 있을 때는, 호출하는 타입에 따라 지정됨
```c++
Base::go(int i=2);
Derived::go(int i=3);

Base& ref;
Derived d;
d.go()  => i = 3;
ref = d;
ref.go(); => i = 2;
```
참조하는 객체의 디폴트 인수를 사용하는게 아닌, 자신의 타입 사용
=> C++ 디폴트 인수는 실행 시간이 아닌 컴파일 시간에 타입 보고 결정
=> C++ 디폴트 인수는 상속이 안됨 ( 변경하면 오버라이드 )

디폴트 인수 메서드 오버라이드는 반드시 인수 지정해주고, 되도록 동일한 값
=> 기호 상수(symbolic constant) 이용하면 더 좋음


**5.베이스 클래스 메서드와 접근 범위를 다르게 지정하는 경우**
메서드 , 데이터 접근 제한 방법 2가지
1. 베이스 클래스 전체에 대한 접근 지정자 변경
2. 파생 클래스에서 접근자를 다르게 지정 ( 오버라이드 )

부모(public) => 자식(protected) 변경은 가능하지만,
=> 부모 참조 포인터로 자식을 가리키면 사용이 가능하다
( protected는 상속 받은 대상만 사용 가능한데, 본래라면 외부호출 불가해야 함 )
=> 접근 권한 변경만 한다면 굳이 정의할 필요 없이 using 사용



접근 권한 축소보다 확장이 더 현실적임
protected -> public

3가지 방식
1. 파생 클래스에서 public 메서드 정의하고 내부에서 부모 protected 호출
2. public으로 오버라이드 => 구현 그대로, 권한만 변경 = using

![image](https://user-images.githubusercontent.com/85000118/139371025-9f4bedd3-2fd5-4f9e-9233-d39f029f224f.png)

부모 타입으로 객체 메서드를 불러와도, 부모가 protected라서 외부에서 호출하면 컴파일 에러


**파생 클래스의 복제 생성자와 대입 연산자**
파생 클래스에서는 특수 데이터가 있거나, 디폴트가 아닌 연산자 정의할 경우가 아니라면 다시 정의할 필요가 없음
=>  디폴트가 생성되고,  베이스 타입들은 베이스에서 처리됨

만약 파생에서 복제 생성자 정의한다면, 부모의 복제 생성자를 호출해야 함
=> 그렇지 않으면 디폴트 생성자가 호출됨
```c++
Base::Base(const Base& src) { }

Derived::Derived(const Derived& src) : Base(src) { }
```

operator 연산 또한, 일부만 대입하는게 아니면 부모 것도 호출
```c++
Derived& Derived::operator=(const Derived& rhs)
{
if(&rhs == this) return *this;

Base::operator=(rhs);

return *this;
}
```

=> 파생에서 연산자 지정하지 않으면 베이스 클래스 것 그대로 사용
(정의 했다면, 명시적으로 호출해야함 initializer or Base:: 호출)

=> 전문 C++ 프로그래머는 복제 기능 구현할 때 다형성 버전의 clone() 메서드를 구현하는 기법 사용
= 표준 복제 생성자 or 복제 대입연산자만으로는 부족해서 (추후 설명)


**실행 시간 타입 정보**
C++은 다른 객체 지향보다 컴파일 시간에 결정하는 것이 많음
(ex.  메서드 오버라이드는 객체 내부에 있는 메서드 선언과 구현 연결 관계 보고 작동 )

실행 시간 객체 들여다 보는 기능 => RTTI, Run-Time Type Information
대표적인 예 = dynamic_cast()  - 객체 사이 타입 안전 변환
=> virtual 메서드가 없는 클래스에 대해 dynamic_cast() 호출하면 컴파일 에러

다른 기능 typeid 연산자 
=> 실행시간 객체의 타입 정보 조회 

```c++
#include <typeinfo>
class Animal { public: virtual ~Animal() = default; };
class Dog : public Animal {};
class Bird : public Animal {};

void speak(const Animal& animal)
{
	if (typeid(animal) == typeid(Dog))
		cout << "WOOF";
	else if (typeid(animal) == typeid(Bird))
		cout << "Chirp!";
}
```
=> 사실 오버라이드하여 사용하는게 객체지향적
(typeid 연산자는 vtable이 있을때만 올바르게 작동)
=> 연산자에 전달된 대상의 레퍼런스, const 정보 제거해버림

주로 디버깅에 활용됨 
![image](https://user-images.githubusercontent.com/85000118/139610984-a9fd8f7e-b434-4e9f-b57b-19990304603a.png)
![image](https://user-images.githubusercontent.com/85000118/139610996-6fe68d01-c5b5-4265-bc1c-cdf418a7d295.png)

=> class Foo : Hello logger. 
=> 3Foo: Hello logger
등 컴파일러마다 구체적인 형태는 다르지만 위처럼 동작됨
(로깅, 디버깅 용도가 아니면 virtual이 좋다)


**non-public 클래스 상속**
지금까지는 뒤에 public Loggable 처럼 public 키워드를 사용했지만, 다른것도 사용은 가능하다. (접근 지정자 안붙이면 class는 기본 private, struct는 기본 public 적용)

부모를 protected 지정?
=> public 멤버들이 모두 protected취급

부모를 private 지정?
=> public, private 멤버가 모두 private

보통 이렇게 일괄 축소하는 경우는 상속 관계를 잘못 디자인 했기 때문
non-public은 매우 드뭄


**가상 베이스 클래스**
다중상속은 모호하다고 하였는데, 부모 클래스 기능 정의하지 않으면 어느정도 해결
![image](https://user-images.githubusercontent.com/85000118/139611518-f9307593-da07-4de4-915c-88cdcbd3bc77.png)

그러나 C++은 부모 클래스도 자체 기능을 가질 수 있는 **가상 베이스 클래스** 제공

sleep 함수를 정의한다고 가정하면, 양쪽 중 어느쪽을 사용할지 모호하기 때문이다.

![image](https://user-images.githubusercontent.com/85000118/139611707-0adace06-8e0b-4b29-8835-790b370cd8e8.png)

virtual로 부모 클래스를 상속 받으면, 해결 가능함
