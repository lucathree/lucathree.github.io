---
title: "Django REST API 환경 구축 (1) - 도커를 사용한 장고 기본 환경 세팅"
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

Django는 파이썬을 기반으로 하는 풀스택 웹프레임워크입니다. 웹사이트를 개발하기 위해 필요한 기능들을 제공해주고 정형화된 구조 아래에서 개발하도록 되어있기 때문에 맨바닥부터 웹 개발을 하기 위해 고민할 필요없이 신속하게 개발을 할 수 있도록 도와줍니다.
  
하지만 최근에는 프론트엔드 서버와 백엔드 서버가 분리되는 경우가 많고 다양한 형태의 클라이언트와 서버 사이에서 정보를 JSON 형태로 주고 받으면서 REST API의 형태로 웹 어플리케이션 서버를 구현해야 하는 일이 많아졌습니다. 현재 제가 속한 팀에서도 백엔드 서버를 REST API로 개발을 하고 있기 때문에 이번 프로젝트로는 Django REST Framework를 기반으로 개발을 하기 위한 기본적인 환경 구축을 진행했습니다.

<br/>

# Docker 컨테이너 생성
제가 찾아봤던 대부분의 장고 튜토리얼들은 파이썬 virtualenv 또는 venv를 이용해서 가상환경을 만드는 과정부터 시작했습니다.

로컬에 먼저 세팅을 하고 도커 컨테이너에는 세팅 된 파일들을 볼륨으로 설정해서 복사해주기만 하면 되는 편한 방법이긴 하지만, 그러면 나중에 배포를 하게 될 것을 가정해서 도커를 사용해서 어디에서든 바로 환경을 구축할 수 있도록 하려는 목적과는 조금 다른 것 같아 저는 처음부터 도커 명령어로만 모든 세팅이 이루어지도록 작업을 진행했습니다.

먼저 **requirements.txt** 파일을 만들어 기본적으로 사용할 Django와 Django REST Framework 패키지를 추가합니다.

```
Django
djangorestframework
```
  
다음, **dockerfile**을 만들고 아래와 같이 작성합니다.

```docker
FROM python:3.8.10

ENV PYTHONUNBUFFERED=1
WORKDIR /app

COPY . .

RUN pip install --upgrade pip
RUN pip install -r requirements.txt
```
- FROM : 파이썬은 제가 기존에 사용하던 익숙한 버전을 사용했습니다. 필요에따라 원하는 파이썬 이미지를 선택하면 됩니다.

- ENV PYTHONUNBUFFERED=1 환경변수를 설정해주면 도커 컨테이너 안에서 파이썬을 실행할 때 파이썬 출력이 버퍼링 없이 곧바로 현재 터미널에 출력되도록 해줍니다. 
이러면 장고 어플리케이션 로그를 실시간으로 확인할 수도 있고, 어플리케이션이 다운됐을 때 로그가 버퍼 안에서 소실되는 것을 막아줍니다.

- WORKDIR 는 컨테이너 내부에서 작업을 진행할 경로를 의미합니다.
- COPY . . 명령어로 현재 작업 중인 폴더의 파일을 컨테이너의 현재 작업 경로(/app)에 복사합니다.
- 마지막으로 pip를 최신 버전으로 업데이트 시킨 후 requirement.txt 안에 작성해놓은 패키지들을 설치하도록 합니다.

**docker-compose.yaml** 파일도 생성하여 아래와 같이 작성합니다. 

```yaml
version: "3"

services:
  web:
    build:
      context: .
      dockerfile: dockerfile
    volumes:
      - .:/app
    ports:
      - 8000:8000
    command: bash -c "
        django-admin startproject toyproject
        && python ./toyproject/manage.py runserver 0.0.0.0:8000
      "
```
- version : docker-compose 작성 방법의 기준이 되는 버전을 표기합니다.
- services : 하위에 만들고자 하는 컨테이너의 정보를 작성합니다.
- build : 컨테이너를 만드는데 사용할 dockerfile 의 위치와 파일명을 입력합니다.
- volumes : 프로젝트 소스를 로컬 환경과 컨테이너 내부에서 공유하며 작업을 할 수 있도록 'bind-mount' 형식으로 볼륨을 생성했습니다.
- ports : 컨테이너 내에서 실행중인 프로세스를 외부에서도 접근이 가능하도록 하기 위해 포트번호를 바인딩 해줍니다.
- command : 현재 컨테이너 내에 실행시킬 프로세스가 없는 상태입니다. 컨테이너 내에 실행 중인 프로세스가 없으면 컨테이너가 자동으로 종료되기 때문에 우선은 장고 프로젝트를 생성하고 runserver로 프로세스를 실행시키는 커맨드를 포함하여 작성했습니다.

**docker-compose.yaml** 파일이 있는 경로에서 터미널을 실행하고 다음 도커 명령어를 실행합니다.

```bash
docker-compose up --build
```

도커 이미지와 컨테이너가 정상적으로 생성되었을 경우 다음과 같은 프로젝트 디렉토리가 만들어집니다.

```
└── Works
	├── toyproject
	│   ├── toyproject
	│   │   ├── __init__.py
	│   │   ├── asgi.py
	│   │   ├── settings.py
	│   │   ├── urls.py
	│   │   └── wsgi.py
	│   ├── db.sqlite3
	│   └── manage.py
	├── requirements.txt
	├── docker-compose.yml
	└── dockerfile
```

서버 접속을 위해 **settings.py** 에서 ALLOWED_HOSTS 에 ‘localhost’, ‘127.0.0.1’ 혹은 원격서버에서 작업 중일 경우 해당 ip 주소를 넣어주고 나면 브라우저를 통해 컨테이너 및 장고 프로젝트가 생성되어 동작하는 것을 확인할 수 있게됩니다.

<p align="center" style="color:#808080"><img src="https://raw.githubusercontent.com/lucathree/lucathree.github.io/master/assets/images/2022/2022-04-13.png">

