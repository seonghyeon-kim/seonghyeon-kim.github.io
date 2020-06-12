---
layout: post
title: Django + Nginx + gunicorn + AWS EC2 Deployment
tags:
- aws
- nginx
- gunicorn
- django
---
django, gunicorn, nginx, AWS EC2를 이용하여 프로젝트 배포.

### 개요

저희가 만든 프로젝트는 로컬의 가상환경에서만 돌아가고 있다. 처음 만들때 목적이 우리가 보기위해 만든것이 아니다, 클라이언트들이 사용할 수 있게끔 서버를 세팅하고 서버위에 장고를 올려보도록 하자.

참고 : ​https://lhy.kr/ec2-ubuntu-deploy
전체적으로 위 페이지를 보고 배포를 시작했다.

제목을 보면 django 빼고는 저게 다 뭐야? 이런 생각이 들것이다. 처음 저도 저걸 보곤 정말 막막했습니다. 아래 그림은 우리가 배포를 완료했을 때의 구성도이다.
![image](/images/deploy/1.png)
그림을 보고나니 뭐가뭔지 모르겠다. 하나씩 뜯어보자.

---
### Web Server 는 뭔데?

web server는 어떤 역할을 하는 것일까.

1. URL로 client가 뭔갈 보여줘! 하고 요청
2. 요청받은 HTML문서나 정적파일(static file,js,image)들을 보내줌

이 것이 전부다. 더 간단히 말하자면 정적인 데이터처리를 주로하는 클라이언트와 소통하는 녀석이다.

그럼 우리가 만든 django는 동적파일들이 있는데 어떻게 해요?
그래서 필요한 것이 WSGI다.

---

### WSGI 는 뭔데?

위에 설명에 따르면.. web server는 django를 모르니 알려줄 놈이 필요한 것이다. 보통 java나 php 를 이용하게 되면 CGI를 사용한다고 구글링하면 나와있지만 우린 파이썬의 규격인 WSGI를 사용하여 웹서버에게 django를 알려주면 될 것 같다.

---

### AWS EC2 는 뭔데? 그림에서 왜 감싸고 있는가?

우리가 배포를 시작할 환경이다. web server, wsgi, web application 들을 내 PC에 올리진 않을
것이니. 24시간 돌아야할 어떠한 서버가 필요할 것이다.

---

### AWS EC2 를 왜 쓴건데?

amazon에서 나온 클라우드 서버로, 비교를 하자면 cafe 24에서 가상 서버를 구매하게 되면 용량을 정하여 사용한다. 이 경우 일정 용량과 메모리 용량 CPU 처리량에 따라 접속자 수가 한정이 될 수 밖에 없다. amazon의 경우는? 변동사항에 따라 신속하게 확장하고 축소해준다. 단, 돈이 더 들뿐.. 이 외에도 키페어를 사용하고 보안 그룹 등 보안도 특화되있다. 이러한 장점으로 최근엔 정말 많이 쓰인다고 한다. 그리하여 EC2에 배포를 해보자.

---
### AWS EC2 인스턴스 생성

https://aws.amazon.com/ko/?nc2=h_lg
위 사이트에 들어가서 가입 후, 로그인하여 관리페이지로 접속한다.

인스턴스 생성전
![image](/images/deploy/2.png)
**지역 설정 꼭 해주자**

conda 설치파일 하나 보내는데 하루가 걸릴 수 있으니 주의. 즉, 사용은 한국에서 하는데 서버는 미국에 있는 것이다;;


검색창에 IAM이라고 입력한 뒤 클릭하여 사용자 및 액세스 키 관리 메뉴로 이동한다.
![image](/images/deploy/3.png)
이동 후 Users 탭으로 이동하여, 추가해준다.
![image](/images/deploy/4.png)
이름을 입력하고, 액세스 타입을 Programmatic access 로 설정한다.

액세스 타입은 두가지가 있는데,
우리가 선택한 타입은 개발 환경에만 액세스를 허용하는 것이고,
AWS management console access 는 콘솔에 액세스를 허용하는 것이다.

![image](/images/deploy/5.png)

next를 눌러 permission 설정 창으로 넘어간 다음 attach existing policies directly를 클릭하여 권한 목록을 불러온다. 검색창에 ec2full 를 검색한 다음 AmazonEC2FullAccess 정책을 체크한후 다음으로 넘어간다.

![image](/images/deploy/6.png)

다음 화면에서 Create user를 눌러 생성을 완료한다.

![image](/images/deploy/7.png)

생성 후 뜨는 창인데, Acess key ID와 Secret access key는 이창을 닫으면 다시는 볼 수 없다. 지금 작성하는 본인도열 수 없어 다른 사이트에서 가져왔다. 그러하니 꼭 다운로드를 하고 따로 저장해야만 한다.

### 키페어 생성

EC2라 하는 것은 키페어라는 것으로 접속할 수 있다. 그런 키페어라는 것을 한번 만들어보자.

상단의 서비스를 누르면 아래에 메뉴가 쭉 뜨는데 EC2를 클릭한다.

![image](/images/deploy/8.png)

메인 화면의 리소스 항목 아래의 키페어를 클릭한다.

![image](/images/deploy/9.png)

그후 키페어 생성을 클릭한 후 이름을 입력하면 pem파일이 다운로드 된다.

다운로드한 pem파일은 관리를 정말 잘해야한다. 잃어버리게 되면 다시 다운받을 수 없고, 유출이 되면 그서버는 끝이다. 무튼 그정도로 중요하게 잘 관리해야한다.

다운로드 받은 후 pem 파일의 권한을 바꿔주어야 한다.

- linux 환경
chmod 400 /파일명/
- window 환경
받은 파일의 마우스 우클릭 후
![image](/images/deploy/10.png)
속성을 클릭하여 본인의 계정의 읽기 및 실행, 읽기 만 허용해준다.
![image](/images/deploy/11.png)

### 인스턴스 생성하기

EC2 관리 메인화면에서 인스턴스 시작을 누릅니다.
![image](/images/deploy/12.png)
AMI 선택에서 ubuntu Server 16.04 를 선택해줍니다.
![image](/images/deploy/13.png)
선택 후 쭉 단계가 있을텐데 다음을 눌러 6.보안 그룹 구성으로 넘어갑니다. 
![image](/images/deploy/13-2.png)
보안 그룹 이름과 설명을 적어준 후 검토및 시작버튼을 누릅니다.
![image](/images/deploy/14.png)
기존 키페어 선택, 만들었던 키페어를 선택하고 체크박스에 체크 후 인스턴스 시작을 눌러준다.


### 인스턴스 접속

인스턴스를 성공적으로 만들었으니 접속할 차례다.

window cmd창을 열어 입력한다.
```
​ C​₩windows₩~ ​: ssh -i <pem파일(키파일) 경로> 유저명@ec2 public domain name
```
여기서 ssh가 무엇인지 궁금할 수 있는데, secure shell 이라 하여 다른 원격접속보다 좀 더 보안에 좋은 원격 접속 프로그램이다. 자세한 건 따로 구글링!

유저명의 경우는 대부분 ubuntu이다
![image](/images/deploy/15.png)
Downloads\EC2user1.pem 키페어 경로는 아까 다운받았던 pem파일의 경로.

퍼블릭 dns주소는 AWS EC2 메뉴에서 인스턴스 메뉴에 들어가면 하단에 나와있다.
![image](/images/deploy/16.png)
아까 전 pem파일의 권한을 정상적으로 주었다면, 아래처럼 뜰것이다.
![image](/images/deploy/17.png)
 권한을 정상적으로 주지 않았다면 이러한 에러가 뜰것이다.
![image](/images/deploy/18.png)
pem파일 권한을 꼭 확인하자.

자, 이렇게 하여 1 단계인 EC2를 생성하였다! 다음 단계인 EC2에 django를 올리자.

---
### Django application

우리가 처음 장고를 시작할 때 환경 구축을 했던 것과 동일하지만 OS의 차이이니 겁먹지말고 차근차근 밟아보자.

- 로케일 설정.

후에 여러가지 패키지나 프로그램등 locale을 필요로하는 것이 있기때문에
설치해준다.
```
​ ubuntu@ip-000-000-000-00​ ​: ~$ sudo nano /etc/default/locale
```
입력하여 연후
```
LC_CTYPE=”en_US.UTF-8”
LC_ALL=”en_US.UTF-8”
LANG=”en_US.UTF-8”
```
입력해준 후 재접속한다. 오탈자 꼭 확인해주자.

- 패키지 정보 업데이트
```
​ ubuntu@ip-000-000-000-00​ ​: ~$ sudo apt-get update
```
- 패키지 의존성 검사 및 업그레이드
```
​ ubuntu@ip-000-000-000-00​ ​: ~$ sudo apt-get dist-upgrade
```
주루룩 설치하면서 마지막엔 이런화면이 뜬다
![image](/images/deploy/19.png)

- python 패키지들을 관리하기 위한 pip 설치
```
​ ubuntu@ip-000-000-000-00​ ​: ~$ sudo apt-get install python-pip
```

- anaconda 설치
https://www.anaconda.com/download/#linux

링크로 이동하여 최신버전의 콘다를 설치해주자.
linux버전의 conda를 다운받게 되면 쉘 스크립트 파일이 하나 다운받아질 것이다. 확장자가 .sh인 파일 로컬에서 인스턴스로 파일을 보내자.

```
​ ubuntu@ip-000-000-000-00​ ​: ~$ sudo chown ubuntu:ubuntu /srv/
```
위처럼 입력하여 srv디렉토리 사용 전 소유권 가져오고, scp 명령어를 입력하여 인스턴스로 파일을 보내보자.

scp -i 키페어경로 -r 보낼폴더 유저명@퍼블릭dns:받을 경로
```
​ C​₩windows₩~ ​: scp -i downloads\ec2user1.pem -r
downloads/Anaconda3-2019.03-Linux-x86_64.sh
ubuntu@ec2-54-180-153-115.ap-northeast-2.compute.amazonaws.com​: /srv/
```

srv 폴더로 옮겨졌다면, 실행시켜주자.
```
​ ubuntu@ip-000-000-000-00​ ​: /srv$ bash Anaconda3-2019.03-Linux-x86_64.sh
```
![image](/images/deploy/20.png)
enter와 yes를 열심히 눌러주자
이 이후 설치가 잘 되었는지 확인해보자.
![image](/images/deploy/21.png)

conda 커맨드를 찾을 수 없다고 한다. 설치는 잘 마쳤는데 커맨드는 찾을 수 없다고 하는 경우 환경변수 설정이 생략되었을 수 있다.
이럴 경우 환경변수에 콘다를 추가해주자.
```
​ ubuntu@ip-000-000-000-00​ ​: ~$ sudo nano ~/.bashrc
```
```
export PATH=”/home/{username}ubuntu/anaconda3/bin:$PATH”
```
이후 빠져나와서 `source ~/.bashrc` 하여 활성화.
source 라는 명령어는 스크립트 파일을 수정한 후에 수정된 값을 바로 적용하기 위해 사용하는 명령어이다.
![image](/images/deploy/23.png)
잘 설치되었다.

srv폴더로 이동하여 우리가 만든 프로젝트를 땡겨오자.
```
​ ubuntu@ip-000-000-000-00​ ​: ~$ cd /srv
​ ubuntu@ip-000-000-000-00​ ​: /srv$ git clone https://your git hub repo
```
![image](/images/deploy/24.png)
잘 가져왔다.

자 이제, 가상환경 켜주고 requirements들을 설치해주자.


가상환경 생성

```
​ ubuntu@ip-000-000-000-00​ ​: /srv/glocal$ conda create -n run python=3.7
```
가상환경 활성화
```
​ (base) ubuntu@ip-000-000-000-00​ ​: /srv/glocal$ source activate run
```
python package 설치
```
​ (run) ubuntu@ip-000-000-000-00​ ​: /srv/glocal$ pip install -r requirements.txt
```

AWS EC2 관리화면으로 들어가, 보안 그룹으로 이동한다.
![image](/images/deploy/25.png)

규칙추가 -> 포트범위에 8080 입력 후 저장
![image](/images/deploy/26.png)
설정 중 보면 인바운드와 아웃바운드가 있는데 뭔지 궁금해서 찾아봤다.

인바운드는 외부에서 ec2인스턴스로 들어오는 트래픽이다. 예로 들면 사용자들의 요청이 되겠다.
아웃바운드는 ec2 에서 외부로 나가는 트래픽이다. 파일을 다운받거나, 내가 ssh를 이용하여 접속한다거나? 등등
```
​ (run) ubuntu@ip-000-000-000-00​ ​: /srv/glocal$ python manage.py runserver
```
입력후 내 퍼블릭 DNS로 접속하니 창이 뜨진 않았지만 GET요청을 받았다
![image](/images/deploy/27.png)
admin사이트로 이동을 하니 성공적으로 띄운다.
![image](/images/deploy/28.png)
이렇게 하여 AWS EC2 인스턴스에 우리의 장고를 올렸다.

다음 단계로 이동하자.

---
### WSGI setting

무엇을 쓸 것이냐,

https://paphopu.tistory.com/entry/WSGI%EC%97%90-%EB%8C%80%ED%95%9C-
%EC%84%A4%EB%AA%85-WSGI%EB%9E%80-%EB%AC%B4%EC%97%87%E
C%9D%B8%EA%B0%
django를 위한 wsgi 종류별 특징 및 왜 사용해야하는지,
너무 잘나와있다. 꼭 읽어보도록 하자.
본인은 gunicorn을 사용하였다.

### gunicorn 설치

가상환경 활성화
```
​ (base) ubuntu@ip-000-000-000-00​ ​: /srv/glocal$ source activate run
```

gunicorn 설치
```
​ (run) ubuntu@ip-000-000-000-00​ ​: /srv/glocal$ pip install gunicorn
```

구동테스트
```
​ (run) ubuntu@ip-000-000-000-00​ ​: /srv/glocal$ gunicorn --bind 0.0.0.0:8080 glocal.wsgi:applicaiton
```
![image](/images/deploy/29.png)

우리가 일일이 이 명령어를 치기엔 너무 귀찮다.
서비스로 등록해버리자.
```
​ (run) ubuntu@ip-000-000-000-00​ ​: /srv/glocal$ sudo nano /etc/systemd/system/gunicorn.service
```
에디터로 열었다면 아래 내용을 입력해주자!
```
[Unit]
Description=gunicorn daemon
After=network.target

[Service]
User=ubuntu
Group=ubuntu
WorkingDirectory=/srv/glocal/
ExecStart=/home/ubuntu/anaconda3/envs/run/bin/gunicorn \
--workers 3 \
--bind unix:/srv/glocal.sock \
glocal.wsgi:application

[Install]
WantedBy=multi-user.target
```

[Unit]
`Description` : Service에 대한 설명
`After` : 이서비스는 network 활성 후에 작동(이 옵션이 없을 경우 os재시작시 실행되지 않을 수 있음)

[Service]
`User` : Service를 사용할 유저
`Group` : Service를 사용할 그룹
`WorkingDirectory` : 우리가 배포할 프로젝트 디렉토리
`ExecStart` : gunicorn에 대한 설정 /home/ubuntu(linux 계정)/anaconda3/envs/run(가상환경 이름)/bin/gunicorn : 런서버될 가상환경 디렉토리 지정
`--workers` : 프로세스의 수
`--bind `: nginx와 통신할 방법 현재는 unixsocket
glocal.wsgi:application glocal 디렉토리의 wsgi.py의 application을 불러오겠다.
[Install]
`Wantedby `: 런레벨 지정

ubuntu service 관한 설정이 궁금하다면 ​https://fmd1225.tistory.com/

저장하고 에디터에서 빠져나오자.

이제 등록하고 시작할 차례.
서비스 등록
```
​ (run) ubuntu@ip-000-000-000-00​ ​: /srv/glocal$ sudo systemctl enable gunicorn
```
서비스 시작
```
​ (run) ubuntu@ip-000-000-000-00​ ​: /srv/glocal$ sudo systemctl start gunicorn
```
![image](/images/deploy/30.png)
등록이 잘되었다.
```
​ (run) ubuntu@ip-000-000-000-00​ ​: /srv/glocal$ systemctl status gunicorn
```
![image](/images/deploy/31.png)
이제 gunicorn에서 우리의 프로젝트를 바라보고 있고,
nginx랑 unix socket으로 통신할 것이다. 라고 알려주었다.

로그는 /var/log/syslog에서 확인가능하다.
그다음단계는 web server가 gunicorn을 바라볼 수 있도록 세팅하자.

---
### WEB SERVER

nginx 설치

```
​ (run) ubuntu@ip-000-000-000-00​ ​: /srv/glocal$ sudo apt-get install nginx
```
nginx 셋팅을 위해
/etc/nginx/sites-enabled/이름(nginx.conf)라고 만들어주자
```
​ (run) ubuntu@ip-000-000-000-00​ ​: /srv/glocal$ sudo nano /etc/nginx/sites-enabled/nginx.conf
```
```
server {
	listen 80;
	server_name your_domain;

	location / {
		proxy_pass http://unix:/srv/glocal.sock;
		include proxy_params;
	}
}
```
80 번 포트를 듣고, server_name 에 도메인을 알려주고 아까 gunicorn에서 설정했던
socket의 위치를 적어준다.
이렇게 작성해주고, gunicorn 할때 처럼 똑같이 실행시켜주자

등록 및 시작
```
​ (run) ubuntu@ip-000-000-000-00​ ​: /srv/glocal$ sudo systemctl daemon-reload
​ (run) ubuntu@ip-000-000-000-00​ ​: /srv/glocal$ sudo systemctl restart nginx
```

상태 확인
```
​ (run) ubuntu@ip-000-000-000-00​ ​: /srv/glocal$ systemctl status nginx
```
![image](/images/deploy/32.png)
```
​ (run) ubuntu@ip-000-000-000-00​ ​: /srv/glocal$ cat /var/log/nginx/error.log
```
![image](/images/deploy/33.png)
에러가 났을 경우 저위치에 로그를 작성하니 참고하자.

---
### Django collectstatic

하지만 보아하니 static에 관한 설정이 빠져서나온다...
구글링을 열심히 해보니, 우리가 개발환경에서 runserver할 경우 장고의 runserver가 알아서 static을 확인해준다. 하지만 우린 nginx를 사용하고 있으니 static을 알수가 없다.

`settings.py` 설정
```python
# settings.py

STATIC_URL='/static/'
STATIC_ROOT = os.path.join(BASE_DIR, 'statics')
```
`manage.py` 기능 중 collectstatic은 모든 정적파일들을
STATIC_ROOT 경로에 모두 모으게된다. 위의 settings의 따르면 static url은 ‘static’이라는 디렉토리로 지정해주었고, 현재 STATIC_ROOT의 경로대로라면 모든 static디렉토리에 있는 것들을 manage.py가 있는 위치의 statics라는 디렉토리를 생성해 모든 정적 파일을 모아줄 것이다.

다음의 명령어를 실행해 static파일들을 모아주자.
```
python manage.py collectstatic
```
실행 전
![image](/images/deploy/34.png)
실행 후
![image](/images/deploy/35.png)

이렇게 하여 static을 모아줬으니 nginx는 이걸 받고 사용자의 요청에 따라 뱉게끔
해주면 되겠다.
```
​ (run) ubuntu@ip-000-000-000-00​ ​: /srv/glocal$ sudo nano /etc/nginx/sites-enabled/nginx.conf
```
conf 파일을 열어 추가해주자.
```
server {
	listen 80;
	server_name your_domain.com;

	location / {
		proxy_pass http://unix:/srv/glocal.sock;
		include proxy_params;
	}
	
	location /static/ {
		alias /srv/glocal/statics/;
	}
}
```

/static/ 이라는 요청이 오면,
/srv/glocal/statics/에서 찾아 뱉겠다. 이런 의미다.
```
​ (run) ubuntu@ip-000-000-000-00​ ​: /srv/glocal$ sudo systemctl daemon-reload
​ (run) ubuntu@ip-000-000-000-00​ ​: /srv/glocal$ sudo systemctl restart nginx
```
재시작해주고 확인해보자.
![image](/images/deploy/36.png)
만일 collectstatic 중 다른 앱이지만 static안에 같은경로이고 이름이 똑같은 파일이 있다면 어떻게 될까?
![image](/images/deploy/37.png)
![image](/images/deploy/38.png)
이런 식으로 glocal/home/static/css안에 주루룩 파일이 있고, 똑같이 glocal/todo_list/static/css안에 같은 파일들이 있다. 이 경우에 static 파일을 모은다면?
![image](/images/deploy/39.png)

`대상 경로 'css \ zabuto_calendar.css'가있는 다른 파일을 찾았습니다. 처음 발견된 파일 만 수집되므로 무시됩니다. 원하는 파일이 아닌 경우 모든 정적 파일에 고유한 경로가 있는지 확인하십시오.`

번역을 돌리니 이렇다고 한다.
statics 디렉토리안에 들어가니 모아지긴 모아졌다.
![image](/images/deploy/40.png)
결론은 django가 처음 발견한 파일만 모아주고, 중복되는 것들은 모아주지 않았다. 이름이 똑같지만 내용이 다를 경우 하나를 못 가져오는 오류를 가져올 수 있을 것 같다. 이것도 한번 테스트해보자.
accounts앱의 css
![image](/images/deploy/41.png)
todo앱의 css
![image](/images/deploy/42.png)
home앱의 css
![image](/images/deploy/43.png)

collectstatic 결과 터미널에 찍히는 오류는 같다.
![image](/images/deploy/44.png)
모은 css는 누구의 것을 들고왔을까.
![image](/images/deploy/45.png)
accounts의 css를 들고왔다.
아마 Installed apps 순으로 들고오는 것 같다.

어떤 파일들이 이름이 겹치는 지 확인 하려면
```
​ (run) ubuntu@ip-000-000-000-00​ ​: /srv/glocal$ python manage.py findstatic staticfile <찾으려는 파일>
```
![image](/images/deploy/46.png)

하면 이름이 동일한 파일의 경로까지 알려준다.


