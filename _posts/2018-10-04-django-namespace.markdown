---
layout: post
title:  "Django 네임스페이스"
date:   2018-10-04 03:43:19
author: Moon Hyungtae
categories: django
tags:	django
cover:  "/assets/instacode.png"
---

- 이 자료는 AskDjango 와 장고문서를 참고로 작성하였습니다.

**Django 의 Namespaces 적용**

적용해야 하는 이유?

`Django documents`

URL 네임 스페이스를 사용하면 다른 응용 프로그램이 동일한 URL 이름을 사용하는 경우에도 명명 된 URL 패턴을 고유하게 역순으로 지정할 수 있습니다. 타사 앱이 항상 네임 스페이스 URL을 사용하는 것이 좋습니다 (자습서 에서처럼). 마찬가지로 응용 프로그램의 여러 인스턴스가 배포되는 경우 URL을 역순으로 지정할 수도 있습니다. 즉, 단일 응용 프로그램의 여러 인스턴스가 명명 된 URL을 공유하므로 네임 스페이스는 이러한 명명 된 URL을 구분할 수있는 방법을 제공합니다.

...(중략)

네임 스페이스는 중첩 될 수도 있습니다. 명명 된 URL 'sports:polls:index'는 최상위 네임 스페이스 'sports' 내에 정의 된 'polls' 네임 스페이스에서 'index' 라는 패턴을 찾습니다.


**config/urls.py**
```python
from django.contrib import admin
from django.shortcuts import redirect
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('blog/', include('blog.urls')),
    path('', lambda req: redirect('blog:post_list')), # URL Reverse
]
```

**blog/urls.py**
```python
from django.urls import path
from . import views

app_name = 'blog'

urlpatterns = [
    path('', views.post_list, name='post_list'),
    path('<int:pk>/', views.post_detail, name='post_detail'),
]
```
