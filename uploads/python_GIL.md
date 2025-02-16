파이썬의 **GIL (Global Interpreter Lock)**이란?
| CPython 인터프리터**에서 **동시에 하나의 스레드만 실행되도록 강제하는 Lock
| 멀티스레드를 사용하더라도 하나의 스레드만 실행되며, 나머지 스레드는 대기하는 방식으로 동작

> GIL은 하나의 프로세스 내에서 파이썬 인터프리터가 한 시점에 단 하나의 쓰레드에 의해서만 실행**될 수 있도록 보장하는 일종의 뮤텍스(mutex)이며, 이는 CPython의 메모리 관리가 thread-safe 하지 않기 때문에 경쟁 상태(race condition)를 방지하기 위해서 도입

위 문장을 이해하기 위해서는 두가지 질문을 해결해야 한다.
1. CPython 인터프리터가 뭐지? [[파이썬이 실행되는 과정]]
2. 왜 인터프리터를 동시에 하나의 스레드만 실행하도록 강제하지?





### Python의 Garbage Collection - Reference Counting
파이썬은 크게 두 가지 방식으로 Garbage Collection이 동작
- **Reference Counting (참조 카운트)** ✅
- **Cycle Detection (순환 참조 탐지, Generational GC)**

**Reference Counting 동작 방식**
- Python의 모든 객체는 **참조 카운트(Reference Count)를 가짐**.
- 객체가 **새로운 변수에 할당되면 참조 카운트가 증가**.
- 변수가 소멸하거나 `del`을 호출하면 **참조 카운트가 감소**.
- **참조 카운트가 0이 되면 메모리에서 자동 해제(Garbage Collection).**

![[Pasted image 20250216235251.png]]
[Confessions of a Code Addict | Abhinav Upadhyay | Substack](https://blog.codingconfessions.com/p/cpython-reference-counting-internals)


**Race Condition이 발생할 수 있음**
Reference Counting을 기록하는 ref_cnt도 일종의 변수
=> critical section이 될 수 있음
하지만 Reference Counting 방식은 Multi-thread 방식에서 violation이 일어날 수 있음


```
(A += 1)
mov eax, num # LOAD
inc eax      # INC    
mov num, eax # STORE
```

![[Pasted image 20250217000034.png]]
([Global Interpreter Lock GIL 자세히 톺아보기, 한성민 - PyCon Korea 2022](https://www.youtube.com/watch?v=hj8BnSAalEs))

따라서 여러 스레드가 동시에 접근하면 데이터 손실이 발생할 수 있음
이 상황을 어떻게 해결할 수 있지?

=> [[Mutex Lock]]
=> 하지만 모든 객체의 연산에 Mutex Lock을 거는 것은 성능이 크게 저하될 수 있음
=> 따라서 Python은 GIL, 즉 인터프리터에 락을 걸어 Reference Counting을 보호



#### CPU Bound VS IO-Bound

``` python
import threading
import time

def compute():
    for _ in range(10**7):
        pass  # 단순 연산

t1 = threading.Thread(target=compute)
t2 = threading.Thread(target=compute)

start = time.time()
t1.start()
t2.start()
t1.join()
t2.join()
end = time.time()

print("멀티스레딩 실행 시간:", end - start)  # GIL로 인해 단일 스레드와 차이 없음

```

```python
import threading
import requests
import time

urls = ["https://www.example.com"] * 10

def fetch(url):
    requests.get(url)

threads = [threading.Thread(target=fetch, args=(url,)) for url in urls]

start = time.time()
for thread in threads:
    thread.start()
for thread in threads:
    thread.join()
end = time.time()

print("멀티스레딩 실행 시간:", end - start)  # 성능 향상 있음

```


#### 어떻게 해결할 수 있을까?
- IO-Bound 

#### 여담
파이썬은 왜 이렇게 Lock을 설계했을까?
- 그냥 인터프리터 전체가 아닌 레퍼런스 카운팅 할 때만 락 걸어도 되는거 아니야?
	- Python을 개발할 당시 (1990년 즈음) 주로 싱글 코어 개발이었음
	- 따라서 스레드에 락을 거는 방식이 일반적임
	- 따라서 해당 디자인이 부채가 되어 지금 지경에 이름

