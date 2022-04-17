---
title: "Django REST API 환경 구축 (3) - Nginx & Gunicorn 연동"
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

Django를 이용해서 개발을 진행중일 때는 Django 명령어 runserver를 통해서 로컬에서 서버를 실행시키고 디버깅을 할 수 있습니다. 하지만 runserver는 보안과 관련된 부분은 전혀 신경쓰지 않았기 때문에 실제 배포에 사용하기에는 부적절하므로 제대로 된 웹서버 및 wsgi 미들웨어가 필요합니다.

## 웹서버와 WSGI
### 웹서버
'웹서버'는 두가지 의미를 가지고 있습니다. 하나는 클라이언트로부터 http request를 받고 그에 해당하는 응답을 반환하는 소프트웨어를 의미하고 다른 하나는 위의 소프트웨어를 실행시키는 컴퓨터 하드웨어를 의미합니다.

여기서, Django를 이용해서 작성한 소스가 실행되는 환경을 웹서버라고 착각할 수 있지만 Django는 어디까지나 웹 프레임워크고, 소스에 따라 프로그램이 동작하는 환경은 웹 어플리케이션 서버(WAS)라고 부릅니다.

웹서버는 클라이언트가 요청을 보내면 WAS의 proxy('대리자')로서 정적 컨텐츠를 처리하고 동적 컨텐츠에 대해 WAS에게 요청을 전달하여 응답을 받아 다시 클라이언트에게 전달합니다. 해당 역할을 정확히는 reverse proxy라고 부르며 웹서버는 이 외에도 로드밸런싱이나 보안관련 기능들도 제공합니다.

주요 웹서버 소프트웨어로는 Nginx와 Apache가 많이 쓰입니다. Apache는 오랫동안 사용되어 안정성이나 기능 제공면에서 뛰어나며, Nginx는 상대적으로 자원 사용률이 낮고 처리 성능이 뛰어나다는 장점을 가지고 있습니다.

### WSGI
웹서버와 어플리케이션서버 사이에서 정보를 주고받으려면 서로 통일된 구조/환경이 필요합니다. WSGI(Web Server Gateway Interface)는 기존에 사용되던 CGI(Common Gateway Interface)의 문제점을 개선하여 웹서버와 파이썬으로 작성된 어플리케이션 사이에서 정보를 주고받기 위해 만들어졌으며, WSGI 방식에 따라 웹서버가 전달한 http request를 파이썬 객체로 바꾸고 다시 파이썬 객체를 http response로 바꾸는 WSGI 서버로 Gunicorn 또는 uWSGI 등이 사용됩니다.

*추후 공부사항: WSGI와 ASGI의 차이

<br/>

## Nginx와 Gunicorn 연동
### Gunicorn

앞서 구축된 Django 개발 환경에 gunicorn을 wsgi로 추가하려 합니다.   
먼저, gunicorn 패키지를 **requirements.txt** 에 추가해줍니다. 이후 빌드를 다시해도 되고, 아니면 컨테이너 내부의 bash에 접속해서 pip 명령어로 설치를 진행해도 됩니다. 

```
Django
djangorestframework
psycopg2
gunicorn
```

그리고 **docker-compose.yaml** 파일에서 web 서비스의 command 항목만 아래처럼 바꿔 웹어플리케이션 프로세스를 runserver 대신 gunicorn으로 실행시키도록 하면 됩니다.

```yaml
command: gunicorn toyproject.wsgi --bind 0.0.0.0:8000
```

### Static 파일 정리

웹서버의 기능 중 하나는 WAS를 거치지 않고도 정적 페이지를 제공할 수 있게하여 서버의 부담을 줄이고 클라이언트의 요청을 빠르게 처리하도록 하는 것입니다. 그러기 위해서는 정적 파일들을 한 곳에 모으고 경로를 설정해줘야 합니다.

먼저 **settings.py** 에서 STATIC_URL과 STATIC_ROOT를 설정해줍니다.

```python
STATIC_URL = 'static/'
STATIC_ROOT = BASE_DIR.joinpath('static')
```

그 다음 web 컨테이너에 접속하여 위에서 설정한 경로로 정적 파일들을 모아주도록 합니다.

```bash
root@{컨테이너ID}:/app# ./manage.py collectstatic
```

그러면 프로젝트 디렉토리 내에 /static 폴더가 생기고 그 안에 css, fonts, img, js 등 정적 파일들이 저장 된 것을 확인할 수 있습니다.

### Nginx

이제 nginx 컨테이너를 띄워서 연동하려고 합니다. 컨테이너 생성을 위해 프로젝트 루트 디렉토리 아래에 nginx 폴더를 만들었습니다.

그리고 해당 폴더 안에는 nginx 이미지 빌드를 위한 별도의 dockerfile과 nginx 설정 파일인 toyproject.conf 를 생성합니다.

```
└── works
	├── nginx
	│	  ├── dockerfile
	│		└── toyproject.conf
	├── toyproject
	│		├── static
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

**toyproject.conf**

```yaml
upstream django {   
    server web:8000;
}

server {
    listen 80;

    location / {
        proxy_pass http://django;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Host $host;
        proxy_redirect off;
    }

    location /static/ {
        alias /static/;
    }

}
```

- Nginx 가 reverse proxy 로서 request와 response를 클라이언트와 WAS 사이에서 중개하기 위한 연결 설정
- 정적 페이지 처리를 위한 static 파일 경로 지정

**dockerfile**

```docker
FROM nginx

RUN rm /etc/nginx/conf.d/default.conf
COPY ./nginx/toyproject.conf /etc/nginx/conf.d/toyproject.conf
```

- nginx 이미지 빌드시 기존의 default 설정 파일을 지우고 앞서 작성한 toyproject.conf 설정 파일을 conf.d 경로에 추가

마지막으로 **docker-compose.yaml** 파일에 nginx 설정을 추가하고 컨테이너를 빌드하여 실행합니다.

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
    command: gunicorn toyproject.wsgi --bind 0.0.0.0:8000
    depends_on:
      - db
      - nginx

  db:
    image: postgres
    volumes:
      - postgresDB:/var/lib/postgresql/data
    environment:
      - POSTGRES_PASSWORD=postgrespw

  nginx:
    build:
      context: .
      dockerfile: ./nginx/dockerfile
    volumes:
      - ./toyproject/static:/static
    ports:
      - 80:80

volumes:
  postgresDB:
```
컨테이너를 모두 실행시키고 nginx 서버와 gunicorn 프로세스가 정상 작동한다면 이제 포트번호 없이 url에 로컬 아이피주소를 입력하는 것만으로도 테스트 페이지가 출력되는 것을 확인할 수 있습니다.

또한 개발자 모드로 들어가서 서버를 확인하면 nginx가 표시되는 것도 볼 수 있습니다.

<p align="center" style="color:#808080"><img src="https://raw.githubusercontent.com/lucathree/lucathree.github.io/master/assets/images/2022/2022-04-17.png">