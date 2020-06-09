---
layout: post
title: Django google mail 보내기
tags:
- django
- smtp
- google
---
### 개요
프로젝트 진행 중 mail을 이용할 일이 생겨, google의 smtp를 사용하여 mail을 전송해보려 한다.

---
### GMail SMTP 서버
GMail의 SMTP 서버를 이용하기 위해서는 먼저 두 가지 설정을 해줘야한다.

-   [다른 이메일 클라언트에서 GMail을 확인할 수 있도록 IMAP 사용 설정](https://support.google.com/mail/answer/7126229?hl=ko&rd=3&visit_id=1-636281811566888160-3239280507#ts=1665018)
-   [보안 수준이 낮은 앱 및 Google 계정 허용](https://myaccount.google.com/lesssecureapps)
---
이제 프로젝트로 돌아와서
`settings.py` 를 열고 아래 내용을 추가해준다.
```python
# settings.py

EMAIL_BACKEND = 'django.core.mail.backends.smtp.EmailBackend'
EMAIL_HOST = "smtp.gmail.com"
EMAIL_HOST_USER = 'your-gmail@gmail.com'
EMAIL_HOST_PASSWORD = 'your-gmail-password'
EMAIL_PORT = 587
EMAIL_USE_TLS = True
DEFAULT_FROM_EMAIL = EMAIL_HOST_USER
```
EMAIL_HOST_USER에는 자신의 이메일을 적어주고, EMAIL_HOST_PASSWORD에는 비밀번호를 적어주면 된다.
이제 사용을 위한 준비는 마쳤다.
실제로 한번 보내보자. 
```python
from django.core.mail import EmailMessage

email = EmailMessage(
    '안녕하세요',                # 제목
    '김성현입니다.',       # 내용
    'ghgjgj123419@gmail.com',
    to=['ghgjgj123@naver.com',]  # 받는 이메일 리스트
)
email.send()
```
![image](/images/mail/1.png)
이렇게 하여 메일을 보내는 방법을 알아보았다.
프로젝트 내에 메일을 보내는 일이 잦다면 아예 function이나 file로 빼두는 것이 좋을 듯 하다.
