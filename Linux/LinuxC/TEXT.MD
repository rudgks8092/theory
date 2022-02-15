# 텍스트 처리

리눅스 시스템 프로그래밍으로 분류하긴 애매하지만, 기초만 다룸


텍스트에서 특정 패턴을 찾을 때  
=> memchr, strchr 등을 제대로 작성하면 빠름  
=> but, 패턴이나 규격이 바뀌면 코드를 수정해야 함  

**정규표현식**을 이용하여 해결  

### 정규표현식 REGEX  

정규표현식 쉽게 작업할 수 있는 예  
1. 전역 환경으로 사용되는 파일을 다른 스크립트, 쉘에서 사용할 수 있도록 쉽게 추출 변환 가능
2. DB에서 전달받은 데이터를 분석할 때, 동적으로 바뀔 수 있는 형태  

단순한 정형화된 텍스트 규격이면 memchr, strchr, strtok도 괜찮다.  
그러나 규칙이 변화한다면 매번 수정할 수 없으니 정규표현식을 사용한다.  

**C언어의 정규표현식**  
1. BSD 방식
2. POSIX 방식

호환성 문제로 POSIX가  더 나은 경우가 많음  
PCRE(Perl Compatible Regular Expression)이라는 강력한 정규표현식 기능도 있으므로, POSIX 정규표현식으로 해결안된다면 사용 고려  

### HTML Parsing 예제
```c++
/* 
     Usage : argv[0] search_string pattern
	 e.g.)  
	 ./posix_regex "Gnu means Gnu is not Unix" "(Gnu)+"
	 ./posix_regex " item = value1" "([a-zA-Z0-9]+)[\ ]*=[\ ]*([a-zA-Z0-9]+)"
*/
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <regex.h>
#define MAX_EXPR_SUB_MATCH	10
#define DEFAULT_REGEX_STR	"(</.+>).*<br>"
#define DEFAULT_DEST_STR 	"<center>align to center</center> align to left<br>New Line<br><br><p>"

int main(int argc, char **argv) {
	char *p_regex_str;	/* pattern string */
	char *p_dest_str;	/* string to apply pattern */
	regex_t	re_expr;	/* posix regex patern buffer */
	regmatch_t rm_matchtab[MAX_EXPR_SUB_MATCH];	/* pattern match table */
	int	i, ret;
	char errbuf[0xff];

    // 인자 처리
	if (argc != 3) {
		printf("Using default string!!\n");
		printf("* Dest str : %s\n", DEFAULT_DEST_STR);
		printf("* Regex str: %s\n", DEFAULT_REGEX_STR); 
		p_dest_str = strdup(DEFAULT_DEST_STR);
		p_regex_str = strdup(DEFAULT_REGEX_STR);
	} else {
		p_dest_str = strdup(argv[1]);
		p_regex_str = strdup(argv[2]);
	}

	/* compile re pattern buffer */
    // 컴파일 = 데이터 변환 및 처리 과정
	if ((ret = regcomp(&re_expr, p_regex_str, REG_EXTENDED|REG_NEWLINE))) {
		regerror(ret, &re_expr, errbuf, sizeof(errbuf));
		printf("[ERROR] regcomp() : %s\n", errbuf);
		exit(EXIT_FAILURE);
	}

	printf("regcomp : %s\n", p_regex_str);

	memset(rm_matchtab, 0x00, sizeof(rm_matchtab));
    // 패턴 매칭 확인
    // 맨 뒤 인자는 FLAG로 패턴에 대한 조건을 추가
	if (regexec(&re_expr, p_dest_str, MAX_EXPR_SUB_MATCH, rm_matchtab, 0)) {
		printf("[ERROR] Can't find pattern from \"%s\".\n", p_dest_str);
	} else {
		printf("* All Match offset : (%d -> %d), len(%d) : %.*s\n", 
				rm_matchtab[0].rm_so,
				rm_matchtab[0].rm_eo,
				rm_matchtab[0].rm_eo - rm_matchtab[0].rm_so,
				rm_matchtab[0].rm_eo - rm_matchtab[0].rm_so,
				&p_dest_str[rm_matchtab[0].rm_so]);
		for (i=1; i<MAX_EXPR_SUB_MATCH; i++) {
			if (rm_matchtab[i].rm_so == -1) break;
			printf("* Submatch[%d] offset : (%d -> %d), len(%d) : %.*s\n", i,
					rm_matchtab[i].rm_so,
					rm_matchtab[i].rm_eo,
					rm_matchtab[i].rm_eo - rm_matchtab[i].rm_so,
					rm_matchtab[i].rm_eo - rm_matchtab[i].rm_so,
					&p_dest_str[rm_matchtab[i].rm_so]);
		}
	}
	regfree(&re_expr); /* free memory */
	return 0;
}

```

자세한 정규표현식의 표현방법은 검색 필요


2008 개정된 문자열 함수 중 특수  
1. strndup (strdup의 대체)  
=> strdup는 문자열 복제로 내부적으로 malloc 할당된 포인터를 반환 (free해줘야함)
2. strerror_r  
=> strerror의 Thread-safe
