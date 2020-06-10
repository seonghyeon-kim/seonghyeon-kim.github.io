---
layout: post
title: letsencrypt ssl 사용 및 갱신
tags:
- ssl
- letsencrypt
- certbot
- https
---

### 개요
우리 팀에서 만든 웹앱 중 지도 api를 사용하는 것이 있었는데, https를 사용하지 않으면 현재 위치를 불러올 수 없는 상황이 생겼다. 이러한 이유로 서버에 https를 설정하였다.

 ![image](/images/ssl/image2.png)

차이점에 대한 참고사이트: [https://nhj12311.tistory.com/83](https://www.google.com/url?q=https://nhj12311.tistory.com/83&sa=D&ust=1591631519233000)

---  

### SSL 인증서 발급받고 설정

letsencrypt 사용하여 SSL 인증서 발급받기
무료 인증서 발급이 가능하고, 발급방법이 간단하여 수많은 인증서 중 택했고 중요한 결제나 그런 것들이 필요없어서 무료인 것을 택했다. letsencrypt 를 이용하게 되면 certbot이라고 조금더 발급을 편하게끔 만든 프로그램이 있어 사용하여 발급받았다.

certbot 설치
```
(base)ubuntu@--.--.--.--/srv\$ sudo apt-get update
(base)ubuntu@--.--.--.--/srv\$ sudo apt-get install software-properties-common
(base)ubuntu@--.--.--.--/srv\$ sudo add-apt-repository ppa:certbot/certbot
(base)ubuntu@--.--.--.--/srv\$ sudo apt-get update
```
설치 전 앞서 결정해야하는 것이 있다.

1.  Standalone 방식

이 방법은 Certbot이 간이 웹 서버를 돌려 도메인 인증 요청을 처리하는 방식이다. 하지만 인증용 간이 서버가 80,443번 포트를 사용하기 때문에 운영중인 서버가 해당포트를 쓰게 되면, 포트가 겹치기 때문에 서버를 내려야한다.

  

2.  Webroot 방식

이방법은 도메인 인증을 위해 외부에서 접근 가능한 경로를 제공하고, let’s encrypt 측에서 해당 경로로 접속해 인증을 하는 방식이다. 

나는 아직 오픈 전이므로 1번 방식으로 발급받겠다.

**certbot 설치**
```
(base)ubuntu@--.--.--.--/letsencrypt \$ sudo apt-get install certbot
```
발급받기
```
(base)ubuntu@--.--.--.--/letsencrypt \$ certbot certonly --standalone -d \<도메인이름\>
```
![image](/images/ssl/image5.png)

인증서 만료 시 안내받을 메일만 적어주면 된다.
우리는 경로를 지정을 안했기때문에 default 위치인 /etc/letsencrypt/live/도메인이름/ 에 생겼을 것이다 확인해보자

참, 그리고 AWS ec2로 연습해보는 사람들은 아쉽게도 안되는 듯 하다. EC2 퍼블릭 도메인으로 연결 하였더니 amazonaws.com이 블랙리스트에 있다고 한다. 무조건 자신의 도메인 이름을 사용하라고 나와있다. 임시적으로 사용하는 도메인인 걸 알아서 그렇다고 한다. 발급받는 과정은 간단하다.
![image](/images/ssl/image4.png)

잘 생성되었다.

이제 Nginx에서 http 의 default 포트인 80번포트에서 https의 포트 443번으로 바꿔준후 ssl 인증을 걸어주자.
```
server {
    listen 443;
    server_name your_domain;
    
    ssl on;
    ssl_certificate /etc/letsencrypt/live/domain/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/domain/privkey.pem;

    location / {
        proxy\_pass http://unix:/srv/socket_name.sock;
        include proxy\_params;
        }
    
    '''
```
  

SSL 인증서 갱신 방법과 자동 갱신 설정

갱신 전에 갱신이 가능한지 파악할 수 있는 옵션
```
(base)ubuntu@--.--.--.--/letsencrypt \$ certbot-auto renew --dry-run
```
  

![](images/image3.png)

이렇게 ‘Cert not yet due for renewal’ 이라고 뜨면 아직 갱신하기 이르다는 뜻이다. 인증서 갱신은 만료일 15일 이내에만 진행가능하다.

갱신 명령어
```
(base)ubuntu@--.--.--.--/letsencrypt \$ certbot-auto renew
```
  

자동갱신을 위해서 crontab으로 관리해보자.
```
(base)ubuntu@--.--.--.--/letsencrypt \$ crontab -e
```
  

```
# crontab
0 0 10,15 2,5,8,11 * /letsencrypt/certbot-auto renew --pre-hook "systemctl stop nginx.service" --post-hook "systemctl start nginx.service"
```
본인은 8월 19일에 발급을 받아서 2,5,8,11월에 10,15일 0시 0분에 갱신하게끔 걸어두었다. 
그리고 80번 포트가 사용 중일 경우 갱신이 불가하다. 하지만 `--pre-hook "systemctl stop nginx.service" --post-hook "systemctl start nginx.service"` 이 옵션을 붙여주면 갱신하기 전 nginx를 종료하고 갱신이 끝나면 nginx를 다시 시작시켜준다.
