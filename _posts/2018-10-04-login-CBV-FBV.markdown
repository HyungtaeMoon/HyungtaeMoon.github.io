---
layout: post
title:  "Django의 Login view 구현 방법"
date:   2018-10-04 04:42:19
author: Moon Hyungtae
categories: django
tags:	django
cover:  "/assets/instacode.png"
---

- 이 자료는 수업내용, AskDjango 와 장고문서를 참고로 작성하였습니다.

유저가 로그인을 하기 위해 머릿속에 그려야 할 것이 있다.

웹프레임워크 그러니까 장고에서 먼저 데이터를 보낼 일은 없다. 유저가 로그인을 하면 해당하는 서버(장고)에서 적절한 응답을 보내주기만 하면 된다.

정상적인 유저가 로그인을 할 경우,
- form 에 데이터가 담겨 오기 때문에 **POST** 방식
- 저장되어 있는 DB에 유저가 있으면 인증을 받아 유저에게 로그인을 시켜주고, 그렇지 않으면 로그인 페이지로 이동시켜준다.

**FBV(Function Based View) 방식의 view 작성**

```python
def login_view(request):
    if request.method == 'POST':
        username = request.POST['username']
        password = request.POST['password']
        print(request.user.is_authenticated)
        user = authenticate(request, username=username, password=password)

        if user is not None:
            login(request, user)
            return redirect('posts:post-list')

        else:
            return redirect('members:login')

    else:
        return render(request, 'members/login.html')

```

**CBV(Class Based View) 방식의 view 작성**

Django에 내장되어 있는 Class 기반의 뷰를 사용하면 추가로 템플릿만 작성하면 로그인 기능을 사용할 수 있다.
DJango 안에 해당 기능이 내장되어 있는데 **auth_views.LoginView** 에서 처리가 가능하다.

```python

urlpatterns = [
path('login/', auth_views.LoginView.as_view(template_name='members/login.html'),
      name='login'),
]
```
