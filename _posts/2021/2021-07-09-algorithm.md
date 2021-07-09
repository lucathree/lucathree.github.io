---
title: "스택과 큐"
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
- Python
- 플레이데이터
- 인공지능SW개발과정
toc: true
toc_sticky: true
---


이 포스팅은 플레이데이터 알고리즘 스터디 1주차 스택/큐 강의와 "Do it! 자료구조와 함께 배우는 알고리즘 입문, 파이썬편 - 4장 스택과 큐"를 사용하여 공부한 내용을 바탕으로 하고 있습니다.

------

&nbsp;

# 스택 (Stack)

데이터를 임시 저장할 때 사용하는 자료구조. 데이터의 입력과 출력 순서는 **LIFO(후입선출, Last-In First-Out)**로 가장 나중에 넣은 데이터를 가장 먼저 꺼내며, 리스트에 대해 한 쪽 방향에서만 접근이 가능하다.

대표적인 내장함수: **push(), peak(), pop()**

&nbsp;

## 스택의 구조

![1](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/77e7b609-5984-4164-afd3-3f15a61a6f70/stack.png)

- 책을 쌓듯이 데이터를 넣고 꺼내는 작업을 맨 위부터 수행한다.
- 작업이 수행되는 윗부분을 **꼭대기(top)**이라 하고, 아랫부분을 **바닥(bottom)**이라고 한다.
- **Push:** 스택의 top에 데이터를 넣는 작업
- **Pop:** top의 데이터를 꺼내는 작업
- **Peek:** top의 데이터를 확인하는 작업

  &nbsp;

## 스택의 활용 예

- 웹브라우저의 '이전 페이지', '다음 페이지' 기능
- 깊이 우선 탐색(DFS)

  &nbsp;

## 스택의 구현방법

**일반적인 자료구조의 구현방법:**

1. 직접구현

2. 이미 구현된 클래스 import

   *Python은 리스트를 스택으로 사용 가능하도록 하는 기능들이 잘 구현되어 있어서 굳이 클래스 import를 할 필요가 없다.

3. 배열을 사용하여 스택으로 활용

### **직접구현**

```python
class Stack(list):
		push = list.append
		def peek(self):
				return self[-1]  #또는 self[len(self)-1]

s = Stack()
s.push(1)
s.push(5)
s.push(10)
print("my stack is:", s)  # my stack is: [1, 5, 10]
print("popped value is:",s.pop())  # popped value is : 10
print("my stack is:", s)  # my stack is : [1, 5]
print("peeked value is:", s.peek())  # peeked value is: 5
print("my stack is:",s)  # my stack is: [1, 5]
```

- pop은 list의 내장함수로 이미 구현되어 있다

### **List를 스택으로 활용**

```python
s = []
s.append(1)
s.append(5)
s.append(10)
print("my stack is:", s)  # my stack is: [1, 5, 10]
print("popped value is:",s.pop())  # popped value is : 10
print("my stack is:", s)  # my stack is : [1, 5]
print("peeked value is:", s[-1])  # peeked value is: 5
print("my stack is:",s)  # my stack is: [1, 5]
```

- 클래스를 직접 구현한 것과 함수 사용 방법만 다르고 결과는 같다

&nbsp;

## 심화 구현

### 고정 길이 스택 객체 FixedStack 만들기

스택의 기본 구조를 활용하여 push, pop, peek 외 추가 메서드를 구현할 수 있다.

#### 구성요소

- 예외처리 클래스 **Empty:** 비어 있는 스택에 pop 또는 peek를 호출할 때 내보내는 예외 처리. Exception 클래스를 상속한 하위 클래스

- 예외처리 클래스 **Full**: 가득 찬 스택에 push를 호출할 때 내보내는 예외 처리. Exception 클래스를 상속한 하위 클래스

- **init**(capacity):

  FixedStack 객체 생성시의 초기값을 정의하는 함수

  - stk: 스택의 본체인 리스트형 배열. 파라미터 capacity를 통해 받은 값으로 원소 수가 capacity이고 모든 원소가 None인 리스트 생성
  - capacity: 스택의 최대 크기를 나타내는 정수. len(stk)와 값이 일치
  - **ptr:** 스택에 쌓여있는 데이터의 개수를 나타내는 정숫값인 **스택 포인터**

- **len**():

  쌓여있는 데이터의 개수를 반환하는 함수. 여기서는 ptr값을 그대로 반환하도록 한다.

  - 더블언더스코어 __(혹은 던더)를 붙인 함수는 파이썬에 내장된 특수 함수 및 변수를 나타낸다.
  - 클래스에 **len** 함수를 정의하면 클래스형의 인스턴스를 **len** 함수에 전달하도록 한다. 예를 들어 s = FixedStack() 을 통해 FixedStack 클래스 형의 인스턴스 s를 만들었다면, s.**len**() 를 간단히 len(s)라고 표현할 수 있게 된다.

- **is_empty():** 스택이 비어 있는 지 판단

- **is_full():** 스택이 가득 차 있는지 판단

- **push(), pop(), peek():** 기존 역할과 동일. 다만 여기서는 예외처리가 추가되고 반드시 스택 포인터를 사용해서 값을 추가, 반환하도록 한다. (**raise문**은 사용자가 에러를 직접 발생시키도록 해준다)

- **clear():** 스택에 쌓여 있는 데이터를 모두 삭제하여 빈 스택을 만든다. 여기서는 스택 포인터를 0으로 초기화시켜서 기능을 구현했는데, 엄밀히 따지면 stk 리스트에 값은 그대로 있지만 포인터가 0이기 때문에 push, pop, peek 또는 is_empty나 is_full 함수를 사용해도 값의 존재를 확인할 수 없다.

- **find():** 데이터를 검색하는 함수. 스택 포인터를 이용하여 꼭대기에서부터 바닥 쪽으로 선형검색을 진행한다.

- **count():** 스택에 쌓여있는 특정 데이터의 개수를 세는 함수

- **contains**():

  스택에 데이터가 있는지 판단하여 있으면 True를 반환하고 없으면 False를 반환.

  - **len**() 처럼 **contains**() 함수를 정의하면 클래스형의 인스턴스에 **멤버십 판단 연산자**인 in을 사용할 수 있다. 예를 들어 스택 s에 x라는 값이 있는지 확인하고 싶다면 s.__contains(x) 대신 x in s 라고 표현할 수 있다.

- **dump():** 스택에 쌓여있는 ptr개의 모든 데이터를 바닥부터 꼭대기까지 출력                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         ****

```python
class FixedStack:
		
    class Empty(Exception):
        pass

    class Full(Exception):
        pass

    def __init__(self, capacity):
        """원소 수가 capacity고 모든 원소가 None인 상태로 초기화된 스택 stk 생성"""
        self.stk = [None] * capacity  # stk: 스택 본체인 리스트형 배열
        self.capacity = capacity      # capacity: 스택의 크기
        self.ptr = 0                  # ptr: 스택 포인터

    def __len__(self) -> int:
        return self.ptr

    def is_empty(self) -> bool:
        return self.ptr <= 0

    def is_full(self) -> bool:
        return self.ptr >= self.capacity

    def push(self, value):
        if self.is_full():              # 스택이 가득 찬 경우 예외처리
            raise FixedStack.Full
        self.stk[self.ptr] = value      # 포인터를 기준으로 리스트 값을 변경
        self.ptr += 1                   # 값을 추가한 후 포인터를 위로 이동

    def pop(self):
        if self.is_empty():             # 스택이 비어있는 경우 예외처리
             raise FixedStack.Empty
        self.ptr -= 1                   # 포인터를 아래로 이동
        return self.stk[self.ptr]       # 포인터가 가리키고 있던 꼭대기 값 반환

    def peek(self):
        if self.is_empty():             # 스택이 비어 있는 경우 예외처리
            raise FixedStack.Empty
        return self.stk[self.ptr - 1]   # 포인터가 가리키고 있는 꼭대기 값 반환, 포인터 위치는 유지

    def clear(self) -> None:
        """스택을 비움(모든 데이터를 삭제)"""
        self.ptr = 0                    # 포인터 위치 초기화

    def find(self, value):
        for i in range(self.ptr - 1, -1, -1):  # 포인터 기준 꼭대기 쪽부터 선형 검색
            if self.stk[i] == value:
                return i  # 검색 성공
        return -1         # 검색 실패

    def count(self, value):
        """스택에 포함되어있는 value의 개수를 반환"""
        c = 0
        for i in range(self.ptr):  # 바닥 쪽부터 선형 검색
            if self.stk[i] == value:
                c += 1             # 들어 있음
        return c

    def __contains__(self, value):
        """스택에 value가 있는가?"""
        return self.count(value)

    def dump(self) -> None:
        if self.is_empty():  # 스택이 비어 있음
            print('스택이 비어 있습니다.')
        else:
            print(self.stk[:self.ptr])  #바닥부터 포인터 위치(꼭대기)까지 리스트 반환
```

&nbsp;

# 큐 (Queue)

스택과 같이 데이터를 임시 저장하는 자료구조. 영문 명칭은 '일이 처리되기를 기다리는 리스트'라는 뜻을 가진다. 리스트 접근이 양쪽 끝에서 가능한 구조로 **FIFO(선입선출, First-in, First-Out)**이 기본원리

대표적인 내장함수: **put(), peek(), get()**

&nbsp;

## 큐의 구조

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/f3a7046b-fd11-41a8-aafd-09455a1378a0/queue.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/f3a7046b-fd11-41a8-aafd-09455a1378a0/queue.png)

- 가장 먼저온 데이터를 가장 먼저 꺼내는, 차례를 기다리는 선착순 구조.
- 데이터를 꺼내는 쪽을 **프런트(front)**라 하고, 데이터를 넣는 쪽을 **리어(rear)**라고 한다.
- **Put (또는 Enqueue):** 큐의 리어에 데이터를 추가하는 작업
- **Get (또는 Dequeue):** 큐의 프런트에서 데이터를 꺼내는 작업
- **Peek:** 큐의 프런트에 있는 데이터를 확인하는 작업

&nbsp;

## 큐의 활용 예

- 프린터 인쇄 대기열
- 너비 우선 탐색(BFS)

&nbsp;

## 큐의 구현방법

**일반적인 자료구조의 구현방법:**

1. 직접구현
2. 이미 구현된 클래스 import
3. 배열을 활용하여 큐로 구현

### **직접구현**

```python
class Queue(list):
		put = list.append
		
		def peek(self):
				return self[0]
		
		def get(self):
				return self.pop(0)

q = Queue()
q.put(1)
q.put(5)
q.put(10)
print("my queue is:", q)  # my queue is: [1, 5, 10]
print("removed value is:", q.get())  #removed value is: 1
print("my queue is:", q)  # my queue is: [5, 10]
print("peeked value is:", q.peek())  #peeked value is: 5
print("my queue is:", q)  # my queue is: [5, 10]
```

### **구현된** **큐 클래스 import**

```python
from queue import Queue

q = Queue()
q.put(1)
q.put(5)
q.put(10)
print("my queue is:", q)  # my queue is: [1, 5, 10]
print("removed value is:", q.get())  #removed value is: 1
print("my queue is:", q)  # my queue is: [5, 10]
print("peeked value is:", q.peek())  #peeked value is: 5
print("my queue is:", q)  # my queue is: [5, 10]
```

- 직접 구현한 것과 동일하다

### **List를 큐로 활용**

```python
q = []
q.append(1)
q.append(5)
q.append(10)
print("my queue is:", q)  # my queue is: [1, 5, 10]
print("removed value is:", q.pop(0))  #removed value is: 1
print("my queue is:", q)  # my queue is: [5, 10]
print("peeked value is:", q[0])  #peeked value is: 5
print("my queue is:", q)  # my queue is: [5, 10]
```

&nbsp;

## 심화 구현

### 링 버퍼로 큐 구현하기

배열로 큐를 구현할 경우 인큐할 때는 데이터를 리어에 한번 추가해주기만 하면 되기 때문에 처리복잡도가 O(1)이다. 하지만 디큐할 때는 데이터를 꺼내면서 2번째 이후의 모든 원소를 프런트 방향으로 n번 옮겨줘야하기 때문에 처리복잡도가 O(n)이 되면서 프로그램의 효율성이 떨어지게 된다.

**링 버퍼**는 배열 맨 끝의 원소 뒤에 맨 앞의 원소가 연결되는 원형의 자료구조로 디큐할 때 배열 안의 원소를 옮기지 않아도 되기 때문에 인큐와 디큐 모두 O(1)이 되어 훨씬 효율적이다. 이 때 링 버퍼에서의 프런트와 리어는 논리적인 데이터 순서일 뿐 배열의 물리적 순서는 아니다.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/febe1430-a3ef-4046-862b-abdcaf35315d/dequeue.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/febe1430-a3ef-4046-862b-abdcaf35315d/dequeue.png)

링 버퍼를 구현하기 위해서는 배열의 크기가 고정되어야 하기 때문에 앞서 구현한 FixedStack과 비슷한 방식으로 FixedQueue를 구현해주면 된다.

&nbsp;

#### 구성요소

- 예외처리 클래스 **Empty, Full:** 인큐(put), ****디큐(get), peek 함수를 호출할 때 내보내는 예외 처리 클래스

- **init**(capacity):

  FixedQueue 객체 생성시 큐 배열 생성 및 변수들의 초기값을 정의하는 함수

  - que: 큐의 본체인 리스트형 배열. 파라미터 capacity를 통해 받은 값으로 원소 수가 capacity이고 모든 원소가 None인 리스트 생성
  - capacity: 큐의 최대 크기를 나타내는 정수. len(que)와 값이 일치
  - **front, rear:** 맨 앞의 원소, 맨 끝의 원소를 나타내는 인덱스.
  - **no**: 큐에 쌓여있는 데이터 개수를 나타내는 int형 정수. 변수 front와 rear의 값이 같을 경우 큐가 비어있는지 또는 가득 차 있는지 구별하기 위해 필요.

- ***\*len\**():** 큐에 추가되어 있는 데이터의 개수를 반환하는 함수. no값을 그대로 반환하도록 한다.

- **is_empty():** 큐가 비어 있는 지 판단

- **is_full():** 큐가 가득 차 있는지 판단

- **put(), get(), peek():** 기존 역할과 동일. 다만 여기서는 예외처리가 추가되고 반드시 front, rear, no값을 사용해서 배열의 위치를 관리한다.

- **clear():** 스택에 쌓여 있는 데이터를 모두 삭제하여 빈 스택을 만든다. front, rear, no값을 모두 0으로 초기화시켜서 기능을 구현.

- **find():** 데이터를 검색하는 함수. "(i + front) % capacity" 식을 사용하여 주목할 인덱스를 구하고 front부터 시작하여 rear까지 선형 검색을 수행한다.

- **count():** 큐에 있는 특정 데이터의 개수를 세는 함수

- ***\*contains\**():** 큐에 데이터가 있는지 판단하여 있으면 True를 반환하고 없으면 False를 반환.

- **dump():** 큐의 전체 데이터를 front부터 rear까지 순서대로 출력                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         ****

```python
class FixedQueue:

    class Empty(Exception):
        """비어 있는 FixedQueue에 대해 get 또는 peek를 호출할 때 내보내는 예외처리"""
        pass

    class Full(Exception):
        """가득 찬 FixedQueue에 put을 호출할 때 내보내는 예외처리"""
        pass

    def __init__(self, capacity: int) -> None:
        """초기화 선언"""
        self.no = 0     # 현재 데이터 개수
        self.front = 0  # 맨앞 원소 커서
        self.rear = 0   # 맨끝 원소 커서
        self.capacity = capacity      # 큐의 크기
        self.que = [None] * capacity  # 큐의 본체

    def __len__(self) -> int:
        """큐에 있는 모든 데이터 개수를 반환"""
        return self.no

    def is_empty(self) -> bool:
        """큐가 비어 있는지 판단"""
        return self.no <= 0

    def is_full(self) -> bool:
        """큐가 가득 찼는지 판단"""
        return self.no >= self.capacity

    def put(self, x):
        """데이터 x를 인큐"""
        if self.is_full():
            raise FixedQueue.Full  # 큐가 가득 찬 경우 예외처리를 발생
        self.que[self.rear] = x    # rear의 데이터 값 변경(데이터 추가)
        self.rear += 1             # rear의 위치를 한 칸 앞으로 이동
        self.no += 1               # no의 수를 하나 늘림
        if self.rear == self.capacity:
            self.rear = 0

    def get(self):
        """데이터를 디큐"""
        if self.is_empty():
            raise FixedQueue.Empty  # 큐가 비어 있는 경우 예외처리를 발생
        x = self.que[self.front]    # front의 데이터 값 반환
        self.front += 1             # front의 위치를 한 칸 앞으로 이동
        self.no -= 1                # no의 수를 하나 줄임 
        if self.front == self.capacity:
            self.front = 0
        return x

    def peek(self):
        if self.is_empty():
            raise FixedQueue.Empty  # 큐가 비어 있으면 예외처리를 발생
        return self.que[self.front]

    def find(self, value):
        """큐에서 value를 찾아 인덱스를 반환하고 없으면 -1을 반환"""
        for i in range(self.no):  # no의 수 = 큐의 길이, 선형 검색
            idx = (i + self.front) % self.capacity  # 인덱스 계산
            if self.que[idx] == value:  # 검색 성공
                return idx
        return -1  # 검색 실패

    def count(self, value):
        """큐에 포함되어 있는 value의 개수를 반환합니다"""
        c = 0
        for i in range(self.no):  # 큐 데이터를 선형 검색
            idx = (i + self.front) % self.capacity
            if self.que[idx] == value:  # 검색 성공
                c += 1  # 들어있음
        return c

    def __contains__(self, value):
        """큐에 value가 포함되어 있는지 판단합니다"""
        return self.count(value)

    def clear(self) -> None:
        """큐의 모든 데이터를 비웁니다"""
        self.no = self.front = self.rear = 0

    def dump(self) -> None:
        """모든 데이터를 맨 앞에서 맨 끝 순서로 출력합니다"""
        if self.is_empty():  # 큐가 비어 있으면 예외처리를 발생
            print('큐가 비어 있습니다.')
        else:
            for i in range(self.no):
                print(self.que[(i + self.front) % self.capacity], end=' ')
            print()
```