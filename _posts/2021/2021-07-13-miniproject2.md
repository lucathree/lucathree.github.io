---
title: "개인 프로젝트: 공공데이터를 활용한 GUI 서비스 구현"
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
- 미니프로젝트
- 플레이데이터
- 인공지능SW개발자과정
toc: true
toc_sticky: true
---


SW개발교육을 시작하고 어느덧 6주가 흘렀다. 지난 미니 프로젝트에서 가장 아쉬웠던 부분이 모든 것이 텍스트기반에 파이썬 콘솔을 통해서만 구현할 수 밖에 없다는 것이었는데 드디어 수업에서 tkinter를 활용하여 GUI를 구현하는 방법에 대한 부분까지 진도를 나가게 되었다.

과제만 진행하는 것으로는 부족하다는 생각이 들어 안 그래도 개인 프로젝트를 해볼까 싶던 참에 수업 중 복습 대신 개인 공부를 더 하고 싶은 사람은 수업 내용 및 공공데이터 활용을 기반으로 프로젝트를 진행해도 좋다고하여 이번에는 팀 대신 개인 작업을 진행해보았다.

## 프로젝트 개요

### 목표: 현재까지 배운 내용들의 복습 & 응용

지난 미니 프로젝트 이후 추가로 배운 내용들은 다음과 같았다:

- numpy 및 pandas를 사용한 데이터 처리
- 외부 파일 또는 API를 연동하여 원하는 결과 도출하기
- matplotlib을 활용한 그래프 생성
- 멀티스레드를 활용한 프로그래밍
- 소켓을 활용한 네트워크 연결
- tkinter를 통한 GUI 구축

학습 내용중에는 이론적인 부분들도 많았고, 각 내용들의 실습을 해보기는 했지만 응용하여 프로그래밍을 해보지는 않았기 때문에 **전부 다!** 이번 프로젝트를 통해 복습을 해보고 추가로 그 전에 배운 클래스를 활용한 객체 지향 프로그래밍과 MySQL을 통한 DB 연동까지 해보기로 마음먹었다.

<center>![1](https://raw.githubusercontent.com/lucathree/lucathree.github.io/master/assets/images/2021/2021-07-13/(1).jpg){: width="400px"}</center>

어째서 이렇게 의욕이 넘쳤었는지는 모르겠지만 아무튼 프로젝트를 시작하기 전에는 머릿속으로 어떻게 각 기능들을 활용할 것인지 계획들이 착착 세워졌었고 주말 동안 손쉽게 프로젝트를 마무리 할 수 있을 것이라고 생각했다. 그러나...

### 주제 & 기능

어떤 공공데이터를 사용하면 좋을지 고민하던 중 최근에 전기차에 대한 관심이 생겨 전국에 전기차 충전소가 얼마나 있는지 궁금하여 찾아보니 마침 **한국 전력공사 전기차 충전소 오픈 API**가 존재했다.

그래서 해당 API를 사용해서 전국 전기차 충전소 현황 조회 서비스를 만들어보기로 결정하고, 다음과 같이 기능들을 구현해 보기로 계획을 세웠다.

**주제: 전국 전기차 충전소 현황 조회 서비스**

**기능**

- 지역별 충전소 리스트 검색

  - 지역 이름으로 검색해서 충전소 리스트 출력
  - 전체 검색 결과 수 표시
  - 충전소 별 상세 현황 조회

  ⇒ numpy, pandas를 활용한 공공데이터 처리 연습

- 충전소 운영 현황 그래프 출력

  ⇒ matplotlib을 활용한 데이터 시각화 연습

- 서비스 이용을 위한 회원가입 & 로그인

- 가상 관리자 및 사용자 간 채팅을 통한 충전소 사용 예약

- 예약 현황 확인 / 취소

  ⇒ DB 연동, 네트워크/스레드 응용, GUI 구현 연습

## 작업 과정 (이라 쓰고 '삽질의 기록'이라 읽는다)

### 오픈 API 활용

사용한 '한국 전력공사 전기차 충전소 오픈 API'는 요청 시 입력 항목으로 서비스키(ServiceKey), 한 페이지당 보여줄 결과 수(numOfRows), 확인할 페이지 번호(pageNo), 그리고 충전소 주소(addr)를 필요로 했다. (상세정보: https://www.data.go.kr/tcs/dss/selectApiDataDetailView.do?publicDataPk=3068728)

이전에 접했던 API들과는 달리 보고 싶은 결과의 페이지 번호와 결과 수를 미리 정해야하고, 정확한 충전소를 찾기 위해 충전소ID나 명칭이 아닌 충전소 주소를 입력해야 하는 점이 작업을 조금 까다롭게 했지만 다행히 충전소 주소는 주소의 일부분만 입력을 해도(예, '서울') 원하는 결과를 찾을 수 있었기 때문에 우려했던 것과는 달리 오히려 지역 별로 검색을 하기가 용이했다.

이 때 반환되는 결과에서 서비스 구현을 위해 내가 필요로 했던 데이터는 아래와 같았으며, 받은 데이터를 pandas를 통해 데이터프레임으로 변환하여 사용하였다.

- 반환된 결과 수 (totalCount)
- 충전소 주소 (addr)
- 충전기ID (cpId)
- 충전기명칭 (cpNm)
- 충전기 상태 코드 (cpStat): 1-충전가능, 2-충전 중, 3-고장/점검, 4-통신장애, 5-통신미연결
- 충전소명칭 (csNm)

**작업결과 (일부)**

```python
class StationVo:
    def __init__(self, cpid=None, csNm=None, cpNm=None, addr=None, cpStat=None):
        self.cpid = cpid
        self.csNm = csNm
        self.cpNm = cpNm
        self.addr = addr
        self.cpStat = cpStat

    def __str__(self):
        return f"충전소ID:{self.cpid}\\n충전소명:{self.csNm}\\n충전기타입:{self.cpNm}\\n주소:{self.addr}\\n상태:{self.cpStat}"

class StationDao:
    def __init__(self):
        self.url = "<http://openapi.kepco.co.kr/service/EvInfoServiceV2/getEvSearchList>"
        self.key = "1vpUi9jK%2B%2FJBVqaJ6TDWANcFxIFeuXBQRAljiZth0kzAvawz7gHoAgqwY3hDOFDi0SAcOe5Vz%2BE3ErfXvypDWw%3D%3D"

    def get_root(self, page_no, num_row, area):
        html = requests.get(self.url + '?pageNo=' + str(page_no) + '&numOfRows=' + str(
            num_row) + '&ServiceKey=' + self.key + '&addr=' + area).text
        root = BeautifulSoup(html, 'lxml-xml')
        return root

    def get_total(self, area):
        r = self.get_root(1, 10, area)
        total = r.select('totalCount')[0].string
        return total

    def get_cslist(self, area):
        page = 1
        cs_list = []

        while True:
            r = self.get_root(page, 200, area)
            lis = r.find_all('item')
            if len(lis) == 0:
                break
            for i in lis:
                if i.cpStat.string == '1':
                    row = [i.cpId.string, i.csNm.string, i.cpNm.string, i.addr.string, '충전가능']
                elif i.cpStat.string == '2':
                    row = [i.cpId.string, i.csNm.string, i.cpNm.string, i.addr.string, '충전중']
                elif i.cpStat.string == '3':
                    row = [i.cpId.string, i.csNm.string, i.cpNm.string, i.addr.string, '고장/점검']
                elif i.cpStat.string == '4':
                    row = [i.cpId.string, i.csNm.string, i.cpNm.string, i.addr.string, '통신장애']
                elif i.cpStat.string == '5':
                    row = [i.cpId.string, i.csNm.string, i.cpNm.string, i.addr.string, '통신미연결']
                else:
                    row = [i.cpId.string, i.csNm.string, i.cpNm.string, i.addr.string, '알수없음']
                cs_list.append(row)
            page += 1
        return cs_list

    def make_df(self, area):
        lst = self.get_cslist(area)
        columns = ['ID', '충전소명', '충전기타입', '주소', '상태']
        return pd.DataFrame(lst, columns=columns)
```

### 기능 구조

이번 프로젝트에서 사실 필수적인 부분은 아니었지만 DB 연동 연습을 위해 최소한의 테이블을 다음과 같이 구축해서 사용했다.

- users
  - user_id (primary), password, nickname
- reservations
  - reservation_id (primary), station, rsvtime, user_id (foreign), status

그리고 파일 구조는 주요 기능 별로 나눠서 다음과 같이 작업을 진행했는데, 각 기능 별 vo, dao, service를 나눠서 작업을 하기는 했지만 이게 가장 효율적인 구조인지 감이 오질 않아서 아직 내가 객체지향 모델링에 대한 개념을 완벽하게 이해하지 못했음을 절실히 느꼈다.

- userModel: 회원가입 및 로그인과 관련된 기능들로 구성
- stationModel: 전기차 충전소 정보를 가져오고 출력하는 기능들 위주로 구성
- reservationModel: 충전소 예약 및 채팅을 하기 위한 기능들 위주로 구성
- main: 기능들을 모아 하나의 gui로 구현

### 실수와 삽질...

위의 내용만 보면 그래도 뭔가 잘 정리가 되어있지만, 사실 아직 너무나도 부족한 실력에 비해 넘치는 의욕이 만들어낸 스노우볼은 결국 주말 시간을 모두 프로젝트에 빼앗겼음에도 내가 의도했던 기능들은 완벽하게 구현해내지 못하는 실수와 삽질의 굴레 속으로 빠트려놓았다...

머릿 속으로 가장 효율적이라고 생각했던 작업과정은 사실 이랬는데,

**1. 기능 구조 정의** → **2. 주피터 노트북을 통한 1차 기능 구현** → **3. 구현한 기능들을 클래스와 메서드로 정리** → **4. 기능을 GUI로 최종 구현**

막상 작업을 해보니 주피터 노트북으로 하나하나의 기능을 만들었다고 해도 이걸 다시 클래스와 메서드로 만들려면 단순히 복사 붙여넣기 하는 것 이상의 작업을 필요로 했다.

게다가 처음 작업을 시작했을 때는 데이터프레임을 만드는 것보다는 API를 통해 가져온 리스트를 바로 활용하는 편이 더 효율적일 것이라는 생각이 들어 리스트 중심으로 작업을 진행했는데, 이 부분 또한 객체 중심으로 바꾸는 과정에서 데이터프레임을 쓰는 게 훨씬 효율적임을 깨닫고 결국 모든 코드를 갈아 엎을 수 밖에 없었다...

<center>![2](https://raw.githubusercontent.com/lucathree/lucathree.github.io/master/assets/images/2021/2021-07-13/(2).jpg){: width="400px"} </center>

그러나 삽질은 여기에서 끝나지 않았는데...

클래스를 VO, DAO, Service로 구분해서 작업해놓고 이를 tkinter를 통해서 GUI로 붙이려고 하니 일단 GUI 자체가 사용자를 위한 기능이기에 사전에 Service를 먼저 구현해 놓을 필요가 없었고, 아직 tkinter에 대해 완벽하게 숙지가 안된 상태로 기능별 화면전환에 대한 욕심이 생겨서 어떻게든 구현을 하려고 계속 파다 보니 처음에 만들어놓은 클래스들과는 완전히 딴판의 결과물을 만들게 되었다.

이 과정에서 프로젝트 작업 시간이 부족하여 결국 채팅 기능은 머릿속으로만 그려놓고 구현은 손도 대지 못하게 되었다...

### 결과물

![3](https://raw.githubusercontent.com/lucathree/lucathree.github.io/master/assets/images/2021/2021-07-13/(3).PNG)

결국 처음 계획과 의욕과는 다르게 많은 부분들을 완성하지 못한채로 이번 프로젝트를 마무리 하게 되었다.

다행히도 가장 중요한 기능인 지역별 전기차 충전소를 검색하여 리스트를 출력받고, 리스트 안의 충전소를 선택하면 충전소 상세현황을 볼 수 있는 기능까지는 구현을 했다.

하지만, 충전소 현황 그래프 출력과 예약 및 취소 기능은 tkinter에 canvas를 사용해서 그래프를 출력하는 방법과 채팅을 통해 화면 속 텍스트가 계속해서 늘어날 경우의 처리 방식을 구현하지 못해 결국 최종적으로 GUI에 담지는 못했다.

부족한 부분이 너무 많고, 스스로도 정확한 작동원리를 이해하지 못한채 작성한 코드가 많아 상세한 코드 리뷰를 올리기에는 민망하지만 그래도 일단 작업물은 깃허브에 업로드를 해두었다.

**프로젝트 결과물:** [https://github.com/lucathree/Code-Practice/tree/main/Mini-Projects/2nd Project - 전기차충전소 현황조회/project2](https://github.com/lucathree/Code-Practice/tree/main/Mini-Projects/2nd Project - 전기차충전소 현황조회/project2)

## 후기

이번 프로젝트를 끝내고 발표를 할 때 어떻게 작업을 했는가에 대한 설명보다는 처음으로 혼자 프로젝트 작업을 하며 얼마나 많이 부족했고 어떤 삽질을 했는지에 대한 부분을 주로 공유를 했다. 그리고 이 글 역시 같은 목적으로 작성하게 되었다.

팀으로 작업한 첫번째 미니 프로젝트에 비해서 두번째 프로젝트이자 혼자 하는 작업이기에 의욕이 훨씬 더 컸고, 그래서 의욕만 앞서나간 것 같아 아쉬움도 더 크다. 하지만 수업 중 강사님이 작성한 코드만 따라하며 복습을 했다면 결코 배운 내용들을 이번 프로젝트를 진행하며 학습한 것만큼 내 것으로 만들지 못했을 것이다.

결국 애초에 프로젝트 목표는 배운 내용을 복습하는 것이었고, 거기에 추가로 다양한 삽질을 하며 작업 방식이나 순서를 어떻게 잡고 진행해야 더 효율적으로 일을 할 수 있을지에 대한 감이나 클래스와 메서드를 활용한 객체 지향 프로그래밍에 대해서도 완벽하지는 못해도 조금 더 이해도를 높일 수 있었기에 공부에는 도움이 많이 되었다.

\+ 프로젝트 발표기한 때문에 강제로 프로젝트를 마무리 시키기는 했지만 원래 기획했던 내용은 모두 구현하지 못했기 때문에 약간의 시간이 흐른 뒤 다시 이 프로젝트로 돌아와 처음에 기획했던 대로 다시 마무리를 할 예정이다. 그 때는 자세한 코드 리뷰까지도 첨부하여 후기를 작성할 수 있도록 하려고 한다.
