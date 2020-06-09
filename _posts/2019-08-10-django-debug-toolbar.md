---
layout: post
title: django debug toolbar 사용하기
tags:
- django
- test
---

### 개요
첫 프로젝트 개발을 마치고 테스트 과정에서 속도를 테스트를 하기 위해 알아보던 중 debug toolbar라는 것이 있어 프로젝트에 적용하는 방법을 기록한다.

---
- 패키지 설치

먼저 패키지를 설치해준다.
```console
(conda) gimseonghyeon-ui-MacBook-Pro:project kimseonghyeon$ pip install django-debug-toolbar
```
<br>

- `settings.py` 수정

다음으로 settings.py를 열고 추가해준다.
installed apps 부분에 debug_toolbar를 추가한다.
```python
INSTALLED_APPS = [  
    'django.contrib.admin',  
    'django.contrib.auth',  
    'django.contrib.contenttypes',  
    'django.contrib.sessions',  
    'django.contrib.messages',  
    'django.contrib.staticfiles',  
    # install app  
    'rest_framework',  
    # custom app  
    'apply',  
    # debugtoolbar
    'debug_toolbar',  
]
```
그후 middleware에 debug_toolbar.middleware.DebugToolbarMiddleware 를 추가해준다.
```python
MIDDLEWARE = [  
    'django.middleware.security.SecurityMiddleware',  
    'django.contrib.sessions.middleware.SessionMiddleware',  
    'django.middleware.common.CommonMiddleware',  
    'django.middleware.csrf.CsrfViewMiddleware',  
    'django.contrib.auth.middleware.AuthenticationMiddleware',  
    'django.contrib.messages.middleware.MessageMiddleware',  
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
    # 추가해줄부분
    'debug_toolbar.middleware.DebugToolbarMiddleware',
]
```
그후 아래에 internal ips를 추가한다.
거의 대부분 로컬에서 테스트를 하기 때문에 127.0.0.1을 적었는데 만약 서버에서 테스트할 일이 있다면 서버의 ip 주소를 적어주면 된다.
```python
# settings.py
INTERNAL_IPS = ('127.0.0.1',)
```
<br>

- `urls.py` 수정

이제 프로젝트의 urls.py에 가서 아래와 같이 추가해준다.
```python
from django.conf import settings
from django.conf.urls import url
'''
urlpatterns = [
    '***'
]
'''
if settings.DEBUG:
    import debug_toolbar
    urlpatterns += [
        url(r'^__debug__/', include(debug_toolbar.urls)),
    ]
   ```

그 후 웹에 접속해보면 아래와 같은 사진을 확인해볼 수 있다.
![image](/images/debugtoolbar/1.png)
![image](/images/debugtoolbar/2.png)
이렇게 하여 적용을 마쳤다.

---
toolbar 사용 시엔 속도가 조금 느려질 수 있다.
페이지 별로 접속해보면서 query수나 time을 체크해볼 수 있어 병목지점을 찾기 쉽다.
