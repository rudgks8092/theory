C언어에서는 null로 끝나는 문자 배열로 스트링을 표현
=> Buffer overflow나 여러 보안 취약점 좀재
C++ 에서는 Standard Library std::string 클래스를 제공

### **동적 스트링**
C언어는 다른 여러 언어처럼 문자열을 주로 다루지 않는다. ( 최소한의 기능 )
C++은 핵심 데이터 타입으로 제공

###**C 스타일 스트링**
C언어 NULL을 이용한 문자배열로 표현 (\0)
따라서, 길이 5의 문자열 표현을 위해서는 최소 6 사이즈의 배열 필요

**<cstring> = <string.h>** 헤더를 사용하여 문자열 기본 함수 사용 가능
동적 할당에 대한 구현은 직접 수행해야 하며, 메모리 해제도 직접 해야 한다.
**strlen 함수로 나온 결과 값의 + 1만큼 해제 해야 함**
**길이를 확인할 때, sizeof 연산자와의 차이를 주의**
이러한 메모리 문제들로 인하여 strcpy_s, strcat_s 같은 표준 라이브러리 함수가 나왔다. 

여러 문자열은 검색하여 확인 필요 (strcat, strtok, strstr)

### **스트링 리터럴**
```c++
cout << "Hello" << endl;
```
위 처럼 변수에 담지 않고 곧바로 표현한 스트링을 String Literal이라 부름
컴파일러는 같은 스트링 리터럴에 대해선 레퍼런스를 재활용하는 식으로 메모리를 절약함
즉, Hello를 몇백번해도 딱 하나의 메모리 공간만 할당함 => 리터럴 풀링(literal Pooling) 이라 함

C++ 표준에서는 스트링 리터럴 = const char가 n개인 배열으로 정의
하지만, 위와같이 동작하지 않는 컴파일러도 있음
따라서, 스트링 리터럴을 변수에 담는 행위는 지양함
```c++
char* ptr = "hello";
ptr[1] = 'a';  // 결과 예측 불가

// 따라서 대입을 해야한다면 const 키워드를 이용
const char* ptr = "hello";
ptr[1] = 'a'; // const라서 오류

// 문자배열 초기값 설정에도 스트링 리터럴 사용 - 배열을 직접 생성하기 때문에, 문제 없음
char arr[] = "hello"; 적절한 크기의 문자배열 컴파일러가 생성
arr[1] = 'a'; // 스트링 수정 가능
```

### **로 스트링 리터럴(Raw String Literal)**
여러 줄에 걸쳐 작성한 스트링 리터럴
=> 인용부호를 이스케이프 시퀀스로 표현하지 않음 (\t \n 등)
```c++
const char* str = "Hello "World"!"; // World를 감싸는 "를 \"로 표현하지 않아 컴파일 에러
const char* str = "Hello \"World\"!";  // 정상

// Raw String Literal 이용하면 이스케이프 시퀀스 사용 안해도 됨  => R"()"
const char* str  = R"(Hello "World"!)";

// 여러 줄 출력에도 용의함 (\n를 안써도 됨)
const char* str = R"(Line 1
Line2)";

// 단, )" 로 끝나기 때문에 )"를 넣을 수 없다.
const har* str = R"(Embedded )" characters)"; // 에러 발생

// 따라서, )" 사용하기 위해서는 확장 로 스트링 리터럴(extended Raw String Literal)구문으로 표현
// 사용 방법은 R"()"  => R"?()?" 형태로 물음표 부분에 최대 16길이의 구분자 시퀀스(delimeter sequence)를 사용
const char* str = R"-(Embedded )" characters)-";
```
로 스트링 리터럴을 이용하여 DB나 파일경로 사용이 편리해짐

### **std::string 클래스**
C에서 개선되어 메모리 할당 작업을 직접 해줌   **<string>**

C 스타일 스트링의 문제점
1. 장점
 - 간단함 ( 기본 문자타입, 배열 구조체 )
 - 가벼움 ( 꼭 필요한 만큼 메모리 사용 )
 - 로우 레벨 ( 메모리 조작 or 복사 쉬움 )
 - 새로 배울 필요 없음
2. 단점
 - 고차원 기능 구현이 필요함
 - 찾기 힘든 메모리 버그 발생 빈도 높음
 - 객체지향 특성 제대로 활용 못함
 - 내부 표현 방식을 이해해야 함

string은 장점을 그대로, 단점을 개선

+ 연산자 오버로딩을 통해 스트링 결합(string concatenation) 연산 수행 정의 가능
Append도 가능
Compare도 가능 ( == ) - C언어는 포인터를 비교하기 떄문에 항상 false ( C언어는 strcmp(a,b) )
Assign도 가능  string a = b; (string에 string 할당

1. Stack 변수로 생성되어 메모리 누수가 발생하지 않음 ( 소멸자가 알아서 처리 )
2. 연산자를 원하는 방식으로 작동 가능

string.c_str( ) 메서드를 사용하면 const 문자 포인터를 리턴하여 C 형태로 호환성 보장
( 다만, c_str에 사용한 string의 객체 제거 or 메모리 재할당을 하면 더이상 사용 불가 ) => 리턴 값으로 활용하면 안됨

비슷한 함수로 string.data()가 있다. 
원래 c_str와 동일하게 const char*를 리턴하였으나, C++17부터는 non-const에 대해서는 char*를 리턴

### **std::string literal**
```c++
// 스트링 리터럴은 보통 const char*로 처리 됨
auto string1 = "Hello World"; // const char*
auto string2 = "Hello World"s; // string - 뒤에 표준 사용자 정의 리터럴 s 를 붙이면 string으로 변환

// 표준 사용자 정의 리터럴 s 사용을 위해서는 using namespace std or std::string_literals; 를 추가
```

### **하이레벨 숫자 변환**
**to_string** 함수를 통하여 다양한 숫자들에 대하여 string을 메모리 할당하여 리턴해줌
 - stoi
 - stol
 - stoul
 - stoll
 - stoull
 - stod
 - stof
 - stold
과 같이 다양한 String To  Num 기능을 제공 
 - 앞의 공백 문자들을 무시하여 처리
 - 변환 실패 = invalid_argument exception
 - 리턴 타입 범위 벗어나면 out_of_range exception

정수형 매개변수 = str, idx, base ( idx = 아직 변환되지 않은 맨 앞 문자 인덱스를 가리키는 포인터 )
실수형 매개변수 = str, idx

```c++
const string toParse = "   123USD";
size_t index = 0;
int value = stoi(toParse,&index);
cout << value << endl; // 123
cout << toParse[index] << endl;  // U
```

### **로우 레벨 숫자 변환**
C++17부터 로우 레벨 숫자 변환 함수도 제공
**<charconv>** 헤더 사용 - 메모리 할당 작업 안해주기 때문에, 버퍼 할당 방식 사용 필요
고성능 + 로케일 독립성(locale-independent)에 튜닝
=> 하이레벨 숫자 변환에 비해 처리 속도가 엄청 빠르다.
```c++
// 리턴하는 타입
struct to_chars_result {
	char* ptr;
	errc ec;
};
to_chars_result to_chars(char* first, char* last, IntegerT value, int base = 10);
// IntegerT는 정수,부호 없는 정수, char 타입이 나올 수 있음

// 정상 변환 => ptr = 끝에서 두 번째 문자, 그렇지 않으면 last 값 (이 때 ec == errc::value_too_large)

string out(10, ' ');
auto result = std::to_chars(out.data(), out.data() + out.size(), 12345);
// 구조적 바인딩 이용한 경우,
auto [ptr, ec] = std::to_chars(out.data(), out.data() + out.size(), 12345);

if (result.ec == std::errc()) { // 제대로 변환 된 경우}
// 구조적 바인딩
if(ec == std::errc()) { }

// 부동 소수점 변환도 제공
to_Char_result to_chars(char* first, char* last, FloatT value);
to_Char_result to_chars(char* first, char* last, FloatT value, chars_format format);
to_Char_result to_chars(char* first, char* last, FloatT value, chars_format format, int precision);
// FloatT 부분에 float, double, long double
// chars_format에 플래그 조합
enum class chars_format {
	scientific, // 스타일 : (-)d.ddde +-dd
	fixed,  // 스타일 : (-)ddd.ddd
	hex, // 스타일 : (-)h.hhhp +-d (주의: 0x는 적지 않는다)
	general = fixed | scientific 
};

```
![image](https://user-images.githubusercontent.com/85000118/136871084-369e53fc-25c2-4188-87b9-25d8bf496ff1.png)
![image](https://user-images.githubusercontent.com/85000118/136871099-7972b9ab-7748-4858-b7bc-d893f5a5ed01.png)
![image](https://user-images.githubusercontent.com/85000118/136871119-a1b89fdc-341d-4f6a-9276-0a99c88f557d.png)


### **std::string_view 클래스**
C++17이전에는 읽기 전용 스트링을 받는 함수 매개변수 타입을 쉽게 결정할 수 없었음
 - const char*로 지정하면 c_str, data()를 이용해야 하는데, 이러면 객체 지향 속성, 헬퍼 메서드 활용 불가
 - const std::string&로 하면 항상 string만 전달해야 함
 - 스트링 리터럴을 전달하면 복제하여 string 객체를 만들고 전달하기 떄문에 오버헤드
 - 오버로딩하여 2개 타입을 만들기도 하지만, 세련되지 않음

C++17부터 추가된 strimng_view 클래스로 해결
**<string_view>** 헤더 사용
c_str이 없다는 점을 제외하면 인터페이스는 string과 같다.

remove_prefix(size_t), remove_suffix(size_t) 메서드도 추가 제공
=>지정한 오프셋만큼 시작 포인터를 앞으로 당기거나 끝 포인터를 뒤로 미뤄 스트링 축소

```c++
string str= "Hello";
string_view sv = " world';
auto result = str + sv; // 컴파일 에러, 연결이나 결합 불가
auto result = str + sv.data();  // 사용 가능
```

string과 string_view의 사용법은 비슷하다.
```c++
// 파일 확장자 추출 함수 구현
string_view extractExtension(string_view fileName) {
	return fileName.substr(fileName.rfind('.'));
}
// 위와 같이 pass by value로 하여도 포인터와 길이만 갖고 있어 복사하는데 오버헤드가 적다. ( 복제 연산이 하나도 발생하지 않음 )
string fileName = R"(c:\temp\my file.ext)";
cout << extractExtension(fileName) << endl;
const char* cString = R"(c:\temp\my file.ext)";
cout << extractExtension(cString) << endl;
cout << extractExtension(R"(c:\temp\my file.ext)") << endl;
```
3개의 코드는 리터럴, const char*, string 모두 지원하며, 복제 연산이 발생하지 않아 굉장히 효율적

생성자 중에는 raw buffer와 길이를 매개변수로 받기도 함 ( NULL로 끝나지 않는 스트링 생성에 유용) ex. string_veiw(raw,length)

string_view는 string을 생성하는 것이 아니기 때문에, 위 함수의 리턴 값을 string을 이용하는 함수의 매개변수에 넣으면 제대로 동작하지 않음.
=> string_view.data()나  String(string_view)  처럼 data 메서드 또는 명시적 ctor를 이용

결론 : 읽기 전용 스트링을 받는 함수나 메서드는 string, char 대신 string_view로 지정하는게 좋다.

string의 스트링 리터럴 s처럼 sv를 이용하면 string_view도 스트링 리터럴을 사용 가능하다.
```c++
auto sv = "My string_view"sv;
// namespace std나 std::string_view_literals를 사용해야 함
```
**비표준 스트링**
C++ 규격이 명확하지 않아, 직접 개발해 쓰거나 다른 라이브러리를 이용하는 사람이 많았다.
(ex. MFC-CString)
=> 따라서, 사전에 어떤 string을 사용할 지 규정하는 것도 중요

단, 아래 사항은 준수하는게 좋다
 - C Style 스트링 사용 X
 - MFC or QT 등에서 기본적으로 제공하는 스트링 처럼 프레임워크 제공 스트링을 표준으로 삼음
 - std::string을 이용한다면, 읽기 전용 스트링은 string_view를 사용 ( 프레임워크 전용이면 비슷한 기능을 사용 )
