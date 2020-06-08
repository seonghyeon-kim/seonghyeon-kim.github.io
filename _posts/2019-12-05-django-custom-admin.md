---
layout: post
title: django custom admin
tags:
- django
- django jet
---

### 개요
jet admin을 찾은 이유는 admin의 리스트와 admin action을 추가할 작업이 생겨, 작업하는 김에 template도 수정할 수 있다는 글을 보고 적용시켜 보았다.

---
django jet 설치
```console
(conda) gimseonghyeon-ui-MacBook-Pro:project kimseonghyeon$ pip install django-jet
```
설치 후, 프로젝트 폴더의 settings.py에 가서 installed app에 추가해준다. 순서는 맨마지막에 추가해주는 것이 좋다.
```python
INSTALLED_APPS = (
    ...
    'jet',
    # dashboard 기능을 사용하려면 다음을 추가
    'jet.dashboard',
    'django.contrib.admin',
)
```

이후 url에 아래처럼 추가해주면 사용이 가능하다.
```python
urlpatterns = patterns(
    '',
    url(r'^jet/', include('jet.urls', 'jet')),  # Django JET URLS
    url(r'^jet/dashboard/', include('jet.dashboard.urls', 'jet-dashboard')),  # Django JET dashboard URLS
    url(r'^admin/', include(admin.site.urls)),
    ...
)
```

적용 후 admin 에 접속해보면
![image](./images/customadmin/1.png)
처럼 바뀐것을 확인할 수 있다.

---

### admin list 수정하기
``` python
class TeacherAdmin(admin.ModelAdmin):
    list_display = (
        'userin',
        '',
    )

    search_fields=('',)

    inlines = [*Inline,''] # foreign 역방향 테이블 가져오기, Inline은 그대로 *있는 부분에 모델명

    actions = ["listPDF"] # custom한 액션
    
    # List method field
    def userin(self, obj):
        user = User.objects.get(pk=obj.user.pk)        
            return user.first_name
    userin.short_description = "이름"
    
    # custom action
    def listPDF(self, request, queryset):
        with tempfile.SpooledTemporaryFile() as tmp:
            with zipfile.ZipFile(tmp, 'w', zipfile.ZIP_DEFLATED) as archive:

                # pdfkit config
                os_env = platform.system()
                if os_env != 'Darwin':
                    config = pdfkit.configuration(wkhtmltopdf="/usr/bin/wkhtmltox/bin/wkhtmltopdf")
                ...
```
위처럼 `listPDF` 액션을 custom 할 수 있고,
list에 method를 추가해 변형하여 보여줄 수 있다.
참고: [django admin cookbook](https://books.agiliq.com/projects/django-admin-cookbook/en/latest/)
