### 리눅스 GCC의 라이브러리 인텔리센스 기능 정상 동작을 위한 TroubleShooting

인터넷 참고하여 Visual Studio와 리눅스 SSH 연결 설정 이후 참고

도구 - 옵션 - 플랫폼 간 - 연결 관리자 - 원격 헤더 IntelliSense 관리자  
에서 추가된 캐시 목록을 "탐색" 누르면 하단의 이미지처럼 폴더가 뜸  

[관련 질문글](https://stackoverflow.com/questions/52125610/visual-studio-remote-linux-headers)
에 따르면 기본 설정인 "rsync_ssh"는 모든 헤더를 다운로드 하지 않아 헤더 위치를 찾지 못한다.  

따라서, 아래처럼 sftp_ssh로 변경해주어야 함

추가로 setting.xml도 기본은 뒤에 확장자 .unused가 붙어 별도로 만들어주거나, 이름을 변경하여 적용해야 함

![image](https://user-images.githubusercontent.com/85000118/153153497-143492de-f63f-4c18-a68f-947e73aabd7b.png)
