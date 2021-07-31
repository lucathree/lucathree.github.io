---
title: "탐색 알고리즘 - 선형탐색/이진탐색"
excerpt: "플레이데이터 알고리즘 스터디 2주차 주제, 탐색 알고리즘에 대한 공부기록"
layout: single
author_profile: true
read_time: true
related: true
categories:
- Algorithm
tags:
- Study Note
- 알고리즘
- 자료구조
- 알고리즘이론
- Python
- 플레이데이터
- 인공지능SW개발과정
toc: true
toc_sticky: true
---



이 포스팅은 플레이데이터 알고리즘 스터디 2주차 "완전탐색/이분탐색" 강의 및 “Do it! 자료구조와 함께 배우는 알고리즘 입문, 파이썬편 - 3장 검색 알고리즘”을 참고하여 공부한 내용을 개인적으로 다시 정리하여 기록했습니다.

---------

<br/>

# 탐색(검색) 알고리즘이란?

탐색(검색)은 수많은 데이터의 집합 속에서 원하는 데이터/값을 가진 원소를 찾아내는 알고리즘.

웹에서의 검색뿐만아니라 신용카드나 버스카드 역시 검색 알고리즘을 사용한다.

## 조건과 키(key)

모든 검색에는 **조건과 키**가 존재한다.

먼저, 조건은 다음과 같을 수 있다:

- 국적이 한국인 사람을 찾습니다.
- 나이가 21세 이상 27세 미만인 사람을 찾습니다.
- 이름에 '민'자가 들어간 사람을 찾습니다.

위 조건들은 모두 어떠한 항목에 주목을 하고 있는데, 조건에서 주목하는 항목을 키라고 한다. 대부분 키는 데이터의 일부이며, 위의 조건들에 대한 검색 알고리즘을 만들 때 키는 다음과 같이 설정할 수 있다:

- 키 = 국적, 키값이 "한국"과 **일치**하도록 지정
- 키 = 나이, 키값의 **구간**을 "21세 이상 27세 미만"으로 지정
- 키 = 문자, 검색결과가 키값('민')에 가깝도록 지정

## **탐색의 종류**

탐색 알고리즘의 종류에는 다양한 방법이 존재하는데, 우선 자료구조에 따라서는 **배열 검색, 연결 리스트 검색, 이진검색트리 검색** 등이 있다.

이 중 먼저 **배열 검색**을 공부하는데, 배열 검색은 또 **선형 검색, 이진 검색, 해시법**으로 나눠진다.

그 외에도 **깊이우선탐색, 너비우선탐색, KMP, BM** 등이 존재한다.

<br/>

# 선형탐색 (linear search)

앞서 [CS50 - 알고리즘 파트 공부기록](https://lucathree.github.io/computer science/cs50-4/)의 내용대로 선형으로 늘어선 배열을 맨 앞부터 끝까지 하나씩 스캔하며 찾는 값이 있는지를 검사하는 알고리즘이다. 순차적으로 진행되기 때문에 **순차 탐색(Sequential Search)**이라고도 한다.

무작위로 늘어놓은 데이터 집합에서 높은 정확도로 검색이 가능하지만, 원하는 값을 찾을 때까지 모든 데이터를 순차적으로 확인 해야하기 때문에 효율성이 떨어진다.

## **구현방법**

### 반복문

- while문을 사용한 구현방법

  ```python
  def solution(array, answer):
  		i = 0
  		while True:
  				if trump[array] == answer:
  						return i
  				elif i == len(array):
  						return -1
  				i+=1
  ```

- for문을 사용한 구현방법

  ```python
  def solution(array, answer):
  		for i in range(len(array)):
  				if array[i] == answer:
  						return i
  		return -1
  ```

- **보초법 (Sentinel Method)**

  위의 구현예시에서는 1.값을 찾았을 때 와 2.값을 찾지 못했을 때 두 가지 상황에 대한 조건판단이 필요했다. 그런데 배열 끝에 원하는 값을 추가하여 선형검색을 진행시키면 종료판단을 한번으로 줄일 수 있다.

  ```python
  def solution(array, answer):
  		array.append(answer)
  		i = 0
  		while True:
  				if array[i] == answer:
  						break
  				i+=1
  		return -1 if i == len(array) else i
  ```

  아니면 그냥 이렇게 해도 될듯?

  ```python
  def solution(array, answer):
  		i = 0
  		while i < len(array):
  				if array[i] == answer:
  						return i
  				i+=1
  		return -1
  ```

## 완전탐색

가능한 모든 경우의 수를 다 체크해서 원하는 값을 찾는 방법.

완전탐색을 하는 방법은 다양하지만 배열로 된 자료구조에서는 모든 경우의 수를 찾기 위한 방법으로 반복문을 통한 **선형검색**을 사용한다. 무식하게 진행하기 때문에 **브루트 포스 (Brute Force)** 라고도 한다.

선형검색 외에도 완전탐색을 하는 방법은 **비트 마스크, 백트래킹, 순열, 너비우선탐색, 깊이우선탐색, 재귀함수** 등이 있다.

- 재귀함수를 통한 선형검색

  재귀함수는 선형 검색 외에도 동적 계획법, 백트래킹, 탐욕법 등에서도 사용된다

  ```python
  def solution(array, i, answer):
  		for array[i] == answer:
  				return loc
  		else:
  				return solution(array, i+1, answer)
  ```

<br/>

# **이진탐색 (Binary Search)**

이분탐색이라고도 표현하며 오름차순 또는 내림차순으로 정렬된 리스트에서 특정 값의 위치를 찾는 알고리즘. 중간의 값을 선택하여 찾고자 하는 값과의 크고 작음을 비교하는 방법.

시간복잡도가 O(log n)으로 선형검색보다 훨씬 빠르게 처리가 가능하다.

['CS50 - 알고리즘 공부기록'](https://lucathree.github.io/computer science/cs50-4/) 포스팅에 더 자세한 설명이 되어있다.

## 구현방법

```python
def solution(trump, ans):
		left = 0  #왼쪽 시작값
		right = len(trump)-1  #오른쪽 끝값
		while(left<=right):  #left가 right 보다 작거나 같다면
				mid = (left+right)//2  #mid, 정중앙값 계산
				if trump[mid] == 8:
						return mid
				elif trump[mid] < 8:  #mid값이 정답보다 작을 경우 중앙의 오른쪽 값들을 검색
						left = mid+1  #왼쪽 시작값을 mid+1로 설정
				elif trump[mid] > 8:  #mid값이 정답보다 클 경우 중앙의 왼쪽 값들을 검색
						right = mid -1  #오른쪽 끝값을 mid-1로 설정
		return mid
```