주석 Comment  = //
여러 줄 주석 Multiline Comment = /*  */

**C++ Build**
PreProcess 전처리 - 소스코드에 담긴 메타 정보 처리
Compile 컴파일 - 기계 처리가 가능한 Object 목적파일로 변환
Link 링크 - 여러 오브젝트 파일을 애플리케이션으로 엮기

선언은 Header파일에
구현은 Source파일에

**전처리 지시자**
#include - 해당 파일의 내용을 그 위치에 삽입 ( 치환 )
#define X Y - X를 Y로 치환
#ifdef X  - X가 define 되어 있다면
#ifndef X - 반대라면
#endif - if의 끝
#pragma - 컴파일러마다 다른 지시자

**Header Guard**
![image](https://user-images.githubusercontent.com/85000118/136638784-15e88311-29fc-4ac2-9c0c-499d94189ac4.png)
컴파일러가 지원한다면 #pragma once 로 간결하게 사용

**main함수**
Retrun = int - 생략하면 자동 0 Return
1번째 매개변수 = 인자 개수 ( int argc)
2번째 매개변수 = 문자열 배열 (char* argv[])

**I/O스트림**
std::cout, std::cin, std:cerr

![image](https://user-images.githubusercontent.com/85000118/136639372-404217cc-fcfa-4497-90ef-d4e7bdba96db.png)

printf와 scanf를 안 쓰는 이유 :
 - 타입 안정성(Type Safe)를 보장하지 않기 때문 - 정해진 타입을 정해진 규칙에 의해 사용하는 것

**네임스페이스**
코드 내 이름 충돌하는 문제를 해결하기 위한 개념

Header 내 선언 = namespace xx{ void func(); }
Source 내 선언 = 
 - namespace xx{ void func(){ int a; } }
 - void xx::func() { int a; }

Scope Resolution Operator 스코프 지정연산자 사용 ( :: )

using으로 스코프 연산자 생략 가능 ex. using namespace std; 
Header 내에 using 사용 금지 ( 의도치 않게 됨 )

중첩된 네임스페이스 ( nested namespace )
 - C++17 부터 사용 쉽게 개선
 - namespace 안에 namespace 사용하는 것 개선
 - namespace x::xx::xxx { } 형태로 사용 가능
 - Namespace alias쓰면 더욱 간단하게 사용 - namespace xxx = x::xx::xxx;

**리터럴Literal**
코드에 표시한 숫자나 스트링 같은 값 ( 변하지 않는 값 )
상수 - 값이 변하지 않는 변수
리터럴 - 변하지 않는 값 (ex. 가장 작은 양의 정수 = 1 )

ex.
 - 10진수 = 123
 - 8진수 = 0173
 - 16진수 = 0x7B
 - 2진수 = 0b11111011
 
C++
ex.
 - 부동소수점 = 3.14f
 - 배정도 부동소수점 = 3.14  ( 단정도의 2배 float double 차이 )
 - 단일문자 = 'a'
 - NULL로 끝나는 문자열 = "AAAAA"
 
자리수 구분자
 - 23'456'789
 - 0.123'456f
 - C++17에서는 16진수 부동소수점도 가능  0x3.ABCp-10, 0Xb.cp121);

**변수**
일반적으로 선언 후, 초기화 하지 않으면 오류나 버그가 발생할 가능성이 높음

상수
const keyword = 값을 변하지 않게 만드는 것
const int 

C++ 타입
 - int
 - short
 - short int
 - long   - long l = -7L;
 - long long ll = 14LL;
 - unsigned ~~ = ex.  2U  2UL  2ULL
 - float - 3.14f
 - double = 3.14
 - long double = 3.14L
 - char
 - char16_t = 16bit 단일문자 char16_t c16 = u'm';
 - char32_t = 32bit   U'm';
 - wchar_t = 단일 확장 single wide 문자 - 컴파일러마다 다름 wchar_t w = L'm';
 - bool = stdbool.h
 - std::byte  = C++17 전용 - std::byte b{42}; - 문자라는 개념이 아니라 정확히 한 바이트라는 의미

**캐스팅(Casting) , 동적 형변환, 타입캐스팅(typecasting)
실행 중 변수 타입 변경

C++의 명시적 형변환
ex. float myFloat = 3.14f
1. int i1 = (int)myFloat;   => C언어 방식 피해야함
2. int i2 = int(myFloat); => 거의 사용 안함
3. int i3 = static_cast<int>(myFloat); => 가장 명확함

소수점 아랫부분을 잘라낸 정수로 만듬

문맥에 따라 강제 형변환(coerce, coercion) 될 떄
short 값을 long에 대입하면 자동 변환  = > long의 정밀도(precision)이 더 넓어서
short someShort;
long someLong = someShort;

자동캐스팅은 대입 대상과 완전히 호환될 때만 사용 추천

**연산자(Operator)**
- 이항연산자 (Binary)
- 단항연산자 (unary)
- 삼항연산자 (ternary)

![image](https://user-images.githubusercontent.com/85000118/136641554-a8cd6e45-092f-416d-a813-2c45c8a46e6c.png)

**추가 타입**
기본 타입의 조합으로 특수한 타입 사용 가능 C언어도 사용 가능한 것
1. 열거타입 (enum)
 - 순차적으로 고유 번호를 갖게 만드는 타입
 - enum a { a = 1, b, c=10, d }  와 같이 구현하면 각각 1,2,10,11 이 대입됨  ( 아무것도 대입하지 않으면 자동 0 시작 )
 - 타입을 엄격하게 따지지 않음 ( a myEnum = 1; 과같이 사용이 가능함 - 정수형으로 대입 )
2. 엄격한 열거 타입
 - 열거 타입과 다르게 Strongly Typed = Type safe 함 
 - enum class 타입이라고 함 ( enum class a { 내용 동일 }; )
 - 스코프(유효 범위)가 자동으로 확장되지 않아서, 내부적으로만 유효 함 ( a myEnum = a::b; ) 이런식으로 지정해야 함
 - 엄격하기 때문에 정수와 비교하기 위해서는 static_cast<int>a::b 와 같이 명시적 형변환 필요
 - 기본적으로 정수 타입이지만, enum class a : unsigned long { } 같이 내부의 타입을 지정 가능함
3. 구조체(Struct)
 - 한 개 이상의 타입을 묶어 만들어 내는 형태 ( ex. DB 레코드 - 내부 선언한 변수를 모두 field로서 가짐 ) 
 - struct A { int a; int b; int c;}; 형태로 struct A instance; instance.c 로 접근 가능 or struct A instance = { 1,2,3 }; 순서대로 대입가능

**조건문**
주어진 코드 참 거짓 판별을 위해 주로 사용

1. if/else문
 - 일반적인 if / else if / else 문
 - C++17부터 초기 값 부여 가능 
 - for문처럼 if(  초기 값 ;  조건문 { }
2. switch
 - switch() { case x: ~~~ break;  default: break; } 
 - switch 문 내 결과 값은 반드시 정수 표현 가능해야함
 - 보통 깔끔해 보이기 위해서 if else if가 많을때 사용함
 - break문 중간중간 넣어주지 않으면 fallthrough폴스루 (흘려보내기)라고 하여 다음 case문도 모두 실행됨
 - C++17에서는 폴스루에 대한 컴파일 경고문이 나옴 ( 의도한 경우, break; 대신 [[fallthrough]]; 사용하면 됨
 - switch( 이니셜라이저 ; 표현식 ) { } 가능
3. 조건 연산자
 - 인수 3개를 받는 유일한 삼항 연산자
 - 조건 ? 동작1 : 동작2
 - 코드 안에서 사용 가능하다는 장점 ex. int i = a>2 ? 1 : 0
 
**논리 연산자**
![image](https://user-images.githubusercontent.com/85000118/136644628-49413a2c-6c8e-42ee-a424-545d3ddc1cd7.png)

C++ 논리 표현식 = 단락 논리(축약논리 - short-circuit logic)방식으로 평가함
ex. && 앞부분이 false면 검사 진행 X  ,   || 도 앞부분이 true면 검사 진행 X

**함수**
큰 프로그램은 main() 안에 모두 담기 힘들어 가독성이나 관리를 위해 함수 단위로 나눔

보통 선언부는 헤더파일에  ex. void abc(int a);
구현부는 소스파일에 ex. void abc(int a) {  printf("%d",a); }

선언부를 함수 프로토타입 or 함수원형 or 함수 헤더라 부름 ( 어떻게 사용할지에 대한 명시 )
리턴 타입 제외한 함수 이름 + 매개변수 목록 = 함수 시그니처 ( 함수 서명) 이라 부름

리턴값이 없을 때나 매개변수가 없으면 void 인데 리턴타입은 반드시 지정해줘야 함
abc();   => void abc();

알면 유용한 함수 설정
1. 함수 리턴 타입 추론
 - C++14부터 사용 가능한 auto 키워드를 리턴 타입 자리에 놓으면 컴파일러가 지정해줌
 - 단, 리턴문이 여러개라면 리턴 타입은 반드시 동일해야 함 ( 재귀문도 마찬가지 )
2. 현재 함수 이름
 - 함수 내부적으로 __func__ 라는 변수를 사용하면 함수 이름 값을 반환 ( 로그용도 )

**배열**
1. C Style
 - 배열 크기는 반드시 상수 표현식(constant expression, constexpr)으로 지정 ex. int arr[3];
 - 배열의 시작은 0
 - 원소 초기화는 Zero-initialize로 디폴트 생성자로 초기화 가능 (char,int는 기본 0, 소수는 0.0 포인터는 nullptr)   ex. int arr[3] = {};  0을 넣어도 되고 안넣어도 가능
 - initilizer List 초기화 리스트 사용해도 됨 int arr[] = {1,2,3 }; arr[3]개의 각 값 지정 - 변수 개수 지정하고 초기화 리스트의 개수가 그보다 적다면 나머지는 0 ex. int arr[3] = {2,};
 - C++17의 기능 - <array> 헤더의 std::size() 함수 사용가능 ex. std::size(arr);  C언어에서는 sizeof(arr[0]) * sizeof(arr) 방식 사용
 - 그 밖에 2차원, 3차원 배열도 있음
2. std::array
 - C언어랑 동일한 형태
 - 장점 : 항상 크기를 정확히 알 수 있고, 자동으로 포인터를 캐스팅(동적 형변환) 하지 않아 버그 방지 기능, 반복자(interator)로 배열에 원소에 대한 반복문 쉽게 작성 가능
 - std::array<int,3> arr = {1,2,3};  - template
 - arr.size(), arr[1] 사용 
 - 위 장점 제외하면 C 배열과 동일
3. std::vector
 - 동적 할당 가능한 배열 ( 제네릭 컨테이너 - 범용 )
 - 거의 모든 종류의 객체를 담을 수 있음
 - vector<template> a[3] = { X , X , X };  , a[0].push_back(A);

**구조적 바인딩(structed binding)**
C++17부터 도입

```c++
std::array<int,3> values = {11,22,33};

auto [x,y,z] = values;
```

 - 구조적 바인딩 사용에는 반드시 auto 키워드로 선언
 - 왼쪽의 선언 변수 개수와 우측 표현식 값 개수가 반드시 일치 필요
 - 구조체나 pair, tuple 등 에도 적용 가능

**반복문**
1. while
2. do while
```c++
do{
 i++;
} while( i < 5);
```
3. for
4. 범위 기반 for문
컨테이너에 담긴 원소에 대한 반복 실행
C스타일 루프, Initializer_list, array, vector 등 컨테이너의 반복자를 리턴하는 begin(), end() 메서드 사용하는 모든 타입에 사용 가능
```c++
std::array<int, 4> arr = { 1,2,3,4 };
for (int i : arr) {
	std::cout << i << std::endl;
}
```
반복을 돌면서 복제본을 자동 사용함(복제하지 않고 사용하려면 뒤의 레퍼런스 변수 활용)


**이니셜라이저 리스트**
<Initializer_list> 헤더 파일에 정의된 클래스 템플릿
``` c++
#include <initializer_list>
int abc(std::initializer_list<int> lst) {
	int sum = 0;
	for (int val : lst)
		sum += val;
	return sum;
}
int main()
{
	std::cout << abc({ 10,20,30,40 });
	return 0;
}
```
Result : 100

Initializer_list 는 타입 세이프 
 - 인수로 지정한 int 외에 double로  10.0이 사용되면 컴파일 에러 or 경고
