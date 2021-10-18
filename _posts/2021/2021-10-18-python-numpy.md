---
title: "Python - numpy 기본 복습, 개념정리"
exerpt: "파이썬 기본기 복습. numpy 라이브러리의 개념과 ndarray 사용법, 주요 함수 정리"
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



## numpy란?

— 고성능 수치계산을 위해 C언어로 구현된 파이썬 라이브러리. numpy란 이름은 'Numerical Python'의 줄임말.

## numpy를 사용하는 이유

- 파이썬 리스트보다 처리속도가 빠르다.
- 파이썬 리스트보다 적은 메모리를 사용한다.
- 선형대수, 통계관련 함수들이 내장되어 있다.
- 브로드캐스팅

## ndarray

— numpy에서 사용되는 다차원 리스트 데이터 타입 객체

- 파이썬 리스트는 연속되지 않은 물리적 메모리로 이루어져 있는데 (참조값들을 사용한 명시적 loop 사용), ndarray는 연속된 메모리 vectorization을 사용하기 때문에 속도가 더 빠르다.

## 주요 numpy 함수

### **리스트 생성 관련**

- np.array() : 리스트를 아규먼트로 받아 ndarray 객체 생성
- np.arange() : 입력된 정수값에 따라 해당 길이의 넘파이 리스트 생성. 시작값과 끝값, 간격을 파라미터로 이용할 수도 있다.
- np.ones() : 입력된 shape의 1 로만 이루어진 넘파이 리스트 생성
- np.zeros() : 입력된 shape의 0 으로만 이루어진 넘파이 리스트 생성
- np.empty() : 입력된 shape의 초기화된 값들로 이루어진 넘파이 리스트 생성
- np.full() : 입력된 shape과 채울 값에 따라 넘파이 리스트 생성
- np.eye() : 단위 행렬(unit matrix) 또는 항등 행렬(identity matrix)라 불리는 주대각 원소가 모두 1이고 나머지는 모두 0인 n x n 정사각 행렬 생성
- np.linsapce() : 시작값, 끝값, 원소수를 파라미터로 받아 시작값부터 끝값까지 입력된 원소수만큼 균등한 간격으로 나눠진 넘파이 리스트 생성
- np.reshape() : 기존 넘파이 리스트의 형태, 차원을 바꾸기 위해 사용. 단, 기존 리스트의 원소수가 바꾸려는 형태로 나뉘어 떨어질 수 있어야만 가능.

### random 서브모듈 (np.random)

- np.random.rand() : 0, 1 사이의 실수 분포로 랜덤한 ndarray 생성
- np.random.randn() : 정규분포(**n**ormal distribution)로 샘플링된 랜덤 ndarray 생성
- np.random.randint() : 특정 정수 사이에서 랜덤하게 샘플링
- np.random.seed() : 랜덤값을 동일하게 생성하기 위한 시드. 랜덤 서브모듈 함수 실행 전 사용
- np.random.choice() : 아규먼트로 주어진 1차원 ndarray로부터 랜덤샘플링. 정수를 입력하면 np.arange(해당숫자)로 간주
- np.random.uniform() : 균등분포 기준으로 샘플링하여 넘파이 리스트 생성
- np.random.normal() : randn() 과 파라미터만 다르고 동일하게 정규분포 기준으로 샘플링하여 넘파이 리스트 생성

### shape 변경

- np.reshape()
- ravel(), np.ravel() : 다차원배열을 1차원으로 변경. order 파라미터 값을 'C'로 하면 row 우선 변경, 'F'로 하면 column 우선 변경
- flatten() : 마찬가지로 다차원배열을 1차원으로 변경
- ravel과 flatten의 차이 : ravel은 얕은카피를 하여 참조값을 사용하기 때문에 원본 데이터에 영향을 주고, flatten은 딥카피를 하기 때문에 원본에 영향이 없음.

### 기본 연산 함수

— 브로드캐스팅이 가능하며 배열의 모양에 유의하여 사용. 일반 연산기호로도 연산이 가능하다.

- np.add(), + : 더하기
- np.subtract(), - : 빼기
- np.multiply(), * : 곱하기
- np.divide(), / : 나누기

### 통계 함수

- np.mean() : 평균
- np.max() : 최대값
- np.argmax() : 최대값의 인덱스 반환 (다차원배열의 경우에도 원소의 순서대로 표시)
- np.var() : 분산
- np.median() : 중앙값
- np.std() : 표준편차

### 집계 함수

- np.sum() : 합계
- np.cumsum() : 누적합계
- np.any(조건식) : 리스트 안의 값들 중 조건식에 맞는 값이 하나라도 있는지 확인하여 True, False 반환
- np.all(조건식) : 리스트 안의 모든 값이 조건식에 맞는지 확인하여 True, False 반환
- np.where(조건, 참일 때 값, 거짓일 때 값) : 조건에 따라 선별적으로 값을 선택, 변환하여 넘파이 리스트 반환

### 그 외 ndarray를 통해 할 수 있는 것들

- 인덱싱 / 슬라이싱
- axis 를 사용한 연산
- 조건식을 사용한 boolean indexing
- linalg 모듈을 이용한 선형대수 연산, 행렬곱 (matrix multiplication)
- matplotlib 과 연동하여 그래프 표현
