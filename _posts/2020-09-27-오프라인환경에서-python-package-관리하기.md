---
layout: post
title: python 오프라인 환경에서 package 관리하기
tags:
- pip
- python
---


## 개요

개발환경을 구축할 때, 보안상의 이유로 내부망을 사용하여 개발하는 경우가 있다. 외부망을 사용하게 되면 절차를 밟아야 하는데 그 절차가 굉장히 복잡하기 때문에 오프라인 상태에서 패키지를 관리할 수 있는지 테스트해보았다.

## 테스트 케이스

아래의 케이스는 오프라인 상태에서 진행하는 것을 전제로 하였다.

1. conda env 생성 가능 여부
2. conda env 패키지 추가 가능 여부

## 1. conda env 생성 가능여부

```bash
(base) C:\Users\user\Desktop\front_ng\todos>conda env list
# conda environments:
#
base                 *   C:\Users\user\anaconda3
todo                     C:\Users\user\anaconda3\envs\todo
```

먼저 생성하기 전 내 Local PC의 env 들이다.

이제 기존 사용하던 명령어로 env를 생성해보자.

```bash
(base) C:\Users\user\Desktop\front_ng\todos>conda create -n testenvs
Collecting package metadata (current_repodata.json): done
Solving environment: done

CondaHTTPError: HTTP 000 CONNECTION FAILED for url <https://repo.anaconda.com/pkgs/main/win-64/current_repodata.json>
Elapsed: -

An HTTP error occurred when trying to retrieve this URL.
HTTP errors are often intermittent, and a simple retry will get you on your way.

If your current network has https://www.anaconda.com blocked, please file
a support request with your network engineering team.

'https://repo.anaconda.com/pkgs/main/win-64'

(base) C:\Users\user\Desktop\front_ng\todos>
```

역시 예상대로 Http connection error를 일으키면서 생성이 되질 않았다.

## 2. conda env 패키지 추가 가능 여부

```bash
(todo) C:\Users\user>pip install django-page
WARNING: Retrying (Retry(total=4, connect=None, read=None, redirect=None, status=None)) after connection broken by 'NewConnectionError('<pip._vendor.urllib3.connection.HTTPSConnection object at 0x000001D8034AE220>: Failed to establish a new connection: [Errno 11001] getaddrinfo failed')': /simple/django-page/
WARNING: Retrying (Retry(total=3, connect=None, read=None, redirect=None, status=None)) after connection broken by 'NewConnectionError('<pip._vendor.urllib3.connection.HTTPSConnection object at 0x000001D8034AEBB0>: Failed to establish a new connection: [Errno 11001] getaddrinfo failed')': /simple/django-page/
WARNING: Retrying (Retry(total=2, connect=None, read=None, redirect=None, status=None)) after connection broken by 'NewConnectionError('<pip._vendor.urllib3.connection.HTTPSConnection object at 0x000001D8034B6670>: Failed to establish a new connection: [Errno 11001] getaddrinfo failed')': /simple/django-page/
WARNING: Retrying (Retry(total=1, connect=None, read=None, redirect=None, status=None)) after connection broken by 'NewConnectionError('<pip._vendor.urllib3.connection.HTTPSConnection object at 0x000001D8034B66D0>: Failed to establish a new connection: [Errno 11001] getaddrinfo failed')': /simple/django-page/
WARNING: Retrying (Retry(total=0, connect=None, read=None, redirect=None, status=None)) after connection broken by 'NewConnectionError('<pip._vendor.urllib3.connection.HTTPSConnection object at 0x000001D8034B6340>: Failed to establish a new connection: [Errno 11001] getaddrinfo failed')': /simple/django-page/
ERROR: Could not find a version that satisfies the requirement django-page (from versions: none)
ERROR: No matching distribution found for django-page
```

역시나 동일하게 설치가 되지 않았다.

---

## 해결책

기존에 사용하던 conda env 폴더안의 파일들을 복사하여 붙여 넣어주면 다.

```bash
(todo) C:\Users\user\anaconda3\envs>ls -al
total 208
drwxr-xr-x 1 user 197121 0 Sep 16 16:02 .
drwxr-xr-x 1 user 197121 0 Aug 24 11:27 ..
-rw-r--r-- 1 user 197121 0 Aug 24 13:54 .conda_envs_dir_test
drwxr-xr-x 1 user 197121 0 Sep  1 09:20 todo
```

anaconda가 설치된 위치에서 envs로 이동후 파일들을 보면 만들었던 env들이 보인다.

이 위치에서 원하는 이름으로 폴더를 하나 만들어준다.

```bash
(todo) C:\Users\user\anaconda3\envs>mkdir testenvs

(todo) C:\Users\user\anaconda3\envs>ls -al
total 208
drwxr-xr-x 1 user 197121 0 Sep 16 16:05 .
drwxr-xr-x 1 user 197121 0 Aug 24 11:27 ..
-rw-r--r-- 1 user 197121 0 Aug 24 13:54 .conda_envs_dir_test
drwxr-xr-x 1 user 197121 0 Sep 16 16:05 testenvs
drwxr-xr-x 1 user 197121 0 Sep  1 09:20 todo
```

기존 사용하던 todo env의 모든 파일들을 복사해서 만든 폴더로 옮긴다.

```bash
(todo) C:\Users\user\anaconda3\envs>cd todo

(todo) C:\Users\user\anaconda3\envs\todo>cp -r * ../testenvs
(todo) C:\Users\user\anaconda3\envs>conda env list
# conda environments:
#
base                     C:\Users\user\anaconda3
testenvs                 C:\Users\user\anaconda3\envs\testenvs
todo                  *  C:\Users\user\anaconda3\envs\todo

(todo) C:\Users\user\anaconda3\envs>
```

복사 후 env list를 확인해보니 아까 만든 폴더의 이름으로 env가 생성되다.

```bash
(todo) C:\Users\user\anaconda3\envs>conda activate testenvs

(testenvs) C:\Users\user\anaconda3\envs>pip list
Package                      Version
---------------------------- -------------------
asgiref                      3.2.10
certifi                      2020.6.20
Django                       2.2.16
django-cors-headers          3.5.0
django-elasticsearch         0.5
django-elasticsearch-dsl     7.1.4
django-elasticsearch-dsl-drf 0.20.8
django-nine                  0.2.3
django-request-logging       0.7.2
djangorestframework          3.11.1
...
```

env 활성화 후 package list들을 확인해보니 기존 사용하던 package들도 잘 옮겨졌다.

이제 django server도 활성화해보자.

```bash
(testenvs) C:\Users\user\anaconda3\envs>cd C:\Users\user\Desktop\front_ng\todos

(testenvs) C:\Users\user\Desktop\front_ng\todos>python manage.py runserver 8080
Watching for file changes with StatReloader
Performing system checks...

System check identified no issues (0 silenced).
September 16, 2020 - 16:24:34
Django version 2.2.16, using settings 'todos.settings'
Starting development server at http://127.0.0.1:8080/
Quit the server with CTRL-BREAK.
```

server도 성공적으로 띄웠다.

## 결론

오프라인 상태 또는 외부망을 사용할 수 없는 경우에는 conda 파일들을 복사하여 옮겨주면 환경을 동일하게 사용이 가능하다.