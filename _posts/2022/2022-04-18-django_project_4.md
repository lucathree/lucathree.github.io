---
title: "Django REST API 환경 구축 (4) - Supervisor를 이용한 프로세스 관리"
excerpt: "Django와 DRF를 이용해서 REST API를 만들기 위한 환경 구축을 진행해봅니다."
layout: single
author_profile: true
read_time: true
related: true
categories:
- Django
tags:
- Python
- Study Note
- Django
- Web
- Backend
toc: true
toc_sticky: true
---

앞의 과정까지는 하나의 도커 컨테이너 당 하나의 프로세스만 실행되도록 구성이 되었습니다. 도커 컨테이너의 경우 Foreground로 실행 중인 프로세스가 반드시 있어야만 컨테이너가 실행 상태를 유지하고 만약 해당 프로세스가 종료될 경우 컨테이너도 같이 중단됩니다. 만약 서비스 운영하는 상황이라면 서버 프로세스가 갑작스럽게 중단되는 경우를 미연에 방지하고, 만약 프로세스가 다운되더라도 자동으로 다시 실행될 수 있도록 할 필요가 있습니다.

컨테이너 당 프로세스가 하나씩만 있을 경우에는 docker-compose 파일에서 컨테이너들에 'restart: always' 설정을 추가해주면 프로세스가 갑작스럽게 중단되어도 컨테이너가 다시 실행됩니다. 하지만 만약 컨테이너 안에서 여러 개의 프로세스를 실행시켜야 하는 경우라면 어떻게 해야할까요? 이런 경우에는 여러 개의 프로세스 중 하나가 죽더라도 별도 프로세스 관리를 하기 어렵기 때문에 프로세스 관리 툴인 **Supervisor**를 사용합니다. Supervisor를 사용하면 **Foreground 프로세스**를 **daemon**화 하여 **Background 프로세스**로 만들어주고 프로세스가 갑작스럽게 종료되어도 자동으로 다시 실행될 수 있도록 관리가 가능합니다.

----
<br/>

먼저 이미지 빌드 시 supervisor 가 설치되도록 **requirements.txt** 에 패키지를 추가하고,

```yaml
Django
djangorestframework
psycopg2
gunicorn
supervisor
```

**supervisord.conf** 파일을 만들어서 web 컨테이너의 볼륨으로 사용 중인 toyproject 폴더 안에 저장합니다.

파일 안에는 백그라운드로 실행시킬 프로세스를 아래와 같이 지정해줍니다. 테스트용으로 gunicorn과 함께 기존의 runserver 프로세스를 추가했습니다. 이렇게 하면 어플리케이션 서버가 동작하는 것을 두 가지 방법으로 비교하며 작업할 수 있습니다.

```yaml
[program:django]
command=/app/manage.py runserver 0.0.0.0:8001
stderr_logfile=/app/logs/django_runserver.log
autostart=yes
autorestart=yes

[program:gunicorn]
command=gunicorn toyproject.wsgi --bind 0.0.0.0:8000
stderr_logfile=/app/logs/gunicorn.err.log
autostart=yes
autorestart=yes
```

- 백그라운드로 실행되기 때문에 터미널로 확인할 수 없는 프로세스 로그는 별도의 디렉토리를 지정해서 저장시킬 수 있습니다.
- autostart, autorestart 옵션을 통해 supervisor 최초 실행 시, 그리고 프로세스가 예상치 못하게 종료됐을 때 자동으로 프로세스 실행을 시도하도록 지정해줍니다.

도커 컨테이너가 종료되지 않고 실행상태를 유지할 수 있도록 nodaemon=true 설정을 추가하여 supervisor를 Foreground 프로세스로 만들어줍니다. 그리고 inet_http_server 주소 및 계정을 설정해주면 웹으로도 프로세스 관리를 할 수 있습니다.

```yaml
[supervisord]
nodaemon=true

[inet_http_server]  
port=0.0.0.0:9001   
username=user   
password=123
```

**docker-compose.yaml** 의 web 항목 중 ports와 command를 수정해서 컨테이너 실행 시 supervisor가 실행되도록 합니다.

```yaml
		ports:
      - 8000:8000
      - 8001:8001
      - 9001:9001
    command: supervisord -c supervisord.conf
```

위에서 supervisor 관리용 웹페이지를 설정해줬기 때문에 http://로컬아이피:9001 로 접속하면 두 프로세스가 supervisor에 의해 관리되고 있는 것을 확인할 수 있습니다.

<p align="center" style="color:#808080"><img src="https://raw.githubusercontent.com/lucathree/lucathree.github.io/master/assets/images/2022/2022-04-18.png">
