---
layout: post
title: Django custom command
tags:
- django
---
### 개요
github webhook 을 이용해 git에 push하게 되면 자동으로 서버에 반영이 되게끔 하는 방법을 찾다가 django에 command를 custom할 수 있다는 글을 보고 따라 해본다.

---
### django command 확인
command를 만들기 이전에 먼저, django의 기본으로 제공하는 command와 겹치면 안되기 때문에 확인을 먼저 해보자.
```
(conda) gimseonghyeon-ui-MacBook-Pro:project kimseonghyeon$ python manage.py --help
Type 'manage.py help <subcommand>' for help on a specific subcommand.

Available subcommands:

[django]
check
compilemessages
createcachetable
dbshell
diffsettings
...
```
django에서 기본으로 제공되는 command와 겹치지 않는 네임을 하나 생각하고..
```
your_app\
    __init__.py
    models.py
    views.py
    urls.py
    templates\
    management\
        commands\
            commandname.py
    tests.py
```
### custom command
앱폴더 안에 위 디렉토리 구조처럼 `management/commands/commandname.py`  만들어 준다. python 파일의 이름은 기본 command와 겹치지 않으면 되고 `management/commands/` 이 디렉토리의 이름은 변경하면 작동이 안된다.

```python
# commandname.py
from django.core.management.base import BaseCommand, CommandError

def action(*args, **kwargs):
    try:
        # 이부분에 원하는 액션을 주면됨
        ...
    except OSError:
        raise CommandError('Django reload shell error....')

class Command(BaseCommand):
    def handle(self, *args, **options):
        print('start commands')
        try:
            action(args=None, kwargs=None)
        except CommandError:
            self.stdout.write(self.style.ERROR('Django reload faild'))
        else:
            self.stdout.write(self.style.SUCCESS('Successfully Django reload '))
```
아래 부분의 Command class의 이름을 수정하면 사용할 수 없다.
def action 부분의 메소드만 수정하여 원하는 기능을 추가하여 주면 된다.
