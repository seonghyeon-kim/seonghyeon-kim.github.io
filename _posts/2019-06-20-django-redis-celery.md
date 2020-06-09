---
layout: post
title: Django redis celery
tags:
- django
- redis
- celery
---
### 개요

개발을 하다보면, client에겐 중요하지 않지만 시간이 너무 오래걸리는 작업들이 생긴다. 뭐 예를 들자면, 여러명에게 알림을 보낸다거나 무슨 문서를 만들어서 저장한다거나.. 굳이 client의 시간을 뺏지 않아도 된다. 이러한 작업들을 background에서 돌릴 수 있는데 그것이 바로 redis, celery다.

---
#### celery?

이 녀석은 그냥 단순히 일을 하는 일꾼이다. 받은 일들을 que
ue에 쌓아두고 일을 한다.

#### redis?

Redis는 실제 컴퓨터 메모리를 이용한 캐쉬다. DB를 이용하는 것보다 캐쉬를 사용하는 것이기 때문에 속도가 훨씬 빠르다는 장점이 있다.

---
### 설치

- celery 설치
```console
(conda) gimseonghyeon-ui-MacBook-Pro:project kimseonghyeon$ pip install 'celery[redis]'
```
- redis 설치
window에선 설치가 불가능하다고 한다.
mac이나 linux에서 사용을 권장한다고 한다.
```console
# linux
[root@managernode ~]# wget http://download.redis.io/redis-stable.tar.gz
...
100%[======================================>] 2,260,604 1.09MB/s in 2.0s

2019-09-05 17:20:27 (1.09 MB/s) - ‘redis-stable.tar.gz’ saved [2260604/2260604]
[root@managernode ~]# tar xvzf redis-stable.tar.gz
[root@managernode ~]# cd redis-stable
[root@managernode ~]# make
[root@managernode ~]# sudo make install
[root@managernode ~]# redis-server
...
```
![image](/images/redis/1.png)

이런 박스가 뜨면서 redis server를 구동시키는데 성공했다.
mac은 brew manager로 설치가 가능하니 아래를 참고하면 될 것 같다.
```console
# mac
(conda) gimseonghyeon-ui-MacBook-Pro:project kimseonghyeon$ brew install redis
(conda) gimseonghyeon-ui-MacBook-Pro:project kimseonghyeon$ brew services start redis
(conda) gimseonghyeon-ui-MacBook-Pro:project kimseonghyeon$ brew services stop redis
(conda) gimseonghyeon-ui-MacBook-Pro:project kimseonghyeon$ brew services restart redis
```

---
### config
이제 django에서 사용할 차례다.

settings.py의 위치에 있는 `__init__.py` 로 이동해 아래처럼 작성해보자.
```python
from .tasks import app as celery_app

__all__=['celery_app']
```

이제 `settings.py` 로 가서 추가해주자.
```python
BROKER_URL='redis://localhost:6379/0'
CELERY_RESULT_BACKEND='redis://localhost:6379/0'
```
프로젝트 폴더에 `celery.py` 라는 이름으로 파일을 하나 만들고 아래처럼 작성해주자.
```python
from __future__ import absolute_import, unicode_literals

import os

from celery import Celery

# `celery` 프로그램을 작동시키기 위한 기본 장고 세팅 값을 정한다. 
os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'glocal.settings')

app = Celery('glocal')

# namespace='CELERY'는 모든 셀러리 관련 구성 키를 의미한다. 반드시 CELERY라는 접두사로 시작해야 한다. 
app.config_from_object('django.conf:settings', namespace='CELERY')

# 장고 app config에 등록된 모든 taks 모듈을 불러온다. 
app.autodiscover_tasks()
```



이제 할 일을 알려줄 차례다.
앱 폴더에 가서 `tasks.py` 라는 이름으로 파일을 하나 만들고 아래처럼 작성해보자.
```python
from __future__ import absolute_import, unicode_literals
from celery import shared_task

@shared_task
def plus(x, y):
    return  x+y
```

여기까지 하면 실행시킬 준비가 완료되었다.
celery를 켜고 한번 실행시켜보자. 아래처럼 명령어를 치면 되는데, 주의할 점은 `manage.py` 위치에서 실행 시켜주면 된다.
```console
(conda) gimseonghyeon-ui-MacBook-Pro:project kimseonghyeon$ celery -A glocal worker -l info
```
