---
title: "Github 블로그에 플라스크 템플릿변수 문자형태로 올리기"
layout: single
author_profile: true
read_time: true
related: true
categories:
- Git
tags:
- 공부기록
- SelfStudy
- Github
toc: true
toc_sticky: true
---

조금전 플라스크를 활용한 웹서비스 프로젝트에 관한 글을 올리면서 이상한 문제를 마주쳤다.

평소처럼 포스트를 작성해서 git으로 커밋을 했는데 아래와 같은 이메일이 오는 것이다.

> The page build failed for the `master` branch with the following error: The tag `endblock` on line 238 in `_posts/2021/2021-07-28-miniproject.md` is not a recognized Liquid tag.

블로그 포스팅을 했는데 블로그에서는 내 글이 보이질 않는다. 이게 무슨일인가!

원인은, 프로젝트 코드 일부를 포스팅에 첨부했는데 그 안에 있는 플라스크 템플릿 코드가 지킬의 리퀴드태그와 형태가 같아서 문제를 일으키는 것이었다.

무슨 말이냐면, 나는 다음과 같이 플라스크에서 사용하는 Jinja2 템플릿 엔진 코드를 블로그 포스트에 올리고 싶은데,

```
{% raw %}{% block content %}{% endraw %}
```

깃허브 페이지를 구현하는데 쓰이는 지킬은 위 문자를 자기에게 내리는 명령어로 인식한다는 것이다.

하필이면 이걸 모른 상태에서 저 '{% raw %}{%  %}{% endraw %} 형태의 코드를 지난 포스팅에 아주 많이 사용했고 덕분에 정상적으로 페이지가 생성될 때까지 커밋과 되돌리기는 수십번은 해야했던 것 같다...

위 문제를 해결하려면 해당 문자열을 raw-endraw 코드로 감싸서 지킬에게 '이것은 문자다'라고 알려주면 된다.

```
{% raw %}{% raw %}{% endraw %}{% raw %}{% 엔드raw %}{% endraw %}
```

endraw를 중간에 넣으려니까 문자열이 끝난줄 알고 또 에러를 뱉어낸다... 그래서 어쩔 수 없이 endraw를 "엔드raw" 라고 썼는데 어쩔 수 없이 이미지로 보여주자면 이렇게 작성해야 한다...

<center><img src="https://raw.githubusercontent.com/lucathree/lucathree.github.io/master/assets/images/2021/2021-07-28.PNG"></center>

이 무슨...

아무튼 이것으로 이번 포스팅을 마친다.
