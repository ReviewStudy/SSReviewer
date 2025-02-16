통상적으로 파이썬은 인터프리터 언어라고 한다.

[[인터프리터 언어 VS 컴파일 언어]]
근데 정말일까?

#### 파이썬은 어떻게 실행되는가

![[Pasted image 20250216173017.png]]([Internal working of Python. Introduction | by KAUSHIK K 1941116 | Medium - https://medium.com/@kaushik.k/internal-working-of-python-415572929e7a](https://medium.com/@kaushik.k/internal-working-of-python-415572929e7a))


1. **소스 코드 작성**
```python
print("Hello World)
```

2. **컴파일 (Byte code 변환)**
- Python 인터프리터는 소스 코드를 Byte Code(.pyc 파일)로 변환을 수행
	- 변환된 바이트 코드는 저장되거나 메모리로 남아 있다가 실행 (__pycache__/의 .pyc 파일들)
- 이때 Byte Code는 CPU가 직접 실행하는 기계어가 아닌, 가상 머신 (PVM, Python Virtual Machine)이 이해할 수 있는 중간 코드

```python
import dis
dis.dis("print('Hello, World!')")
```

3. **인터프리터가 바이트코드 실행**
	1. 변환된 바이트코드는 Python 가상 머신 (PVM, Python Virtual Machine)이 실행
	2. PVM은 한 줄씩 바이트코드를 읽고 실행 -> Python이 Interprete 언어로 불리는 이유



> [!QUESTION] 파이썬을 인터프리트 언어라 할 수 있나요?
> 파이썬도 사실은 컴파일러를 통해 Byte Code를 생성한다.
> 엄밀하게 한 줄 씩 읽는 것은 파이썬 코드 한 줄이 아닌, Byte Code의 한 줄이다.
> 컴파일 VS 인터프리트의 차이는 언어 자체의 특성이 아닌, 구현 형태의 차이일 뿐이다.
> 파이썬도 컴파일 방식으로 실행시킬 수 있다 (PyPy)



#### Python Interpreter의 종류
Python 인터프리터는 여러 가지 종류가 있으며, 사용 목적에 따라 다릅니다.

1. **CPython (가장 널리 사용됨)**
    - C 언어로 작성된 표준 Python 구현체.
    - 바이트코드로 변환 후 **PVM (Python Virtual Machine)**이 실행.
    - `python` 명령어로 실행되는 **기본 인터프리터**.
    - 흔히들 파이썬을 설치한다고 표현하는 것은 사실은 CPython을 설치한다는 뜻
2. **PyPy (JIT 컴파일 지원)**
    - Just-In-Time (JIT) 컴파일러를 사용하여 실행 속도 향상.
    - 실행 도중에 일부 코드가 기계어로 변환되어 빠르게 실행.
3. **Jython (Java 기반)**
    - Python 코드를 **Java 바이트코드**로 변환하여 JVM에서 실행.
4. **IronPython (.NET 기반)**
    - Python 코드를 **.NET CLR (Common Language Runtime)**에서 실행.
5. **MicroPython (임베디드용)**
    - 마이크로컨트롤러에서 실행하기 위한 가벼운 Python 인터프리터.

### 그럼 파이썬은 왜 인터프리트 방식을 채택했을까?
1. **파이썬의 주요 철학 중 하나인 Dynamic과 유연성을 위해**
	1. Dynamic Typing 등으로 인해 코드 실행 중 변수의 타입이나 함수나 객체가 변경될 수 있음
	2. 따라서 미리 컴파일을 수행하기 어려움
2. **컴파일 시간 절약을 통한 빠른 개발 가능**
3. **플랫폼 독립성 (Platform Independent) 보장 가능**
	1. 바이트 코드와 PVM을 통해 어떤 운영체제에서든 동일한 코드 실행 가능



### Python도 통상적인 컴파일 언어처럼 실행할 수는 없을까?
가능하다!

**(1) PyPy (JIT 컴파일)** [[JIT 컴파일러]]
- **JIT(Just-In-Time) 컴파일러**를 사용해 속도를 최적화하는 방식.
- 실행 중 자주 사용되는 코드를 기계어로 변환하여 성능을 향상시킴.
- 근데 그러면 Dynamic Typing과 같은 기능은 어떻게 해결?
	- **타입 추론(Type Specialization):** 실행 중 변수 타입을 감지하여 최적화된 코드 생성
	- **가드(Guards):** 타입 변경이 감지되면 새로운 최적화 코드 실행 또는 인터프리터 모드로 전환
	- **Trace-based JIT:** 반복적으로 실행되는 코드를 감지하여 JIT 컴파일
	- **객체 구조 최적화:** Map-Dictionary 기법을 활용하여 속성 접근 속도 향상

**(2) Cython (C로 변환)**
- 파이썬 코드를 **C로 변환하여 컴파일**하는 방식.
- 속도가 훨씬 빨라지지만, 일부 동적 기능(예: exec())은 제한됨.

**(3) Nuitka**
- 파이썬 코드를 **완전히 기계어로 컴파일**하는 프로젝트.
- 하지만 완벽한 최적화는 어려움.


### 여담

```cardlink
url: https://www.cio.com/article/3824874/c%EC%96%B8%EC%96%B4%EC%99%80-%EC%86%8D%EB%8F%84-%EA%B2%A9%EC%B0%A8-%EC%A2%81%ED%9E%8C%EB%8B%A4%C2%B7%C2%B7%C2%B7-%ED%8C%8C%EC%9D%B4%EC%8D%AC-%EC%83%88-%EB%B2%84%EC%A0%84-%EC%9D%B8%ED%84%B0%ED%94%84.html
title: "C언어와 속도 격차 좁힌다··· 파이썬 새 버전, 인터프리터 개선해 30% 성능↑"
description: "올해 말 출시 예정인 파이썬 3.14에 새로운 인터프리터가 추가되면서, 기존 코드 수정 없이도 최대 30% 성능 향상이 가능해진다. 첫 번째 베타 버전은 2025년 5월 공개될 예정이다."
host: www.cio.com
favicon: https://www.cio.com/wp-content/uploads/2023/02/cropped-CIO-favicon-2023.png?w=32
image: https://www.cio.com/wp-content/uploads/2025/02/3824874-0-41131300-1739518737-shutterstock_2376207999.jpg?quality=50&strip=all&w=1024
```

- Python 3.14 버전에 tail calls 기반의 새로운 인터프리터 탑재
- C 컴파일러가 CPython 코드에서 수행하는 최적화 기법으로, 인터프리터가 바이트코드를 실행하는 과정을 최적화해 전체 실행 속도를 높임


```cardlink
url: https://tech.ssut.me/peephole-how-python-optimizes-bytecode/
title: "Peephole: CPython은 어떻게 코드를 최적화하는가"
description: "많은 스크립트 언어는 \"실행 성능(속도)\"이 좋지 않다는 큰 단점을 지니고 있습니다. 이를 해결하기 위해 JIT 런타임을 붙이거나 개발자 스스로코드를 최적화하기도 하지만 개발자 스스로 코드를 최적화한다고 하여(좋은 코드를 작성했다고 했을 때) 투자하는 시간에 비해 큰 성능향상을 얻기는어렵습니다. Python은 이러한 문제를 조금이나마 해소시키고자 Peephole이라는 Python 바이트코드 최적화를"
host: tech.ssut.me
image: https://tech.ssut.me/content/images/2017/08/2.7-vs-3.6.png
```
- CPython에서 바이트코드를 최적화할 수 있다. -> Peephole Optimization
- 간단한 예시
	- Numeric Calculation
		- 24 * 60 -> 미리 계산해두고 저장해서 사용

 
  
