---
title: "동적계획법"
excerpt: "플레이데이터 알고리즘 스터디 7주차 주제, 동적계획법에 대한 공부기록"
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



## 동적계획법

다이나믹 프로그래밍 (Dynamic Programming, DP) 라고도 불리며, 하나의 큰 문제를 여러 개의 공통되는 작은 문제로 나누어 작은 문제의 정답들을 결합하여 알고리즘을 푸는 과정 (=규칙을 찾는 문제)

**대표적인 예: 점화식**

- 수열에서 n번째 항을 이전에 나온 항들을 이용하여 나타낸 공식. 규칙을 찾아낸다면 문제를 풀기가 수월해진다.

### 동적계획법 접근방법

**Bottom Up & Top Down**

- Bottom Up 방법

  작은 문제에서 큰 문제로 반복문 호출

- Top Down 방법

  큰 문제에서 작은 문제로 재귀 호출

### 피보나치 수열

**Bottom Up 접근**

1. 첫번째(n=0)와 두번째(n=1) 항의 값은 1
2. n=2 일 때, 값은 첫번째와 두번째 항의 합. 1 + 1 = 2
3. n=3, 값은 1 + 2 = 3
4. n=4, 값은 2 + 3 = 5
5. ....

```python
def fib(n):
		fibList = [1, 1]
		for i in range(2, n+1):
				fibList.append(fibList[i-2]+fibList[i-1])
		return fibList[-1]
```

**Top Down 접근**

1. 정답을 구할 n번째 값 선언
2. 이전 두개의 피보나치 수열 사용: fib(n-1), fib(n-2)
3. 위 두개의 값을 찾기 위해 그 이전 두개의 피보나치 수열 필요
   1. fib(n-1) : fib(n-2), fib(n-3)
   2. fib(n-2) : fib(n-3), fib(n-4)
4. 계속 내려가다 보면 결국 fib(0), fib(1) 값들을 필요로 하게 됨 → 모두 1 반환

```python
def fib(n):
		if n == 0 or n == 1:
				return 1
		else:
				return fib(n-1) + fib(n-2)
```

### 메모이제이션 (Memoization)

앞의 예시에서 n번째 값을 찾기 위해 재귀가 반복되고, 중복된 계산이 이루어진다. 메모이제이션을 앞에서 했던 계산을 저장하고 필요에 따라 불러서 중복된 계산이 발생하는 것을 방지.

```python
memo = {0:1, 1:1}  # 메모이제이션에 쓸 딕셔너리 선언 (배열도 ok)

def fib(n):
		# 저장된 값이 존재하는 지 확인
		if n in memo:
				return memo[n]
		# 없으면 피보나치 수열 계산
		else:
				result = fib(n-1) + fib(n-2)
				memo[n] = result  # 값을 딕셔너리에 저장
return result
```

- 배열 혹은 해시를 활용하는 것이 핵심

**예시) data = [3,4,5,6,1,2,5] 에서 이웃하지 않는 숫자들의 합의 최댓값은?**

- 값이 data[0] 하나만 있을 때, 최댓값 = 3 (s0)
- 값이 data[1] 까지 있을 때, s0과 data[1] 중 더 큰 값이 최댓값 = 4 (s1)
- 값이 data[2] 까지 있을 때, data[2]와 이웃하지 않은 최댓값 s0의 합 (3+5)과 이전 최댓값 s1 중 더 큰 값이 최댓값 = 8 (s2)
- 값이 data[3] 까지 있을 때, data[3]와 이웃하지 않은 최댓값 s1의 합 (6+4)과 이전 최댓값 s2 중 더 큰 값이 최댓값 = 10 (s3)
- 값이 data[4] 까지 있을 때, data[4]와 이웃하지 않은 최댓값 s2의 합(1+8)과 이전 최댓값 s3 중 더 큰 값이 최댓값 = 10 (s4)
- 값이 data[5] 까지 있을 때, data[5]와 이웃하지 않은 최댓값 s3의 합(2+10)과 이전 최댓값 s4 중 더 큰 값이 최댓값 = 12 (s5)
- 값이 data[6] 까지 있을 때, data[6]과 이웃하지 않은 최댓값 s4의 합(5+10)과 이전 최댓값 s5 중 더 큰 값이 최댓값 = 15 (s6)

```python
def solution(data):
		if len(data) == 1:
				return data[0]
		result = [data[0], max(data[0], data[1])
		for i in range(2, len(data)):
				result.append(max(result[i-1], result[i-2] + data[i])
		return result[-1]
```
