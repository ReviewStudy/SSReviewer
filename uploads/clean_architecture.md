클린 아키텍처(Clean Architecture)는 **로버트 C. 마틴(Uncle Bob)**이 제안한 소프트웨어 아키텍처 원칙


![](https://i.imgur.com/jC4AMZJ.png)


- **엔티티(Entity)**, **도메인**
    - 도메인: 애플리케이션이 해결하고자 하는 특정한 주제나 분야를 가리키며, 해당 분야에 적용되는 개념, 규칙, 데이터, 프로세스 등을 포함
    - 도메인 모델: 해당 비즈니스 영역에서 도메인의 핵심 개념, 엔티티, 도메인 간의 관계, 도메인이 지켜야 하는 규칙 및 도메인의 상태를 나타냄
    - 도메인 계층: 소프트웨어 시스템 내에서 핵심 비즈니스 로직과 엔터프라이즈의 핵심 도메인 관련 기능을 관리하고 구현하는 부분, 도메인이 가지는 비즈니스 로직은 시스템의 핵심 목적을 달성하기 위한 연산, 규칙, 데이터 처리 및 동작을 정의
    - 데이터베이스, 프레임워크, UI 등에 의존하지 않고 독립적이어야 한다.
	- 예: 도메인 모델, 애그리게이트(Aggregate), 엔티티(Entity), 값 객체(Value Object) 등.
- **유스케이스(Use Case)**, **애플리케이션**
    - 애플리케이션의 핵심 동작(비즈니스 규칙의 흐름)을 담당하는 계층이다.
    - 엔티티를 활용하여 특정 작업을 수행하는 비즈니스 로직을 포함한다.
    - 이 계층은 엔티티와 인터페이스를 통해 외부 계층과만 상호작용한다.
    - 예: 서비스(Service), 애플리케이션 로직, 인터랙터(Interactor).
- **인터페이스 어댑터(Interface Adapter)**, **인터페이스**
    - 유스케이스 계층과 외부 시스템(DB, 웹, API 등)을 연결하는 역할을 한다.
    - 유스케이스에서 필요한 데이터를 가져오거나, 유스케이스가 반환하는 데이터를 변환하는 작업을 한다.
    - 예: 리포지토리(Repository), 컨트롤러(Controller), 프레젠터(Presenter), 뷰 모델(View Model).
- **프레임워크 & 드라이버(Frameworks & Drivers)**, **인프라**
    - 가장 바깥 계층으로, 특정 기술 스택(웹 프레임워크, DB, UI 등)을 포함한다.
    - 이 계층은 애플리케이션의 핵심 로직과 분리되어 있어야 한다.
    - 예: Django, Spring, FastAPI, PostgreSQL, React, Android 등.


**의존성 역전 원칙(Dependency Inversion Principle)  
- 고수준 모듈은 저수준 모듈에 의존해서는 안되며, 양쪽 모듈 모두 추상화에 의존해야 합니다. 이를 통해 느슨한 결합을 유지할 수 있습니다.
	- **안쪽 계층(엔티티 → 유스케이스 → 인터페이스 어댑터 → 프레임워크 & 드라이버)으로만 의존 가능**해야 한다.

**경계(Boundary)의 분리  
- 시스템을 여러 영역으로 나누고, 각 영역 사이의 인터페이스를 정의하여 각 영역의 독립성을 보장합니다.

**인터페이스 분리 원칙(Interface Segregation Principle)  
- 클라이언트가 자신이 사용하지 않는 메서드에 의존하지 않아야 합니다. 즉, 인터페이스는 클라이언트의 요구에 딱 맞는 형태로 분리되어야 합니다.



**직접 FastAPI로 구현해보자!**
domain, application, infra, interface로 계층을 나누어 구현해보자.
![](https://i.imgur.com/2yjEqwF.png)


회원을 관리하는 User 리소스를 구현해보자.
- User 추가, 삭제, 조회 등의 기능을 지원해야 한다.
1. domain 계층
>  소프트웨어 시스템 내에서 핵심 비즈니스 로직과 엔터프라이즈의 핵심 도메인 관련 기능을 관리하고 구현하는 부분

```python
from dataclasses import dataclass
from datetime import datetime


@dataclass
class Profile:
    # id 없이 데이터만 가지고 있는 도메인 객체를 Value Object(VO)라고 한다.
    name: str
    email: str

@dataclass
class User:
    id: str
    profile: Profile
    password: str
    created_at: datetime
    updated_at: datetime
```

VO에 대해 더욱 자세히 알아보려면?

```cardlink
url: https://product.kyobobook.co.kr/detail/S000001810495
title: "도메인 주도 개발 시작하기: DDD 핵심 개념 정리부터 구현까지 | 최범균 - 교보문고"
description: "도메인 주도 개발 시작하기: DDD 핵심 개념 정리부터 구현까지 | 가장 쉽게 배우는 도메인 주도 설계 입문서!이 책은 도메인 주도 설계(DDD)를 처음 배우는 개발자를 위한 책이다. 실제 업무에 DDD를 적용할 수 있도록 기본적인 ……"
host: product.kyobobook.co.kr
favicon: https://contents.kyobobook.co.kr/resources/fo/images/common/ink/favicon/favicon_256x256.png
image: https://contents.kyobobook.co.kr/sih/fit-in/458x0/pdt/9791162245385.jpg
```



2. application 계층
> 애플리케이션의 핵심 동작(비즈니스 규칙의 흐름)을 담당하는 계층이다.
 엔티티를 활용하여 특정 작업을 수행하는 비즈니스 로직을 포함한다.

``` python
class UserService:
    def __init__(self):
        self.user_repo: IUserRepository = UserRepository() # DIP 위반임 ->  수정 필요
        self.ulid = ULID() # 정렬 가능한 범용 고유 식별자 ( Universally Unique Lexicographically Sortable Identifier ) -> 정렬이 가능하므로, 검색 속도 향상 가능
        self.crypto = Crypto()

    def create_user(self, name: str, email: str, password: str):
        _user = None

        try:
            _user = self.user_repo.find_by_email(email)
        except Exception as e:
            if e.status_code != 422:
                raise e
        
        if _user:
            raise HTTPException(status_code=422, detail="Email already exists")
        
        now = datetime.now()
        user: User = User(
            id=self.ulid.generate(),
            name=name,
            email=email,
            password=self.crypto.encrypt(password),
            created_at=now,
            updated_at=now
        )
        self.user_repo.save(user)
        return user
```

[[ULID]]는 또 뭐지?