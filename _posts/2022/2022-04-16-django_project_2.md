---
title: "Django REST API 환경 구축 (2) - PostgreSQL 컨테이너 연결"
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

Django 프로젝트를 생성하면 기본 DB로 SQLite가 설정되어 있습니다. SQLite는 임베디드 DBMS로 별도 설치없이 사용이 가능하고 파일 하나에 데이터베이스 전체가 저장되기 때문에 가볍다는 장점이 있습니다.

하지만 경량화 되어있기 때문에 일반적인 SQL로 사용할 수 있는 기능이 제한적이라는 단점도 존재합니다. 그래서 저는 SQLite 대신 장고에서 사용했을 때 지원되는 기능이 다른 RDBMS 보다 많은 PostgreSQL을 사용했습니다. (참고:[https://docs.djangoproject.com/en/4.0/ref/contrib/postgres/](https://docs.djangoproject.com/en/4.0/ref/contrib/postgres/))

장고 어플리케이션에 PostgreSQL을 연결시키기 위해 **requirements.txt** 에 PostgreSQL DB 어댑터인 psycopg2 패키지를 추가해줍니다.

```
Django
djangorestframework
psycopg2
```

‘web’ 컨테이너의 볼륨으로 toyproject 폴더를 사용하기 위해 디렉토리 구조 및 dockerfile을 수정해주었습니다. 또한 PostgreSQL을 사용할 것이기 때문에 db.sqlite3 파일은 삭제했습니다.

```
└── works
	├── toyproject
	│		├── toyproject
	│	  │   ├── __init__.py
	│		│   ├── asgi.py
	│	  │   ├── settings.py
	│	  │   ├── urls.py
	│	  │   └── wsgi.py
	│		├── requirements.txt
	│	  └── manage.py
	├── docker-compose.yml
	└── dockerfile
```

```docker
FROM python:3.8.10

ENV PYTHONUNBUFFERED=1
WORKDIR /app

COPY /toyproject .

RUN pip install --upgrade pip
RUN pip install -r requirements.txt
```

그 다음 PostgreSQL 컨테이너를 더하기 위해 docker-compose.yaml 파일을 수정했습니다.

```yaml
version: "3"

services:
  web:
    build:
      context: .
      dockerfile: dockerfile
    volumes:
      - ./toyproject:/app
    ports:
      - 8000:8000
    command: python ./manage.py runserver 0.0.0.0:8000
		depends_on:
			- db

  db:
    image: postgres
    volumes:
      - postgresDB:/var/lib/postgresql/data
    environment:
      - POSTGRES_PASSWORD=postgrespw

volumes:
  postgresDB:
```

- 앞서 ‘web’ 컨테이너를 띄우면서 장고 프로젝트 폴더 및 파일들이 이미 만들어졌기 때문에 이제부터는 해당 파일들을 다시 생성할 필요없이 볼륨으로 참조하기만 하면 됩니다. ‘volumes’와 ‘command’ 항목을 적절하게 수정해줍니다.
- image : db 컨테이너에는 별도 패키지가 필요하지 않으므로 도커 허브에서 ‘postgres’ 공식 이미지를 가져다가 사용합니다.
- environment : [https://hub.docker.com/_/postgres](https://hub.docker.com/_/postgres) 도커 허브 공식 문서에 따라 web 컨테이너에서 db 연결이 가능하도록 환경변수를 설정했습니다. 설정 가능한 항목이 다양하지만 기본적으로는 비밀번호만 설정하면 postgresql db 연결이 가능합니다. POSTGRES_USER 와 POSTGRES_DB는 별도 설정하지 않을 경우 default 값으로 ‘postgres’를 사용합니다.
- 컨테이너를 지워도 db에 저장된 정보는 사라지지 않도록 postgresDB 볼륨을 설정해줬습니다. bind-mount 형태와는 다르기 때문에 해당 볼륨은 도커에 의해 관리되며 프로젝트 디렉토리에는 폴더 및 파일이 생성되지 않습니다.

**settings.py** 의 DATABASES 항목은 아래와 같이 수정해줍니다.

```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': 'postgres',
        'USER': 'postgres',
        'PASSWORD': 'postgrespw',
        'HOST': 'db',
        'PORT': 5432,
    }
}
```

도커 이미지와 컨테이너를 다시 생성한 뒤, 아래 명령어로 실행중인 web 컨테이너에 접속해서 migration을 진행해줍니다. 장고는 ORM을 지원하기 때문에 db 연결 후 migration을 해주지 않으면 서버 실행 시 오류가 발생할 수 있습니다.

migrate를 해주고 나면 db에 기본 장고 사용자 테이블이 만들어졌을 것이기 때문에 createsuperuser 명령어를 사용해서 관리자 계정도 추가할 수 있습니다.

```bash
docker exec -it works_web_1
```

```bash
root@{컨테이너ID}:/app# ./manage.py migrate
root@{컨테이너ID}:/app# ./manage.py createsuperuser
```

‘http://127.0.0.1:8000/admin’ 으로 접속하면 이제 로그인 화면이 표시되고, 이제 postgres db에 저장되어있을 관리자 계정을 통해 대시보드 로그인도 가능합니다.

<p align="center" style="color:#808080"><img src="https://raw.githubusercontent.com/lucathree/lucathree.github.io/master/assets/images/2022/2022-04-16/1.png">