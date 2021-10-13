---
title: "Python - lambda 함수"
exerpt: "파이썬 기본기 복습. lambda 함수의 기본 및 filter, map, reduce 응용"
layout: single
author_profile: true
read_time: true
related: true
categories:
- Python
tags:
- Python
- Study Note
---



## **lambda 함수**

간단한 함수를 def 을 사용하는 대신 한 줄로 표현. 함수를 반복해서 사용하지 않는 경우에 유용하다.

기본 형태는 다음과 같다.

```python
lambda 파라미터1, 파라미터2, ... : 표현식
```

lambda 예약어 뒤에 사용할 파라미터를 적고 콜론 뒤에는 파라미터를 사용한 함수 내용을 적는다.

**예시)**

파라미터를 하나만 사용하는 경우

```python
square = lambda x : x**2
square(5)  # OUT: 25
```

여러 파라미터를 사용

```python
sum = lambda x, y, z : x + y + z
sum(10, 20, 30)  # OUT: 60
```

반드시 변수에 지정하지 않아도 다음처럼 아규먼트 값으로 바로 쓸 수도 있다.

```python
names = ['stephen', 'bob', 'chris', 'alex']
names.sort(key=lambda s:len(s))
print(names)  # OUT: ['bob', 'alex', 'chris', 'stephen']
```



## lambda가 자주 쓰이는 함수

### filter()

```python
filter(function, iterable)
```

여러 데이터들 중에서 일부를 추려내는 함수.

리스트나 튜플 같은 iterable 객체의 값들 중 function 의 조건에 맞는(True) 값들만을 반환한다.

function 파라미터에는 조건 함수가 들어가야 되는데, 이 부분을 lambda 함수로 대체할 수 있다.

**예) 리스트의 숫자들 중 짝수만 반환하기**

```python
nums = [1,2,3,4,7,9,10,14,12,11,16,21]
list(filter(lambda x:x%2==0, nums))  # OUT: [2, 4, 10, 14, 12, 16]
```

※ filter 함수의 반환값은 filter 객체이기 때문에 해당 객체를 다시 리스트나 튜플로 형변환을 시켜줘야 한다.

### map()

```python
map(function, iterable)
```

map 함수는 앞의 filter 함수처럼 여러 데이터를 지정된 함수로 처리하여 반환한다. 그래서 마찬가지로 function 파라미터에 lambda 함수를 입력할 수 있다.

**예) 리스트의 숫자를 제곱하여 반환**

```python
nums = [1,2,3,5,7]
list(map(lambda x:x**2, nums))  # OUT: [1, 4, 9, 25, 49]
```

※ map 함수 또한 반환값이 map 객체이기 때문에 해당 객체를 다시 리스트나 튜플로 형변환 시켜줘야 한다.

### reduce()

```python
from functools import reduce
reduce(function, sequence[, initial]) 
```

reduce 함수는 여러 데이터들을 이용한 누적 연산 결과를 반환한다.

python2 까지는 기본 내장함수였지만 python3 부터는 functools 모듈을 통해 임포트해서 사용해야한다고 한다.

앞의 다른 함수들처럼 function 파라미터의 조건에 따라 연산을 하며, 연속된 데이터들의 순서에 따라 연산한 값을 누적하여 다음 연산에 사용하며 더 이상 연산할 데이터가 없을 때까지 반복한다. 그리고 initial 파라미터를 통해 연산 초기값을 지정해 줄 수도 있다.

**예) 단순 누적연산**

```python
nums = [2,3,5,7,9]
reduce(lambda x,y:(x+y)*2, nums)  # OUT: 166
```

(2+3)*2 = 10

​	→ (10+5)*2 = 30

​		→ (30+7)*2 = 74

​			→ (74+9)*2 = 166

reduce 함수는 단순 연산뿐만 아니라 reduce 함수를 이용한 map, filter 함수 구현도 가능하며 빅데이터 처리를 위한 MapReduce 프레임워크까지 map과 reduce를 활용해서 만들어질 정도로 다양한 방법으로 사용이 가능하다고 하니 다음에 더 알아보도록 한다.
