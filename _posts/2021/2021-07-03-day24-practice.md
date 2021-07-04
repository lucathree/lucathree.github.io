---
title: "Python - [실습]API를 통한 버스 노선 정보 가져오기"
layout: single
author_profile: true
read_time: true
related: true
categories:
- Python
tags:
- Study Note
- Python
- Code Practice
- 플레이데이터
- 인공지능SW개발과정
toc: true
toc_sticky: true
---

# 서울시 버스 노선 정보 가져오기

**사용 API**

- 서울특별시_노선정보조회 서비스  (xml)
- 서울특별시_정류소정보조회 서비스 (xml)
- 공공데이터포털(https://www.data.go.kr/index.do)에서 필요한 오픈API를 찾아 활용신청을 하고 인증키를 발급받아서 사용했습니다.

**라이브러리 사용**


```python
import pandas as pd
import numpy as np
import requests
from bs4 import BeautifulSoup
```


## 1. 버스번호를 입력하여 버스 정보 가져오기
#### 노선 ID 찾기
- 발급받은 인증키를 사용하여 API의 노선정보조회 서비스(getBusRouteList) 연결
- 버스 노선 번호(strSrch)를 파라미터로 하여 노선ID(busRouteId) 반환


```python
key = #발급받은키
def getBusRouteId(strSrch):
    html = requests.get('http://ws.bus.go.kr/api/rest/busRouteInfo/getBusRouteList?'ServiceKey='+key+'&strSrch='+strCrch).text
    root = BeautifulSoup(html, 'html.parser')
    try:  #존재하지 않는 노선번호를 입력한 경우에 대한 예외처리
        loc = root.find('busrouteid')
        return loc.string
    except Exception as e:
        print(e)

getBusRouteId('273')
```


    '100100049'



#### 버스 정차 정류소 리스트 출력
- 발급받은 인증키를 사용하여 API의 노선별 경유 정류소 조회 서비스(getStationByRoute) 연결
- 노선ID(busRouteId)를 파라미터로 하여 해당 노선이 정차하는 정류소명(stationNm) 리스트를 출력
- 노선 운행 방향(direction)에 따라 정차하는 정류소가 다르거나 중복되어 표시될 수 있으므로 별도 구분


```python
def getStationList(routeid):
    html = requests.get('http://ws.bus.go.kr/api/rest/busRouteInfo/getStaionByRoute?ServiceKey='+key+'&busRouteId='+routeid).text
    root = BeautifulSoup(html, 'html.parser')
    items = root.find_all('itemlist')
    direction = root.find_all('direction')
    for i in set(direction):  #set() 함수를 통해 중복되어 표시되는 운행방향값들을 정리
        print(f'======= 운행방향: {i.string} =======')
        for j in items:
            if j.direction == i:  #itemlist 안의 direction 값이 일치하는 경우에만 정류소명을 출력
                print(f'- {j.stationnm.string}')
        print()
        
getStationList('100100049')
```

    ======= 운행방향: 중랑공영차고지 =======
    - 홍대입구역사거리
    - 홍대입구역(가상)
    - 홍대입구역
    - 동교동삼거리
    - 신촌오거리.현대백화점
    - 신촌오거리.2호선신촌역
    - 이대역
    - 웨딩타운
    - 아현역
    - 충정로역
    - 서대문역사거리
    - 서울역사박물관.경희궁앞
    - 광화문
    - 종로1가
    - 종로2가
    - 종로3가.탑골공원
    - 종로4가.종묘
    - 종로5가.광장시장
    - 종로5가.효제초등학교
    - 종로5가.효제동.김상옥의거터
    - 방송통신대.이화장
    - 혜화역.마로니에공원
    - 혜화역.동성중고(장면총리가옥)
    - 삼선교.한성대학교.조소앙활동터
    - 돈암초교입구
    - 삼선동주민센터
    - 성북구청.성북경찰서
    - 보문역
    - 안암동주민센터
    - 고려대이공대.고대병원
    - 안암역
    - 고려대학교앞
    - 국방연구원
    - 한국과학기술원.홍릉초등학교
    - 서울바이오허브.농촌경제연구원
    - 경희대입구
    - 회기시장
    - 경희중고
    - 외대앞
    - 외대역앞
    - 이경시장
    - 이문1동주민센터
    - 이문동쌍용아파트
    - 이문동현대아파트
    - 중화2동주민센터
    - 지하철7호선중화역3번출구
    - 지하철7호선중화역2번출구
    - 중흥초등학교
    - 이화연립경동제일교회앞
    - 쌍용아파트신내테크노타운앞
    - 엘지아파트앞
    - 신현중학교
    - 중랑구청
    - 동성프라자앞
    - 신내교회.신내데시앙아파트
    - 중랑공영차고지
    - 중랑공영차고지.신내역
    
    ======= 운행방향: 홍대입구역 =======
    - 중랑공영차고지.신내역
    - 신내교회.신내데시앙아파트
    - 동성프라자앞
    - 중랑구청
    - 신현중학교
    - 엘지아파트앞
    - 쌍용아파트신내테크노타운앞
    - 이화연립경동제일교회앞
    - 중흥초등학교
    - 지하철7호선중화역2번출구
    - 지하철7호선중화역3번출구
    - 중화2동주민센터
    - 이문동현대아파트앞
    - 이문동쌍용아파트
    - 이문동삼익아파트
    - 종로사약국
    - 외대역앞
    - 외대앞
    - 경희중고
    - 회기시장
    - 경희대입구
    - 서울바이오허브.농촌경제연구원
    - 한국과학기술원.홍릉초등학교
    - 국방연구원
    - 고대앞
    - 안암전철역
    - 고려대이공대.고대병원
    - 안암동주민센터
    - 보문역
    - 보문역
    - 성북구청.성북경찰서
    - 삼선동주민센터
    - 삼선교.한성대학교
    - 혜화역.서울연극센터(장면총리가옥)
    - 혜화역.서울대병원입구
    - 방송통신대.이화장
    - 이화동(이화장)
    - 원남동
    - 혜화경찰서
    - 종로3가.탑골공원
    - 종로2가
    - 종로1가
    - 광화문
    - 서울역사박물관.경희궁앞
    - 서대문역사거리
    - 충정로역
    - 아현역
    - 웨딩타운
    - 이대역
    - 신촌오거리.2호선신촌역
    - 신촌오거리.현대백화점
    - 산울림소극장
    - 삼진제약
    - 홍익대학교



#### 버스 첫차, 막차, 배차 시간 출력
- 발급받은 인증키를 사용하여 API의 노선 기본정보 항목 조회 서비스(getRouteInfo) 연결
- 노선ID(RouteId)를 파라미터로 하여 해당 노선의 첫차시간(firstBusTm), 막차시간(lastBusTm), 배차간격(term) 출력
- 버스시간 정보가 매일 업데이트되며, '2021070404100000' 형태로 정보가 제공되기 때문에 문자열 슬라이싱을 사용


```python
def getBusTime(routeid):
    html = requests.get('http://ws.bus.go.kr/api/rest/busRouteInfo/getRouteInfo?ServiceKey='+key+'&busRouteId='+routeid).text
    root = BeautifulSoup(html, 'html.parser')
    first = root.find('firstbustm')
    last = root.find('lastbustm')
    term = root.find('term')
    print('====== 버스 시간 정보 ======')
    print(f'첫차시간: {first.string[8:10]}시 {first.string[10:12]}분')
    print(f'막차시간: {last.string[8:10]}시 {last.string[10:12]}분')
    print(f'배차간격: {term.string}분')
getBusTime('100100049')
```

    ====== 버스 시간 정보 ======
    첫차시간: 04시 10분
    막차시간: 22시 30분
    배차간격: 8분



#### main()


```python
def main():
    bus_num = input('버스노선 번호 입력:')
    routeid = getBusRouteId(bus_num)
    getBusTime(routeid)
    getStationList(routeid)

main()
```

    버스노선 번호 입력:273
    ====== 버스 시간 정보 ======
    첫차시간: 04시 10분
    막차시간: 22시 30분
    배차간격: 8분
    ======= 운행방향: 중랑공영차고지 =======
    - 홍대입구역사거리
    - 홍대입구역(가상)
    - 홍대입구역
    - 동교동삼거리
    - 신촌오거리.현대백화점
    - 신촌오거리.2호선신촌역
    - 이대역
    - 웨딩타운
    - 아현역
    - 충정로역
    - 서대문역사거리
    - 서울역사박물관.경희궁앞
    - 광화문
    - 종로1가
    - 종로2가
    - 종로3가.탑골공원
    - 종로4가.종묘
    - 종로5가.광장시장
    - 종로5가.효제초등학교
    - 종로5가.효제동.김상옥의거터
    - 방송통신대.이화장
    - 혜화역.마로니에공원
    - 혜화역.동성중고(장면총리가옥)
    - 삼선교.한성대학교.조소앙활동터
    - 돈암초교입구
    - 삼선동주민센터
    - 성북구청.성북경찰서
    - 보문역
    - 안암동주민센터
    - 고려대이공대.고대병원
    - 안암역
    - 고려대학교앞
    - 국방연구원
    - 한국과학기술원.홍릉초등학교
    - 서울바이오허브.농촌경제연구원
    - 경희대입구
    - 회기시장
    - 경희중고
    - 외대앞
    - 외대역앞
    - 이경시장
    - 이문1동주민센터
    - 이문동쌍용아파트
    - 이문동현대아파트
    - 중화2동주민센터
    - 지하철7호선중화역3번출구
    - 지하철7호선중화역2번출구
    - 중흥초등학교
    - 이화연립경동제일교회앞
    - 쌍용아파트신내테크노타운앞
    - 엘지아파트앞
    - 신현중학교
    - 중랑구청
    - 동성프라자앞
    - 신내교회.신내데시앙아파트
    - 중랑공영차고지
    - 중랑공영차고지.신내역
    
    ======= 운행방향: 홍대입구역 =======
    - 중랑공영차고지.신내역
    - 신내교회.신내데시앙아파트
    - 동성프라자앞
    - 중랑구청
    - 신현중학교
    - 엘지아파트앞
    - 쌍용아파트신내테크노타운앞
    - 이화연립경동제일교회앞
    - 중흥초등학교
    - 지하철7호선중화역2번출구
    - 지하철7호선중화역3번출구
    - 중화2동주민센터
    - 이문동현대아파트앞
    - 이문동쌍용아파트
    - 이문동삼익아파트
    - 종로사약국
    - 외대역앞
    - 외대앞
    - 경희중고
    - 회기시장
    - 경희대입구
    - 서울바이오허브.농촌경제연구원
    - 한국과학기술원.홍릉초등학교
    - 국방연구원
    - 고대앞
    - 안암전철역
    - 고려대이공대.고대병원
    - 안암동주민센터
    - 보문역
    - 보문역
    - 성북구청.성북경찰서
    - 삼선동주민센터
    - 삼선교.한성대학교
    - 혜화역.서울연극센터(장면총리가옥)
    - 혜화역.서울대병원입구
    - 방송통신대.이화장
    - 이화동(이화장)
    - 원남동
    - 혜화경찰서
    - 종로3가.탑골공원
    - 종로2가
    - 종로1가
    - 광화문
    - 서울역사박물관.경희궁앞
    - 서대문역사거리
    - 충정로역
    - 아현역
    - 웨딩타운
    - 이대역
    - 신촌오거리.2호선신촌역
    - 신촌오거리.현대백화점
    - 산울림소극장
    - 삼진제약
    - 홍익대학교


​    

## 2. 정거장명을 입력하여 정차하는 버스 목록 출력
#### 정거장이름 검색 후 노선ID 선택
- 발급받은 인증키를 사용하여 API의 명칭별 정류소 목록 조회 서비스(getStationByName) 연결
- 정류소검색어(stSrch)를 파라미터로 하여 검색어가 포함된 정류소명(stNm)과 정류소ID(arsId) 반환
- 검색되는 정류소가 여러개일 수 있기 때문에 결과 중 하나를 선택


```python
key = #발급받은키
def getStationId(stSrch):
    html = requests.get('http://ws.bus.go.kr/api/rest/stationinfo/getStationByName?ServiceKey='+key+'&stSrch='+stSrch).text
    root = BeautifulSoup(html, 'html.parser')
    items = root.find_all('itemlist')
    item_num = 1
    for i in items:
        print(f'====== 검색결과:{item_num} ======  ')
        print(f'정류소명:{i.stnm.string}\n고유번호:{i.arsid.string}')
        item_num += 1
    s = input('\n찾으시는 정류소의 검색결과 번호를 입력해주세요:')
    try:  #정류소 검색결과가 없거나 잘못된 번호를 선택할 경우에 대한 예외처리
        res = items[int(s)-1].arsid.string
    except Exception as e:
        print(e)
    else:
        return res
    
getStationId('뱅뱅사거리')
```

    ====== 검색결과:1 ======  
    정류소명:뱅뱅사거리
    고유번호:22005
    ====== 검색결과:2 ======  
    정류소명:뱅뱅사거리
    고유번호:22006
    ====== 검색결과:3 ======  
    정류소명:뱅뱅사거리
    고유번호:22125
    ====== 검색결과:4 ======  
    정류소명:뱅뱅사거리
    고유번호:22126
    ====== 검색결과:5 ======  
    정류소명:뱅뱅사거리
    고유번호:22887
    ====== 검색결과:6 ======  
    정류소명:뱅뱅사거리(경유)
    고유번호:78037
    ====== 검색결과:7 ======  
    정류소명:뱅뱅사거리(경유)
    고유번호:78259
    ====== 검색결과:8 ======  
    정류소명:뱅뱅사거리.구영동중학교
    고유번호:23319
    ====== 검색결과:9 ======  
    정류소명:뱅뱅사거리.역삼동
    고유번호:23294
    ====== 검색결과:10 ======  
    정류소명:뱅뱅사거리뱅뱅프라자
    고유번호:23321
    
    찾으시는 정류소의 검색결과 번호를 입력해주세요:4
    
    '22126'



#### 정류소에 정차하는 버스 리스트 출력
- 발급받은 인증키를 사용하여 API의 명칭별 정류소 목록 조회 서비스(getRouteByStation) 연결
- 정류소 고유번호(arsId)를 파라미터로 하여 해당 정류소에 정차하는 버스노선(busRouteNm) 리스트 반환


```python
def getBusNumber(arsid):
    html = requests.get('http://ws.bus.go.kr/api/rest/stationinfo/getRouteByStation?ServiceKey='+key+'&arsId='+arsid).text
    root = BeautifulSoup(html, 'html.parser')
    items = root.find_all('itemlist')
    print('====== 정류소 정차 버스 ======')
    for i in items:
        print(f'- {i.busroutenm.string}')

getBusNumber('22126')
```

    ====== 정류소 정차 버스 ======
    - 461
    - 641
    - 3012
    - 4319



#### main()


```python
def main():
    name = input('검색할 정류소 이름:')
    arsid = getStationId(name)
    print()
    getBusNumber(arsid)

main()
```

    검색할 정류소 이름:뱅뱅사거리
    ====== 검색결과:1 ======  
    정류소명:뱅뱅사거리
    고유번호:22005
    ====== 검색결과:2 ======  
    정류소명:뱅뱅사거리
    고유번호:22006
    ====== 검색결과:3 ======  
    정류소명:뱅뱅사거리
    고유번호:22125
    ====== 검색결과:4 ======  
    정류소명:뱅뱅사거리
    고유번호:22126
    ====== 검색결과:5 ======  
    정류소명:뱅뱅사거리
    고유번호:22887
    ====== 검색결과:6 ======  
    정류소명:뱅뱅사거리(경유)
    고유번호:78037
    ====== 검색결과:7 ======  
    정류소명:뱅뱅사거리(경유)
    고유번호:78259
    ====== 검색결과:8 ======  
    정류소명:뱅뱅사거리.구영동중학교
    고유번호:23319
    ====== 검색결과:9 ======  
    정류소명:뱅뱅사거리.역삼동
    고유번호:23294
    ====== 검색결과:10 ======  
    정류소명:뱅뱅사거리뱅뱅프라자
    고유번호:23321
    
    찾으시는 정류소의 검색결과 번호를 입력해주세요:4
    
    ====== 정류소 정차 버스 ======
    - 461
    - 641
    - 3012
    - 4319

