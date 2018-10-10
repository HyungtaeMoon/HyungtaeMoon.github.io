---
layout: post
title:  "AWS부터 Django까지"
date:   2018-09-26 12:43:59
author: Moon Hyungtae
categories: DJango
tags:	aws
cover:  "/assets/instacode.png"
---

- 모든 내용은 수업내용을 토대로 개인의 생각을 추가로 작성합니다.

### # 장고로 AWS 를 이용한 배포하기(feat: Docker)

--------------------

`홈페이지 소개`

**`Amazon Web services(AWS)는 안전한 클라우드 서비스 플랫폼으로서, 컴퓨팅 파워, 데이터베이스 스토리지, 콘텐츠 전송 및 기타 기능을 제공하여 기업이 확장하고 성장하도록 지원합니다...(이하생략)`**

_**MS 에서 나온 Azure 서비스**_ 도 이와 비슷한 클라우드 컴퓨팅 플랫폼 서비스이지만 수강을 하면서 AWS 를 이용했기 때문에 Azure는 나중에 기회가 되면 다루도록 하겠다.

이와같은 클라우드 플랫폼을 사용하는 이유는 중소 기업의 경우 서버의 모든 환경을 세팅하고 유지하기에는 시간/금전적 비용이 부담스럽다. Amazon 역시도 이 부분에 대한 니즈가 필요한 기업들이 많다고 판단하여 이 사업에 뛰어들었을 것이다.

즉, AWS 를 통한 서버 환경 제공 서비스를 구축하여 수익을 올리고 있다.

<br>
<br>

#### # **실제 작업 사이클**

---------------------

`AWS(EB, EC2, S3, RDS) // Docker // Django // PostgreSQL`

장고에서 작업을 했다고 장고에서 시작이라고 생각하면 헷갈린다. 브라우저가 request 를 보낸 것에 대한 response 를 장고에서 하는 것이라고 생각하자.

<br>
<br>

**브라우저의 요청부터 EC2접근**

------------------------

**#Browser가 요청** 하면 AWS 의 **#ElasticBeanstalk(EB)** 가 이 응답을 받고 서버(EC2)를 늘릴 것인지 유지할 것인지를 결정한다. 그리고 **#Load Balancer** 가 **#Auto Scaling** 그룹의 인스턴스로 들어오는 트래픽에 대한 판단을 하고 **#EC2** 의 수를 증감시킨다.

`번외설명` Load Balancer 는 scale-out 의 기법으로 처리가 되는데 하드웨어와 관련한 기법 2가지를 설명한다.

- scale-up: 하드웨어 스펙향상
- scale-out: 하드웨어 수량을 늘림

_하드웨어의 스펙향상보다는 수량을 늘리는 쪽이 비용적으로 절감이 된다고 하더라._


```
Broswer ---------> ElasticBeanstalk -----> EC2(*eb ssh로 접근 가능)
       Load Balancer
     (Auto Scaling이 판단)
```

<br>
<br>

**생성된 EC2의 역할과 Nginx에 접근**

--------------------

이렇게 생성된 EC2는 다음과 같은 2가지 경로(역할)를 가지게 된다.

- **Nginx**(EB 안의 Nginx, .ebextensions 포함)로 응답에 대한 처리를 한다. (.ebextensions 는 EB 를 같이 참조하는 역할을 해준다.)

- 이 EC2는 RDS, S3 에 접속하여 데이터를 조회할 수 있는 역할을 한다.

작업을 하면서 포트 번호를 장고 안에서 설정했는데, 이를 Nginx 가 몇번인지를 알지 못한다. 이러한 때 Reverse Proxy 를 이용하여 몇번 포트로 오던 Nginx 가 알아챌 수 있도록 해야 한다.

```
EC2 ----> Nginx(웹서버) ---->
        - EB안의 Nginx
        - (Reverse Proxy):80 --> 7999 (EXPOSE(Dockerfile))

```

`웹서버`

정해진 통신 규약에 의해 서버-클라이언트가 송수신할 수 있도록 해주는 역할을 함

- 전통강자인 Apache(많은기능/그만큼 무거움)
- 신흥강자인 Nginx(기능은 다소 부족/속도빠름)

<br>
<br>

**EC2부터 Docker Container 안의 사이클**

-----------------------------

이제부터는 Docker 안에서 이루어지는 작업들이다.

Docker 를 사용하는 이유는 효율적인 자원 활용을 하기 위함인데 Dockerfile 자체를 따로 구분하면 더욱 큰 시너지 효과를 기대할 수 있다.

1) **Dockerfile.base**

- python, 의존성 업그레이드, nginx, supervisor, requirements의 패키지들을 설치

2) **Dockerfile.local**

- DJANGO_SETTINGS_MODULE 을 local 로 변경해주고, 현재 작업한 내용들을 docker의 srv/project 에 복사

3) **Dockerfile.dev**

- DJANGO_SETTINGS_MODULE 을 dev 로 변경해주고 Nginx 설정파일들을 복사하는 등의 작업내용

4) **production**

- DJANGO_SETTINGS_MODULE 을 production 으로 변경해주고, Nginx 설정파일들을 복사하는 등의 작업내용(dev 모드와 유사)

(이와 같이 할 수 있는 이유는 settings 를 파이썬 패키지화 시켰기 때문에 가능하다.)

이렇게 설정된 Docker image 에 따라서 테스트/실제 서버 등의 분할 작업 처리가 가능해졌다.

```

>>>>>>>>>>> (Docker Container의 안에서 작업) <<<<<<<<<<<<<

-----> Docker(Container):7999 ----> Nginx(Docker Container)

--------> uWSGI -----> Django
(socket)
```

그리고 uWSGI 의 역할은 Nginx 가 파이썬 언어를 모르기 때문에 이를 변환시켜주는 역할을 하기 때문에 Django 에 보내기 위해서는 필수적으로 필요하다.
