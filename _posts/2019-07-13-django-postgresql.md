---
layout: post
title: Django postgresql 사용하기
tags:
- django
- postgresql
---


postgresql 설치
```console
(base)ubuntu@--.--.--.--/ \$ sudo apt-get install postgresql
```
설치 확인
```console
(base)ubuntu@--.--.--.--/ \$ dpkg -l | grep postgres
```
설치가 잘 되었다면 database를 만들어보자.
postgres 설정을 위한 user 변경
```console
(base)ubuntu@--.--.--.--/ \$ sudo su postgres
```
database 접속
```console
(base)postgres@--.--.--.--/ \$ psql
```
접속이 다 되었다면 생성하자.

생성
```console
postgres=\# create database \<DB이름\>;
CREATE DATABASE
```
만든 DB를 위한 유저 설정
```console
postgres=# create user <유저이름> with password ‘password’;

postgres=# alter role <유저이름> set client\_encoding to ‘utf-8’;

postgres=# alter role root set timezone to ‘Asia/Seoul’;

postgres=# grant all privileges on database <DB이름> to <유저이름>;
```
![](/images/postgresql/1.png)

이렇게 하면 설정이 마무리 된다.
django 설정
설정 전에 필요한 것이 있다.
가상환경에 진입해서, psycopg2라는 것을 설치해주자
psycopg2 는 python과 postgresql을 연동하기 위한 python 라이브러리라고 한다.
```
(base)ubuntu@--.--.--.--/ \$ pip install psycopg
```
django의 우리 project 폴더로 들어가 settings.py의 DATABASE 내용을 아래와 같이 바꿔준다.
```python
DATABASE ={
    ‘default:{
        ‘ENGINE’:’django.db.backends.postgresql’,
        ‘NAME’:’DATABASE 이름’,
        ‘USER’:’유저이름’,
        ‘PASSWORD’:’유저 비밀번호’,
        ‘HOST’:’localhost’,
        ‘PORT’:’포트(default 포트는 5432라고 한다)’
    }
}
```
  

이렇게 하면 설정이 완료된다.
내 설정은 전에 만들었던 secret.json파일에 넣어놓았다.
![](/images/postgresql/2.png)

  

내 계정과 비밀번호가 다 나와있으니 감춰주는게 정상일 듯 싶다.
새로 세팅을 했으니 마이그레이션해주자.
DB migrations, migrate
```
(run)ubuntu@--.--.--.--/srv/glocal \$ python manage.py makemigrations
(run)ubuntu@--.--.--.--/srv/glocal \$ python manage.py migrate
```
