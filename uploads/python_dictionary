> Python Dictionary에서 Key 값은 순서 보장이 될까?


아래 문제를 풀어봅시다.
1) True
2) False

중에 정답은 무엇일까요?
``` python

example_dict = {
'a': 1,
'b': 2,
'c': 3
}

if example_dict.keys() == ['a','b','c']:
	print('True')
else:
	print('False')
```




정답은 Python 3.6+ 기준으로 True
그 이전까지는 순서가 보장이 안됐다. 보장하려면 OrderedDict를 사용해야 했다.
왜 그랬을까?
겉보기에는 당연한 이야기지만, 재밌는 내용들이 있다.


### 파이썬 Dictionary는 어떻게 동작할까?
Python Dictionary는 내부적으로 Hash를 활용하는 자료구조입니다.
Hash의 정의는 아래와 같습니다.
- TBA

따라서 Python Dictionary는 다음과 같아요.
- Hash table을 기반으로 동작하는 자료구조.
- Hash Function을 활용하여 Key를 Hash Value로 변환 한 후, 이를 Index로 변환하여 특정한 메모리 위치에 저장.
- 따라서 Hash Key를 통해 Value에 O(1)만에 접근 가능
[분리 체이닝을 사용한 파이썬 해시 테이블 구현 - GeeksforGeeks - https://www.geeksforgeeks.org/implementation-of-hash-table-in-python-using-separate-chaining/](https://www.geeksforgeeks.org/implementation-of-hash-table-in-python-using-separate-chaining/)


#### Collision Resolution
- 해시에서 중요한 개념 중 하나
- 해시 테이블에서 서로 다른 키가 같은 해시 값을 가질 수 있음. -> 이를 해시 충돌 (Hash Collision)이라고 부름
- Python에서는 이를 어떻게 해결했을까?
- Open Addressing
	- 충돌이 발생하면, 다른 빈 슬롯을 찾아 값을 저장한다.
	- 파이썬은 Probing(탐색) 기법을 이용하여 충돌이 발생하였을 때 적절한 슬롯을 찾으러 감

[[Collision Resolution Strategies]]


#### Dictionary Optimization
> 이외에도 Python에서는 Dictionary 동작을 최적화하기 위해 아래와 같은 기법도 도입했습니다. 그리고 여기서 4번째로 알아볼 Compact Dict에 순서가 보장되는 이유가 숨어 있다.
1. Resizing
2. Compact Dict


```


[0] * 100000
for


1000
list.apend(0)

```

**Rezising**
- 파이썬의 해시 테이블은 **로드 팩터(Load Factor, 사용률)** 가 일정 수준을 초과하면 크기를 확장
- 로드 팩터 = 사용된 슬롯 개수 / 총 슬롯 개수
- 파이썬에서는 일반적으로 **해시 테이블이 2/3(약 66%) 이상 차면 크기를 증가**
- 크기 증가 시, **기존 크기의 약 2배로 확장** 되며 새로운 크기는 항상 **소수(prime number)에 가깝게 설정**
	- Why?
- 과정
	- 새로운 크기의 **배열(해시 테이블 슬롯)을 할당**.
	- - 기존 데이터의 **모든 키를 새로운 슬롯에 재해싱(Rehashing)**.
		-  `index = hash(key) % table_size`
		- 테이블 사이즈가 바뀌면 인덱스가 달라질 수 있음
	- 1. 기존 테이블을 제거하고, 새로운 테이블을 사용.


> [!NOTE] Title
> 리사이징을 어떻게 최적화할 수 있을까?


**Compact Dict**
- 기존
	- 해시 테이블 자체와 별도의 **Key-Value 저장 공간** 이 있었음.
	- 키를 저장하는 테이블과 값이 따로 존재하여 **추가적인 포인터(Reference) 비용** 발생.
- Compact Dict 구조 (파이썬 3.6+)
	- **연결된 리스트(Ordered Array)를 사용** 하여 키와 값을 **순차적으로 저장**.
	- **메모리 사용량 감소** (파이썬 3.6 이후 딕셔너리의 메모리 사용량이 약 20% 감소).
	- 키의 **삽입 순서가 유지됨** (파이썬 3.6+ 부터).

![](https://i.imgur.com/PverM1B.png)


![](https://i.imgur.com/KgH3ryt.png)

- 기존에는 하나의 해시 테이블에 (index, hash, key, value)를 한번에 관리
- compact dict는 indices라는 해시 테이블을 통해 key 값에 해당하는 위치를 저장
- 메모리 사용량을 절약할 수 있음
	- 예를 들어 해시 테이블의 크기 dk_size를 8이라고 가정하면
	- 기존: 8 (dk_size) * 8 * 3 (hash, key, value) = 192
	- 개선: 8 (dk_size) * 1 (char) + (8 * 3) * 3 (현재 Item 개수) = 80
	- dk_size가 커짐에 따라 값은 더욱 차이 나게 될 것





