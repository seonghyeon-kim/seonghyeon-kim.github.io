---
layout: post
title: Django Tutorial!
tags:
  - django
  - tags
---

# Django Tutorial

### 목표
django라는 web framework을 사용하여 상담 접수를 받는 웹페이지를 만들어보자!

---
### 개발환경세팅
먼저 conda를 설치하자. 자신의 os에 맞춰서 다운로드 받아주면 된다.
https://anaconda.com/download

설치가 완료되었다면 conda prompt를 열어 conda -V를 실행해 현재 버전을 확인해보자. conda prompt를 열어 가상환경을 하나 만든다. 아래 conda-name은 가상환경의 이름으로 자신이 적절한 이름을 지어서 만들어 주자.
나는 conda라는 이름으로 진행하려고 한다.
```console
foo@bar:~$ conda create -n conda-name python=3.7

Proceed ([y]/n)? y
  
Preparing transaction: done
Verifying transaction: done
Executing transaction: done
#
# To activate this environment, use
#
# $ conda activate conda
#
# To deactivate an active environment, use
#
# $ conda deactivate
```

명확하게 설치가 끝났다면 확인해본다.
```console
foo@bar:~$ conda env list

# conda environments:
#
base                  *   //anaconda3
conda                     //anaconda3/envs/conda
```
conda env list라는 명령어를 터미널에 입력하게 되면 자신의 가상환경의 목록을 확인할 수 있다. conda 라고 생성된 것을 확인할 수 있다.

다음으로 가상환경을 활성화 해보자.
`윈도우의 경우에는 source를 생략하고 쳐준다.`
```console
(base) foo@bar:~$ source activate conda-name
(conda) foo@bar:~$
```

다음은 장고를 설치하자.
```console
(conda-name) foo@bar:~$ pip install django
Collecting django
Downloading Django-3.0.6-py3-none-any.whl (7.5 MB)
|████████████████████████████████| 7.5 MB 233 kB/s
Collecting asgiref~=3.2
Using cached asgiref-3.2.7-py2.py3-none-any.whl (19 kB)
Collecting pytz
Downloading pytz-2020.1-py2.py3-none-any.whl (510 kB)
|████████████████████████████████| 510 kB 275 kB/s
Collecting sqlparse>=0.2.2
Using cached sqlparse-0.3.1-py2.py3-none-any.whl (40 kB)
Installing collected packages: asgiref, pytz, sqlparse, django
Successfully installed asgiref-3.2.7 django-3.0.6 pytz-2020.1 sqlparse-0.3.1
```

설치가 잘되었는지 확인하려면 
```console
(conda-name) foo@bar:~$ pip list
Package  Version
---------- -------------------
asgiref  3.2.7
certifi  2020.4.5.1
Django 3.0.6
pip  20.0.2
pytz 2020.1
setuptools 46.4.0.post20190513
sqlparse 0.3.1
wheel  0.34.2
```
위처럼 list에 django가 확인되었다면 설치가 완료된 것이다.

---
### django 시작하기
django project를 시작해보자.
```console
(conda-name) foo@bar:~$ django-admin startproject 프로젝트명
```
나는 프로젝트명을 임의로 tutorial 이란 이름으로 시작하겠다. 

이후 생성된 프로젝트 구조를 한번 살펴보자.
```console
-- /tutorial -- manage.py
             -- /tutorial -- __init.py
                          -- asgi.py
                          -- settings.py
                          -- urls.py
                          -- wsgi.py             
```
저 startproject 라는 명령어 하나로 django는 프로젝트 시작을 위한 파일들을 만들어준다.

자 여기까지 준비가 되었다면, 서버를 구동시켜보자.
```console
(conda-name) foo@bar:~$ python manage.py runserver
Watching for file changes with StatReloader

Performing system checks...

System check identified no issues (0 silenced).

You have 17 unapplied migration(s). Your project may not work properly until you apply the migrations for app(s): admin, auth, contenttypes, sessions.
Run 'python manage.py migrate' to apply them.

May 13, 2019 - 15:12:03
Django version 3.0.6, using settings 'tutorial.settings'
Starting development server at http://127.0.0.1:8000/
Quit the server with CONTROL-C.
```
우선 무시하고 자신의 웹브라우저에 url창에 127.0.0.1:8000이라고 입력후 접속해본다.

멋진 로케트가 보이면서 The install worked successfully! Congratulations!

django project 생성을 마쳤다.
django도 축하해준다.

이후 터미널로 돌아와 
```console
Quit the server with CONTROL-C.
```

컨트롤키와 c를 동시에 눌러줍니다.

여기서 잠시 터미널에 쳤던 명령어들을 한번 알아보자.
python 이란 명령어는 python 파일을 실행할때 쓰는 명령어다.
그 다음인 manage.py는 django 프로젝트 생성 시 django가 자동으로 만들어준 파이썬 파일이다.
아까 어마무시했던 경고는 django 서버를 실행 시에는 앱들이 있는데 그 앱들이 사용하는 DB 테이블들을 찾을 수 없다고 django가 우리에게 알려준 것이다.
migrate은 이러한 DB 테이블들을 자동으로 생성해주는 명령어다.

---
### django app 만들기
자 그럼 저번시간에 만들어 놓은 개발환경에 들어와 앱을 만들기만 하면 된다.
```console
(conda) foo@bar:~$ python manage.py startapp apply
```

위 명령어 한줄로 django는 앱을 기본적으로 만들때 필요한 대부분의 것을 만들어준다.

디렉토리 구조를 살펴보면
```console
-- /tutorial -- manage.py
             -- /tutorial -- __init__.py
                          -- asgi.py
                          -- settings.py
                          -- urls.py
                          -- wsgi.py
             -- db.sqlite3
             -- /apply -- __init__.py
                          -- admin.py
                          -- apps.py
                          -- migrations
                          -- models.py
                          -- tests.py
                          -- views.py
```
처럼 바뀌었을 것이다.

이제 `settings.py` 파일을 열어 installed_apps 부분을 수정해준다.
```python
INSTALLED_APPS = [  
    'django.contrib.admin',  
    'django.contrib.auth',  
    'django.contrib.contenttypes',  
    'django.contrib.sessions',  
    'django.contrib.messages',  
    'django.contrib.staticfiles',
    'apply',   
]
```
apply를 추가해준다.

apply라는 앱은 상담을 접수받는 앱이다.

상담을 받을때 필요한 정보들은?

간단하게 이름, 휴대폰번호, 신청내용 정도?
이 내용을 `models.py`를 열어 추가해주자.
```python
class Apply(models.Model):  
    name = models.CharField(verbose_name="이름",max_length=255)  
    phone = models.CharField(verbose_name="전화번호", max_length=20)  
    content = models.CharField(verbose_name="신청내용", max_length=255)  
```
 
저장을 한다면 우린 apply 앱의 모델을 다 작성하게 되었다.
그럼 만들어진 모델에 따른 DB 테이블을 만들어야 한다.

터미널을 열어 python manage.py makemigrations apply

실제로 apply의 앱 폴더를 들여다 보면 migrations 폴더안에 0001_initial.py 파일이 생겨났을 것이다.

자, DB 테이블까지 생성을 마쳤다.


이제 데이터를 입력해보자.
데이터를 입력하는 방법에는 정말 많지만, 어드민 페이지에서 기본 데이터를 입력하는 방법을 알아보도록 하자.
일단 생성된 apply 앱 폴더안에 admin.py 파일을 열어 수정한다.

```python
from django.contrib import admin
from .models import Apply 
  
# Register your models here.  
class ApplyAdmin(admin.ModelAdmin):  
    list_display = ('name', 'phone', 'content')  
  
admin.site.register(Apply, ApplyAdmin)
```

작성이 끝났다.

admin페이지에 접속하기 위해 admin 유저를 만들어보자.
터미널에서 python manage.py createsuperuser 를 실행하면 이름, 메일, 패스워드를 입력하라고 나온다.
```console
(conda-name) foo@bar:~$ python manage.py createsuperuser

Username (leave blank to use 'kimseonghyeon'): admin
Email address:
Password:
Password (again):
**The password is too similar to the username.**
**This password is too short. It must contain at least 8 characters.**
**This password is too common.**
Bypass password validation and create user anyway? [y/N]: y
Superuser created successfully.
```

python manage.py runserver 0:8000

데이터를 입력해보자!
![image](./images/tutorial/)

---

여기까지 django framework를 사용하여, 프로젝트를 만들고, 웹앱을 만드는 과정을 해보았고 admin을 이용해 데이터까지 입력시켜보았다.
