---
layout: post
title:  "module is not callable 오류"
date:   2018-10-10 14:41:19
author: Moon Hyungtae
categories: django
tags:	django
cover:  "/assets/instacode.png"
---

- 개인의 공부를 위한 목적으로 포스팅을 정리 중입니다.
- 개인의 주관적인 생각으로 포스팅을 하기 때문에 추측성 글들도 담겨있습니다.

<br>
<br>

`Adding django to Pipfile's [packages]...
Pipfile.lock not found, creating...
Locking [dev-packages] dependencies...
Locking [packages] dependencies...
lib/python3.7/site-packages/pipenv/utils.py", line 402, in resolve_deps
    req_dir=req_dir
  File "/usr/local/Cellar/pipenv/2018.7.1/libexec/lib/python3.7/site-packages/pipenv/utils.py", line 250, in actually_resolve_deps
    req = Requirement.from_line(dep)
  File "/usr/local/Cellar/pipenv/2018.7.1/libexec/lib/python3.7/site-packages/pipenv/vendor/requirementslib/models/requirements.py", line 704, in from_line
    line, extras = _strip_extras(line)
TypeError: 'module' object is not callable`

**주목할 점**

```python
**TypeError: 'module' object is not callable**
```

<br>
<br>

**발생원인**

-------------------

`brew upgrade pyenv` 로 업데이트 진행

```python
1.2.5 --> 1.2.7 # pyenv 버전 업데이트
```

<br>
<br>

**발생하는 문제**

--------------------

가상환경에서 django, awsebcli 를 install 할 때마다 module is not callable 오류가 발생

**pyenv 만 업그레이드 했기 때문에 이와 관련한 패키지 파일들이 서로 호환이 되지 않는 상황이 발생**

<br>
<br>

**해결방법**

-----------------------

해결방법은 brew upgrade 를 통해서 모든 파일들을 업그레이드

<br>
<br>

**추가내용**

------------------------

Pipfile은 가상환경을 생성함과 동시에 만들어지고, 가상환경에 패키지를 설치하는 순간부터 Pipfile.lock 이 생성된다.

예) pipenv install django 등의 패키지 설치
