---
layout: post
title:  "커스텀 유저 모델 정의"
date:   2018-10-09 14:41:19
author: Moon Hyungtae
categories: django
tags:	django
cover:  "/assets/instacode.png"
---

- 개인의 공부를 위한 목적으로 포스팅을 정리 중입니다.
- 개인의 주관적인 생각으로 포스팅을 하기 때문에 추측성 글들도 담겨있습니다.
<br>
<br>

### **커스텀 유저 모델을 사용하여 재정의하는 이유**

----------------------

장고는 기본적으로 유저 모델을 정의하는 애플리케이션 `django.contrib.auth` 에 정의되어 있다.

그러나 장고에서 제공하는 기본적인 필드만으로는 실제 서비스를 운영하기에는 구성 자체가 빈약하다.

- username, password 만 필수 입력값으로 지정되고 나머지는 선택사항이기 때문에 만일 실제 서비스에서 유저가 자신의 비밀번호를 잊었을 경우에 어떻게 증명할 것인가?

물론 원한다면 실제 서비스에서 기본 유저 모델만을 이용하여 운영이 가능하지만 서비스를 운영하는 중에 유저 모델을 재정의한다면 아래와 같은 오류가 발생할 것이다.

`
django.db.migrations.exceptions.InconsistentMigrationHistory: Migration admin.0001_initial is applied before its dependency members.0001_initial on database 'default'.`

이와 같은 오류의 원인은 **migrate** 를 하면 **make migrations** 를 통해 만들어진 테이블이 나의 애플리케이션에 적용되는데,

`required`한 필요한 데이터를 기존의 데이터들은 가지고 있지 않을 것이다. 그렇게 되면 기존 데이터는 필드를 채워넣을 수 없기 때문에 오류가 발생하게 되는 것이다.

<br>
<br>

### **마이그레이션 파일 삭제하는 방법**

-------------------------------

해결방법은 migrations 디렉토리 내에 `__init.py__` 를 남기고 모든 파일을 지워주면 된다.

- `./manage.py showmigrations` 로 지워야 할 마이그레이션 앱을 확인

- `./manage.py migrate zero <app이름>`

나의 경우에는 ./manage.py migrate zero members

- `./manage.py makemigrations`

- `./manage.py migrate`

이와 같이 해주면 기존의 DB 데이터는 모두 삭제되고 테이블을 새로 생성하게 된다.


#### **오류가 여전히 해결되지 않았다면**

(최종 방법이기도 하고 실제로 많이 써먹기도 한 방법인데) 만약 위의 방법으로도 오류가 계속 발생한다면 `db.sqlite3 파일을 삭제`하고 새로 마이그레이션을 하면 된다.

<br>
<br>

### **커스텀 유저 모델 만들기**

--------------------------------

`app/config/settings.py`
```python
AUTH_USER_MODEL = 'members.User' #<app name>.User

```

`app/members/admin.py`
```python
from django.contrib import admin
from django.contrib.auth.admin import UserAdmin as BaseUserAdmin

from .models import User

class UserAdmin(BaseUserAdmin):
  pass


admin.site.register(User, UserAdmin)
```

`app/members/models.py`
```python
from django.contrib.auth.models import AbstractUser
from django.db import models

class User(AbstractUser):
  pass
```

<br>
<br>

### **결과**

-----------------------

loop back 주소인 `127.0.0.1:8000/admin` 페이지로 createsuperuser 로 생성한 계정을 통하여 접속하면 MEMBERS(#app이름) 에 사용자(들) 이라는 테이블이 생성됨을 확인할 수 있었다.
