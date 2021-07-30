---
title: "팀 프로젝트2: 플라스크를 활용한 공공데이터 기반 웹서비스"
excerpt: "플레이데이터 인공지능SW개발자 과정에서 진행한 두번째 프로젝트의 진행기록"
layout: single
author_profile: true
read_time: true
related: true
categories:
- 개발일기
tags:
- Diary
- Python
- MySQL
- Flask
- Web
- 미니프로젝트
- 플레이데이터
- 인공지능SW개발자과정
toc: true
toc_sticky: true
---

# 프로젝트 설명

## 개요

- 프로젝트 주제: 따릉이 이용기록 서비스
- 참여인원: 3명 (5명)
- 소요기간: 5일
- 사용언어 및 환경
  - 언어: Python, MySQL, HTML, CSS, JavaScript
  - 프레임워크: Flask, Bootstrap
- 프로젝트 목표: 공공데이터 및 파이썬 라이브러리(numpy, pandas, matplotlib 등)를 사용하여 데이터 분석을 진행하고, 해당 내용을 플라스크를 사용하여 웹서비스로 구현

## 기획

### 주제 선정

현재까지 진행했던 프로젝트들에서는 데이터의 사이즈가 비교적 작은 편이었고(대략 5000건 미만) 데이터를 활용은 해봤지만 분석을 한 경우들이 적었기 때문에 큰 데이터를 가지고 비교분석을 해보는 것을 목표로 했다.

이 과정에서 서울시 공공자전거 "따릉이" 이용정보 데이터를 찾았고, 사용가능한 데이터가 70만건 이상에 항목도 다양했기 때문에 이를 활용한 서비스를 구축해보기로 했다.

### 사용 데이터

**서울특별시 공공자전거 이용정보 (2021년 1월 기준)**

- http://data.seoul.go.kr/dataList/OA-15245/F/1/datasetView.do
- 파일형태: CSV
- 데이터 종류: 대여일자, 대여시간, 대여소번호, 대여소명, 대여구분코드(정기권 사용여부), 성별, 연령대코드, 이용건수, 운동량, 탄소량, 이동거리, 사용시간
- 데이터 수: 총 718,857 건

**서울특별시 공공자전거 대여소 정보 (2021년 5월 기준)**

- https://www.data.go.kr/data/15051893/fileData.do
- 파일형태: CSV
- 데이터 종류: 대여소번호, 대여소 명, 소재지(자치구, 상세주소, 위도, 경도), 설치시기, 거치대수(LCD, QR), 운영방식
- 데이터 수: 총 2,154건

### 구현 기능

핵심 기능은 **1. 따릉이 전체 이용정보를 활용한 통계 그래프 표시** 와 **2. 사용자 이용내역을 입력받아 평균을 계산하고 전체 이용자 통계와의 비교** 였다.

여기에 웹서비스로 구현하기 위한 기본적인 회원가입 및 관리 기능과 게시판 기능을 추가하였는데, 수업을 통해 배운 내용 외 추가 연습을 해보기 위해 댓글기능을 추가했다. ****

- 회원관리: 가입, 로그인/로그아웃, 내 정보 확인/수정/탈퇴
- 사용자 게시판: 게시글 작성, 게시글 확인/수정/삭제, 댓글 작성/삭제
- 사용통계 조회: 연령별, 요일별, 시간대별, 지역별
- 이용현황 기록/조회: 이용내역 입력, 이용내역 확인 (전체 이용기록 & 이용통계), 사용통계 비교

**+**여기에 추가로 부트스트랩을 사용한 UI 구현 및 카카오맵 API 활용까지 연습해보기로 했다.

# 프로그램 구성

## DB 구조

<center><img src="https://raw.githubusercontent.com/lucathree/lucathree.github.io/master/assets/images/2021/2021-07-28/DB.PNG"></center>

- 테이블은 총 4개로 회원관련 정보를 저장하기 위한 member, 사용자 이용기록을 저장하기 위한 history, 게시글을 저장하기 위한 board와 댓글을 저장하기 위한 reply로 구성했다.
- 게시글 마다 댓글이 여러건이 달릴 수 있기 때문에 테이블을 2개로 나누고, reply 테이블에서는 board 테이블의 primary key인 'num'을 foreign key로 참조하도록 했다. 그 외 게시글, 댓글 및 이용기록의 작성자 확인을 위해 각 테이블에서 memeber의 id를 참조하고 있다.

## 파일 및 클래스 구조

이번 프로젝트에서는 웹 서비스를 구현하기 위해 **MVC 패턴**을 기반으로 작업을 진행했다.

서비스의 기능별로 회원정보 관련해서는 member, 게시판 관련은 board,  이용통계 관련은 bikedata, 그리고 이용기록 관련은 history라는 클래스로 구분하여 작업을 진행했고, MVC 패턴에 기반하여 다음과 같이 파일 구조를 나누었다.

### **Models**

기능 구현에 필요한 데이터를 불러오고 처리. 각 파일마다 VO, DAO, Service 클래스로 구성되어 있다.

- **member.py**

  회원정보를 DB member 테이블에서 호출하고 회원정보 추가, 수정, 삭제 등의 작업처리

- **board.py**

  게시판 및 댓글정보를 DB board와 reply 테이블에서 호출하고 게시글&댓글 추가, 수정, 삭제 등의 기능 담당

- **bikedata.py**

  따릉이 데이터 파일(csv)을 불러와 데이터 프레임을 만들고 전체 이용자 통계를 내어 그래프 생성

- **history.py**

  개인 이용자의 따릉이 이용내역을 DB의 history 테이블에 기록하고 다시 해당 데이터를 사용해서 따릉이 전체 사용량, 사용시간, 이동거리, 탄소배출 감축량 등을 계산하도록 함.

### **Routes**

MVC 패턴의 Controller에 해당하는 부분으로, Model과 View를 연결하고 제어하여 페이지별로 주소를 부여하고 필요한 기능들이 실행되도록 함.

- member_route
- board_route
- bikedata_route
- history_route

### **Templates**

MVC에서 View에 해당하는 부분으로 실제로 보일 페이지에 대한 html 파일들을 저장.

- member
  - 회원가입 페이지 (join.html)
  - 회원정보 확인 페이지 (detail.html)
  - 로그인 페이지 (login.html)
- board
  - 게시글 리스트 페이지 (list.html)
  - 게시글 본문 내용 페이지 (content.html)
  - 게시글 수정 페이지 (edit.html)
  - 게시글 작성 페이지 (form.html)
- bikedata
  - 이용통계 종류 표시 페이지 (button.html)
  - 각 통계 종류별 표시 페이지 (meanBy____.html)
- history
  - 이용내역 입력 페이지 (add.html)
  - 이용내역 및 사용자 통계 표시 페이지 (list.html)
- 그 외
  - index.html : 웹서비스의 메인 페이지
  - base.html: 웹서비스의 모든 페이지에 공통적으로 들어가는 부분 (예, 네비게이션 바). 플라스크를 통해 템플릿 상속을 적용하여 매번 같은 내용을 반복해서 입력하지 않도록 만들었다.

### Static

웹서비스의 구현에 필요한 그래프, 이미지, 통계데이터 등 변하지 않는 정적 파일들을 저장.

- data - 이용통계를 내기위한 공공데이터 파일들을 저장
- img - 웹페이지를 꾸미는데에 사용되는 배경 및 아이콘 등의 이미지 파일들을 저장
- graph - 데이터 분석을 통해 작성한 결과물인 그래프 이미지를 저장

# 개발과정

## 역할 분담

이번 프로젝트에서는 내가 조장을 맡아서 진행하게 되었다.

다같이 구현하고 싶은 서비스 주제를 정하고, 기능들을 정리한 뒤 팀원들이 각자 맡고 싶은 파트를 담당해서 개발을 진행하도록 했다. 우선 나는 각 팀원들이 모델 및 뷰 파일을 작성하면 그걸 한데 모아서 하나의 웹서비스로 묶고, 부트스트랩을 사용해서 UI 구현을 하기로 했다.

팀원들은 한 분은 데이터 처리 경험을 해보고 싶다고 하셔서 분석 및 그래프 작성까지 담당하도록 하고, 다른 한 분은 조금 어려워보이는 기능 구현에 도전해보고 싶다고 하여 입력된 이용자 정보를 바탕으로 사용시간, 이동거리, 탄소감축량 등을 계산하고 전체 사용자 통계와 비교하는 기능을 맡을 수 있도록 해드렸다.

그리고 초반 참여가 조금 저조했던 팀원 두 명이 있었는데, 그 분들께는 수업 중에도 진행해봤기 때문에 비교적 난이도가 낮은 회원가입 및 게시판 기능을 만들도록 했다. 그러나... 한 명은 교육과정 중도포기를 하고 나갔고, 다른 한 명은 말도 없이 잠수를 타버리는 바람에 상대적으로 내가 맡은 분량이 꽤 많은 프로젝트가 되어버렸다...

## 작업 순서

작업 순서는 각자 맡은 파트의 Model 파일과 기초적인 템플릿 구현을 먼저 진행하고, 결과물들을 모아서 라우트 설정을 한 뒤 부트스트랩으로 UI 구현 마무리를 짓는 방식으로 진행했다.

특이 사항은, 원본 데이터가 70만건이 넘다보니 서버에서 실시간으로 데이터를 처리하여 결과물을 만들 경우 스레딩 관련 프로그램 에러가 발생하였다.

위 문제를 해결하기 위해 중간에 다른 방식으로 작업을 진행하게 되었는데, 이번 프로젝트에서 사용한 데이터는 서비스 이용자로부터 입력받는 따릉이 사용기록 외에는 어차피 실시간으로 바뀌지 않는 고정적인 데이터였다. 그래서 데이터 분석 및 출력은 굳이 서버에서 실시간으로 처리하지 않아도 되겠다 싶어 분석은 주피터 노트북을 통해 처리하고, 웹에서는 그래프의 이미지만 출력하여 보여주는 방식으로 바꾸게 되었다.

# 개발결과

이번 프로젝트에서 작성한 모든 코드를 설명하기엔 양이 조금 많지만, 이번 프로젝트를 통해 처음 구현해본 주요 기능들을 짚어보자면...

### 부트스트랩

부트스트랩은 프런트엔드 디자인을 위한 프레임워크로 별도 디자인을 하지 않아도 전반적으로 깔끔한 UI를 구성할 수 있도록 도와준다.

css, js 파일 등을 직접 다운받아서 프로젝트 폴더 안에 넣고 사용해도 되지만, html 템플릿의 head와 body 안에 다음 코드만 붙여 넣으면 사용이 가능하다.

```html
<!-- head에 삽입하는 코드 -->
<link href="<https://cdn.jsdelivr.net/npm/bootstrap@5.0.2/dist/css/bootstrap.min.css>" rel="stylesheet" integrity="sha384-EVSTQN3/azprG1Anm3QDgpJLIm9Nao0Yz1ztcQTwFspd3yD65VohhpuuCOmLASjC" crossorigin="anonymous">

<!-- body에 삽입하는 코드 -->
<script src="<https://cdn.jsdelivr.net/npm/bootstrap@5.0.2/dist/js/bootstrap.bundle.min.js>" integrity="sha384-MrcW6ZMFYlzcLA8Nl+NtUVF0sA7MsXsP1UyJoMp4YLEuNSfAP+JcXn/tWtIaxVXM" crossorigin="anonymous"></script>
```

각 태그에 정해진 형식의 클래스 설정을 해주기만 하면 부트스트랩 디자인 적용이 가능한 방식인데 사용방법이 그리 어렵지 않아 [부트스트랩 공식 다큐먼트](https://getbootstrap.kr/docs/5.0/getting-started/introduction/)를 참고하여 작업을 진행했다.

### 네비게이션바 공통적용

위에서 조금 설명하긴 했지만, 웹사이트 전체에 공통으로 네비게이션 바를 적용하기 위해 플라스크의 템플릿 상속 기능을 사용했다.

먼저 공통적으로 적용시킬 템플릿을 만든 뒤 다음과 같이 중간에 block content, endblock 을 넣어주면 된다.

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>2조 - Project 따릉이</title>
    <link href="<https://cdn.jsdelivr.net/npm/bootstrap@5.0.2/dist/css/bootstrap.min.css>" rel="stylesheet" integrity="sha384-EVSTQN3/azprG1Anm3QDgpJLIm9Nao0Yz1ztcQTwFspd3yD65VohhpuuCOmLASjC" crossorigin="anonymous">
    <script src="<http://code.jquery.com/jquery.min.js>"></script>
</head>
<body>
		<!-- 아래부터 네비게이션바 코드 -->
    <nav class="navbar navbar-expand-lg navbar-dark bg-dark">
      <div class="container-fluid">
        <a class="navbar-brand" href="/">
            <img src="/static/img/project_logo.png" alt="" height="30" class="d-inline-block align-text-top">
            <b>Project따릉이</b>
        </a>
        <button class="navbar-toggler" type="button" data-bs-toggle="collapse" data-bs-target="#navbarNav" aria-controls="navbarNav" aria-expanded="false" aria-label="Toggle navigation">
          <span class="navbar-toggler-icon"></span>
        </button>
        <div class="collapse navbar-collapse" id="navbarNav">
          <ul class="navbar-nav me-auto">
            <li class="nav-item">
              <a class="nav-link" href="/history">My따릉이</a>
            </li>
            <li class="nav-item">
              <a class="nav-link" href="/bikedata">이용통계</a>
            </li>
            <li class="nav-item">
              <a class="nav-link" href="/board">커뮤니티</a>
            </li>
          </ul>
          <ul class="navbar-nav d-flex">
            {% raw %}{% if 'id' in session %}{% endraw %}
            <li class="nav-item">
              <a class="nav-link" href="/member/logout">로그아웃</a>
            </li>
            <li class="nav-item">
              <a class="nav-link" href="/member/info">마이페이지</a>
            </li>
            {% raw %}{% else %}{% endraw %}
            <li class="nav-item">
              <a class="nav-link" href="/member/login">로그인</a>
            </li>
            <li class="nav-item">
              <a class="nav-link" href="/member/join">회원가입</a>
            </li>
            {% raw %}{% endif %}{% endraw %}
          </ul>
        </div>
      </div>
    </nav>
    {% raw %}
    {% block content %}
    {% endblock %}
    {% endraw %}
  <script src="<https://cdn.jsdelivr.net/npm/bootstrap@5.0.2/dist/js/bootstrap.bundle.min.js>" integrity="sha384-MrcW6ZMFYlzcLA8Nl+NtUVF0sA7MsXsP1UyJoMp4YLEuNSfAP+JcXn/tWtIaxVXM" crossorigin="anonymous"></script>
</body>
</html>
```

다음, 위 템플릿 내용을 상속받을 템플릿들은 아래 형태로 작성을 해주면 된다.

아래에서 block content와 endblock 사이의 코드가 위 base.html의 block content와  endblock 사이에 들어간다고 보면 된다.

```html
{% raw %}{% extends "base.html" %}{% endraw %}

{% raw %}{% block content %}{% endraw %}
    <div class="container">
				<h1> 안녕하세요~ </h1>
		</div>
{% raw %}{% endblock %}{% endraw %}
```

### 게시판 댓글작성

DB에서 board 테이블과 reply 테이블을 만든 뒤 reply 테이블에는 게시물 번호를 참조하는 board_num 컬럼을 추가해줬다. 각 게시물마다 해당 게시물 번호를 참조하는 댓글을 불러와서 표시하도록 했는데, 플라스크의 Jinja2 템플릿 엔진 코드를 사용해서 댓글을 반복해서 출력하도록 하고, 자바스크립트를 사용해서 삭제 기능을 다음과 같이 구현했다.

```html
<script>
    function delreply(board_num, reply_num){
        location.href="/board/delreply?num="+board_num+"&reply_num="+reply_num;
    }
</script>

<form action="/board/reply" method="post">
    <input type="hidden" name="board_num" value="{{b.num}}">
    <table class="table table-bordered mt-3 mb-5 pb-5 col-8">
        {% raw %}{% if 'id' in session %}{% endraw %}
        <tr class="table-info"><th>댓글작성</th></tr>
        <tr><td class="my-0">
            <table class="table table-borderless my-0 py-0">
                <tr class="px-0 py-0"><td class="pb-0">
                    <textarea class="form-control" name="content" rows="2"></textarea>
                </td></tr>
                <tr class="px-0 py-0 d-grid justify-content-end"><td>
                    <input type="submit" class="btn btn-primary btn-outline" value="댓글입력">
                </td></tr>
            </table>
        </td></tr>
        {% raw %}
        {% endif %}
        {% for i in r %}
        {% endraw %}
        <tr class="table-secondary"><th>작성자: 작성일: </th></tr>
        <tr><td style="white-space:pre-wrap">
            <a href="javascript:delreply('','');" style="color:red">삭제</a></td></tr>
        {% raw %}{% endfor %}{% endraw %}
    </table>
</form>
```

### 이용자 통계

서비스 이용자가 입력한 정보를 바탕으로 서버에서 값을 계산하여 출력하는 것이 이번 프로젝트의 가장 핵심이었다고 할 수 있다. 서비스에서 구현한 기능들 중 "가장 자주 사용한 대여소" 비교 부분만 보자면...

우선 따릉이 대여소(보관소) 정보 csv 파일에서 보관소명을 가져와서 사용내역 입력시에 존재하는 보관소 중에서 선택하여 기록을 저장하도록 했다.

- history.py

```python
class HistoryDao:
		def selectStations(self):
		    data = pandas.read_csv('static/data/station_list.csv', encoding='cp949')
		    lst = data['보관소명']
		    res = lst.tolist()
		    return res

class HistoryService:
		def getStationList(self):
        return self.dao.selectStations()
```

- history_route.py

```python
@bp.route('/add')
def add_form():
    stlst = history_service.getStationList()
    return render_template('history/add.html', stlst=stlst)
```

- add.html

```html
<div class="col">
    <label for="rent_station" class="form-label fw-bold">대여한 보관소</label>
    <select class="form-select" id="rent_station" name="rent_station">
        <option selected>보관소명</option>
        {% raw %}{% for s in stlst %}{% endraw %}
        <option value="{% raw %}{{s}}{% endraw %}">{% raw %}{{s}}{% endraw %}</option>
        {% raw %}{% endfor %}{% endraw %}
    </select>
</div>
```

사용한 보관소 기록이 DB에 저장되면 해당 유저 id로 사용한 보관소 내역을 모두 가져와 리스트로 만든 뒤 리스트 안에서 가장 많은 요소가 무엇인지를 반환하도록 했다.

```python
class HistoryDao:
		def selectUserStations(self, id):
        self.connect()
        cur = self.conn.cursor()
        sql = 'select rent_station, return_station from history where member_id=%s'
        vals = (id,)
        cur.execute(sql, vals)
        stats = []
        for row in cur:
            stats.append(row[0])
            stats.append(row[1])
        self.disconnect()
        return stats

    def calcFavStn(self, id):
        stations = self.selectUserStations(id)
        count_list=[]
        for i in stations:
            count_list.append(stations.count(i))
        return stations[count_list.index(max(count_list))]

class HistoryService:
		def getFavStn(self, id):
        return self.dao.calcFavStn(id)
```

이 외에도 카카오맵 API를 활용하여 따릉이 설치현황을 보여주는 기능도 있고, 회원 로그인 상태에 따라 보여지는 메뉴들이 달라지게 하느 부분 등도 구현이 되어있으며 최종적으로 아래의 모습으로 서비스가 완성이 되었다.

<center><img src="https://raw.githubusercontent.com/lucathree/lucathree.github.io/master/assets/images/2021/2021-07-28/1.png"></center>

<center><img src="https://raw.githubusercontent.com/lucathree/lucathree.github.io/master/assets/images/2021/2021-07-28/2.png"></center>

<center><img src="https://raw.githubusercontent.com/lucathree/lucathree.github.io/master/assets/images/2021/2021-07-28/3.png"></center>

<center><img src="https://raw.githubusercontent.com/lucathree/lucathree.github.io/master/assets/images/2021/2021-07-28/4.png"></center>

<center><img src="https://raw.githubusercontent.com/lucathree/lucathree.github.io/master/assets/images/2021/2021-07-28/5.png"></center>

<center><img src="https://raw.githubusercontent.com/lucathree/lucathree.github.io/master/assets/images/2021/2021-07-28/6.png"></center>

자세한 작업 결과물은 아래 깃허브 링크를 통해 확인이 가능하다.

**프로젝트 결과물:**

[Code-Practice/Mini-Projects/3rd Project - BikeSeoul at main · lucathree/Code-Practice](https://github.com/lucathree/Code-Practice/tree/main/Mini-Projects/3rd Project - BikeSeoul)