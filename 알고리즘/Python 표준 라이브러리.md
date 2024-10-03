

아래는 각 모듈에 대한 간단한 예제들입니다.

---

### **2. 데이터 구조 및 관련 모듈**

#### **`collections`**: 고급 데이터 구조 (예: `deque`, `Counter`, `OrderedDict` 등)

```python
from collections import Counter

data = ['apple', 'banana', 'apple', 'orange', 'banana', 'banana']
counter = Counter(data)
print(counter)
```

#### 결과:
```
Counter({'banana': 3, 'apple': 2, 'orange': 1})
```

네, **`collections`** 모듈에는 **`Counter`** 외에도 다양한 데이터 구조들이 있습니다. 아래는 그 각각의 간단한 예제들입니다.

---

### 1. **`deque`** (Double-Ended Queue)
`deque`는 양방향 큐로, 양 끝에서 요소를 빠르게 추가하고 제거할 수 있습니다.

```python
from collections import deque

d = deque([1, 2, 3])
d.append(4)  # 오른쪽에 추가
d.appendleft(0)  # 왼쪽에 추가
print(d)
d.pop()  # 오른쪽에서 제거
d.popleft()  # 왼쪽에서 제거
print(d)
```

#### 결과:
```
deque([0, 1, 2, 3, 4])
deque([1, 2, 3])
```

---

### 2. **`OrderedDict`** (순서가 유지되는 딕셔너리)
`OrderedDict`는 입력 순서를 유지하는 딕셔너리입니다. Python 3.7부터는 일반 딕셔너리도 입력 순서를 유지하지만, `OrderedDict`는 더 오래된 Python 버전에서도 사용됩니다.

```python
from collections import OrderedDict

od = OrderedDict()
od['apple'] = 1
od['banana'] = 2
od['cherry'] = 3
print(od)
```

#### 결과:
```
OrderedDict([('apple', 1), ('banana', 2), ('cherry', 3)])
```

---

### 3. **`defaultdict`** (기본값이 있는 딕셔너리)
`defaultdict`는 키가 존재하지 않을 때 기본값을 자동으로 생성하는 딕셔너리입니다.

```python
from collections import defaultdict

dd = defaultdict(int)  # 기본값은 0
dd['apple'] += 1
dd['banana'] += 2
print(dd)
```

#### 결과:
```
defaultdict(<class 'int'>, {'apple': 1, 'banana': 2})
```

---

### 4. **`ChainMap`** (여러 딕셔너리를 하나로 묶어 관리)
`ChainMap`은 여러 딕셔너리를 하나의 뷰로 묶어서 다룰 수 있는 데이터 구조입니다.

```python
from collections import ChainMap

dict1 = {'a': 1, 'b': 2}
dict2 = {'b': 3, 'c': 4}
chain = ChainMap(dict1, dict2)
print(chain['b'])  # dict1의 값이 우선
print(chain['c'])  # dict2의 값
```

#### 결과:
```
2
4
```

---

### 5. **`UserDict`** (사용자 정의 딕셔너리)
`UserDict`는 `dict`를 상속하여 딕셔너리를 커스터마이징할 수 있는 클래스입니다.

```python
from collections import UserDict

class MyDict(UserDict):
    def __setitem__(self, key, value):
        print(f"Setting {key} to {value}")
        super().__setitem__(key, value)

my_dict = MyDict()
my_dict['apple'] = 5
```

#### 결과:
```
Setting apple to 5
```

---

### 6. **`UserList`** (사용자 정의 리스트)
`UserList`는 `list`를 상속하여 리스트를 커스터마이징할 수 있는 클래스입니다.

```python
from collections import UserList

class MyList(UserList):
    def append(self, item):
        print(f"Appending {item}")
        super().append(item)

my_list = MyList([1, 2, 3

---

#### **`heapq`**: 힙(우선순위 큐) 알고리즘 구현

```python
import heapq

numbers = [5, 7, 9, 1, 3]
heapq.heapify(numbers)  # 리스트를 힙 구조로 변환
heapq.heappush(numbers, 4)  # 힙에 원소 추가
print(numbers)
```

#### 결과:
```
[1, 3, 4, 7, 5, 9]
```

---

#### **`bisect`**: 이진 검색 및 리스트 삽입 알고리즘

```python
import bisect

numbers = [1, 3, 4, 6, 8]
bisect.insort(numbers, 5)  # 정렬된 상태를 유지하면서 5를 삽입
print(numbers)
```

#### 결과:
```
[1, 3, 4, 5, 6, 8]
```

---

#### **`queue`**: 동기화 큐 클래스 제공

```python
from queue import Queue

q = Queue()
q.put(10)
q.put(20)
print(q.get())  # 10
print(q.get())  # 20
```

#### 결과:
```
10
20
```

---

### **3. 수학 및 통계 관련 모듈**

#### **`math`**: 수학 함수 (예: 삼각 함수, 로그 등)

```python
import math

print(math.sqrt(16))  # 16의 제곱근
```

#### 결과:
```
4.0
```

---

#### **`statistics`**: 기본적인 통계 함수 (평균, 분산 등)

```python
import statistics

data = [1, 2, 3, 4, 5]
print(statistics.mean(data))  # 평균 계산
```

#### 결과:
```
3
```

---

#### **`itertools`**: 반복 가능한 데이터 구조를 위한 함수들

```python
import itertools

data = [1, 2, 3]
perms = list(itertools.permutations(data))
print(perms)
```

#### 결과:
```
[(1, 2, 3), (1, 3, 2), (2, 1, 3), (2, 3, 1), (3, 1, 2), (3, 2, 1)]
```

---

### **4. 날짜 및 시간 관련 모듈**

#### **`datetime`**: 날짜 및 시간 처리 도구

```python
from datetime import datetime

now = datetime.now()
print(now)
```

#### 결과:
```
2024-10-01 12:34:56.123456  # 예시 값, 실행 시점에 따라 달라짐
```

---

#### **`time`**: 시간 관련 함수 제공 (시간 측정 및 변환)

```python
import time

start = time.time()
# 실행할 코드
time.sleep(2)  # 2초 대기
end = time.time()
print(f"Elapsed time: {end - start} seconds")
```

#### 결과:
```
Elapsed time: 2.0 seconds
```

--- 

이 예제들은 각 모듈의 대표적인 기능을 간단히 보여주는 코드들입니다.