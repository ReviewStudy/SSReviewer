# Python Dictionary의 Key 순서 보장

#Python #Dictionary #HashTable #CompactDict #CollisionResolution

Python 3.6부터 Dictionary의 Key 순서가 보장됩니다. 이전 버전에서는 순서 보장을 위해 OrderedDict를 사용해야 했습니다.

## Python Dictionary의 동작 원리

Python Dictionary는 내부적으로 해시 테이블을 사용하는 자료구조입니다. 해시 함수는 키를 해시 값으로 변환하고, 이를 인덱스로 변환하여 특정 메모리 위치에 저장합니다. 이를 통해 해시 키를 사용하여 O(1) 시간 복잡도로 값에 접근할 수 있습니다.

### 해시 충돌 해결

해시 테이블에서 서로 다른 키가 같은 해시 값을 가질 수 있는데, 이를 해시 충돌이라고 합니다. Python은 충돌이 발생하면 Open Addressing 기법을 사용하여 다른 빈 슬롯을 찾아 값을 저장합니다.

### Dictionary 최적화

Python은 Dictionary의 성능을 최적화하기 위해 여러 기법을 도입했습니다. 특히 Compact Dict 구조는 키의 삽입 순서를 보장합니다.

#### Resizing

- 해시 테이블의 로드 팩터가 일정 수준을 초과하면 크기를 확장합니다.
- 일반적으로 해시 테이블이 66% 이상 차면 크기를 두 배로 늘리고, 새로운 크기는 소수에 가깝게 설정합니다.
- 새로운 배열을 할당하고 기존 데이터의 모든 키를 재해싱합니다.

#### Compact Dict

- 기존에는 해시 테이블과 별도의 Key-Value 저장 공간이 있었습니다.
- Compact Dict는 연결된 리스트를 사용하여 키와 값을 순차적으로 저장합니다.
- 메모리 사용량이 감소하고, 키의 삽입 순서가 유지됩니다.

![Compact Dict 구조](https://i.imgur.com/PverM1B.png)

![Compact Dict 메모리 절약](https://i.imgur.com/KgH3ryt.png)

## 핵심 포인트

- Python 3.6부터 Dictionary의 Key 순서가 보장됩니다.
- 해시 테이블은 해시 충돌을 Open Addressing으로 해결합니다.
- Compact Dict는 메모리 사용량을 줄이고 키의 삽입 순서를 유지합니다.

## 토론 및 질문

1. Python 3.6 이전 버전에서 OrderedDict를 사용해야 했던 이유는 무엇인가요?
2. 해시 충돌을 해결하는 다른 방법에는 어떤 것들이 있을까요?
3. Compact Dict가 메모리 사용량을 줄이는 방법은 무엇인가요?
4. 해시 테이블의 크기를 소수로 설정하는 이유는 무엇인가요?
5. Python Dictionary의 성능을 최적화할 수 있는 다른 방법은 무엇일까요?