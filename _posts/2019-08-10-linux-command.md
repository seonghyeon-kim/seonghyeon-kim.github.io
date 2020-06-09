---
layout: post
title: 간단한 리눅스 명령어 정리
tags:
- linux
---

### 개요
서버를 빌려 사용할 경우 쉘로 접속해 터미널을 이용하여 서버를 만지게 된다. 이로 인해 간단한, 서버 구축에 최소로 필요한 명령어들을 정리했다. 서버를 만지기 전 꼭 아래에 있는 명령어들은 숙지해야만 서버를 만질 수 있다. 글로만 읽지 않고 서버에 대고 직접 해보는 것을 권장한다.

---
- ssh ​user​@​서버 IP
입력한 서버에 입력한 유저로 ssh 쉘에 접속한다.
![image](/images/linux/1.png)
- cd ​폴더명
입력한 폴더로 이동한다.
![image](/images/linux/2.png)
- cd ..
현재위치의 상위 디렉토리로 이동한다.
![image](/images/linux/3.png)
- ls -al
현재 폴더의 파일들이 무엇이 있는지 알려준다.
![image](/images/linux/4.png)


- mkdir ​폴더명
입력한 폴더명으로 폴더를 생성한다.
![image](/images/linux/5.png)
- rmdir ​폴더명
입력한 폴더명의 폴더를 제거한다.
![image](/images/linux/6.png)
- rm -rf 폴더명
![image](/images/linux/7.png)
- chown ​이름:그룹 폴더명
입력한 폴더명의 소유자를 입력한 이름:그룹으로 바꾼다.
![image](/images/linux/8.png)
![image](/images/linux/8-2.png)

- su - ​유저명
입력한 유저명으로 사용자를 바꾼다.
![image](/images/linux/9.png)
- su
root 사용자로 바꿔 로그인한다.
![image](/images/linux/10.png)


**nano editor** 사용
- nano ​파일명
nano editor로 입력한 파일명의 파일을 연다.
![image](/images/linux/11.png)
![image](/images/linux/12.png)
- ctrl + x 저장
확인 창이 하단에 뜰것이다.
![image](/images/linux/13.png)
y를 입력하면 내용을 변경하고 n을 입력하면 취소하고 창을 닫는다.
