---
layout: post
title:  "Fork 한 git 은 잔디가 심어지지 않는다"
date:   2018-10-03 08:42:31
author: Moon Hyungtae
categories: git
tags:	git
cover:  "/assets/instacode.png"
---

Fork 한 저장소는 내 저장소에서 `Commit` 에는 반영되지만 잔디(contributions)에는 반영되지 않습니다.

이를 위해 여러 삽질(?)을 했지만 결국 답은 가장 가까이 있었다는 것.

(물론 이 방법이 정확한 정답이 아닐 수는 있지만 해결이 되었으니 참고)


**첫번째 확인**
```
# .git/config 에 내 계정이 등록되어있는지 확인

[remote "origin"]
        url = git@내 저장소명
        fetch = +refs/heads/*:refs/remotes/origin/*
```

Fork 해 온 다른 사람의 저장소가 아니라 내 저장소로 확인이 되었다.

**두번째 확인**

- 우선 Fork 한 저장소를 내 잔디에 반영되는 순간은 상대방이 Pull Request 를 해주었을 때 반영이 될 것으로 추측된다.(이는 아직 실험해보지 않았다.)
- 1) 나의 저장소에 2) 내가 쓴 글이 반영될 수 있도록 3) 새로운 저장소를 만들어 4) git clone 을 해준다.

```python
# 문제의 Fork해 온 저장소를 clone 해준다
git clone git@repository 주소

# origin 의 저장소가 내 저장소인지 확인
git remote -v

# 맞으면 push 를 해준다
git push -u origin master
```

**문제 발생**

push 를 하고 내 저장소를 보았더니 "귀하의 의존성에 잠재적인 보안 취약점이 있음을 발견했습니다" 라는 문구와 함께 친절하게 Gemfile.lock에 정의 된 종속성 중 일부는 알려진 보안 취약성이 있으므로 업데이트해야 합니다. 라고 알려준다.

**문제 해결**
Gemfile.lock은 bundle 을 통해 관리할 수 있으므로 `bundle update`를 해주어 최신판으로 업데이트를 시켜준다.
