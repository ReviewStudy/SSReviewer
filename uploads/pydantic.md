> FastAPI에서 Pydantic을 사용하면 어떤 점이 좋을까?


**주요 장점**
1. 자동 데이터 검증 (데이터가 올바른 형식인지 자동으로 확인하고, 잘못된 데이터에 대해서는 명확한 에러 메시지를 제공)
2. 자동 문서화
3. 타입 안정성
4. JSON 직렬화/역직렬화
5. 성능 (매우 빠른 데이터 검증과 모델 처리를 제공)
6. 데이터 변환 (Example. 문자열 -> 날짜)


```python
class CreatedUserBody(BaseModel):
    name: str = Field(min_length=2, max_length=32)
    email: EmailStr = Field(max_length=64)
    password: str = Field(min_length=8, max_length=32)

class UpdateUser(BaseModel):
    name: str | None = Field(min_length=2, max_length=32, default=None)
    password: str | None = Field(min_length=8, max_length=32, default=None)

class UserResponse(BaseModel):
    id: str
    name: str
    email: EmailStr
    created_at: datetime
    updated_at: datetime

class GetUsersResponse(BaseModel):
    total_count: int
    page: int
    users: list[UserResponse]
```

```python
@router.get("")

@inject

def get_users(
    page: int = 1,
    items_per_page: int = 10,
    user_service: UserService = Depends(Provide[Container.user_service]),
) -> GetUsersResponse:

    total, users = user_service.get_users(page, items_per_page)
    return {"total_count": total, "page": page, "users": users} # type: ignore
```

get_users_response
```json
{
    "total_count": 1,
    "page": 1,
    "users": [
        {
            "id": "01JNV0NGX1Z5346W915MPH34KZ",
            "name": "UpdateTestUser",
            "email": "test@example.com",
            "created_at": "2025-03-08T22:56:12",
            "updated_at": "2025-03-09T22:52:55"
        }
    ]
}
```


원래 user domain
```python
@dataclass
class User:
    id: str
    name: str
    email: str
    password: str
    created_at: datetime
    updated_at: datetime
    memo: str | None


```


**Pydantic은 아주 느리다. 불필요한 곳에서 가급적 사용하지 말자.**

```cardlink
url: https://hyperconnect.github.io/2023/05/30/Python-Performance-Tips.html#5-pydantic%EC%9D%80-%EC%95%84%EC%A3%BC-%EB%8A%90%EB%A6%AC%EB%8B%A4-%EB%B6%88%ED%95%84%EC%9A%94%ED%95%9C-%EA%B3%B3%EC%97%90%EC%84%9C-%EA%B0%80%EA%B8%89%EC%A0%81-%EC%82%AC%EC%9A%A9%ED%95%98%EC%A7%80-%EB%A7%90%EC%9E%90
title: "고성능 ML 백엔드를 위한 10가지 Python 성능 최적화 팁"
description: "다량의 데이터를 사용하는 ML 워크로드에 특화된 최적화 기법들과, ML 백엔드에서 자주 사용되는 third-party 라이브러리를 효과적으로 사용하는 방법들을 실제 하이퍼커넥트의 사례와 함께 공유합니다."
host: hyperconnect.github.io
favicon: https://hyperconnect.github.io/assets/favicon.svg
image: https://hyperconnect.github.io/assets/2023-05-30-Python-Performance-Tips/cover.png
```


```python
import timeit
from typing import List

from pydantic import BaseModel

class FeatureSet(BaseModel):
    user_id: int
    features: List[float]

def create_pydantic_instances() -> None:
    for i in range(400):
        obj = FeatureSet(
            user_id=i,
            features=[1.0 * i + j for j in range(50)],
        )

elapsed_time = timeit.timeit(create_pydantic_instances, number=1)
print(f"pydantic: {elapsed_time * 1000:.2f}ms")
```
=> 
12.29ms

```python
import timeit
from typing import List

class FeatureSet:
    def __init__(self, user_id: int, features: List[float]) -> None:
        self.user_id = user_id
        self.features = features
    
def create_class_instances() -> None:
    for i in range(400):
        obj = FeatureSet(
            user_id=i,
            features=[1.0 * i + j for j in range(50)],
        )

elapsed_time = timeit.timeit(create_class_instances, number=1)
print(f"class: {elapsed_time * 1000:.2f}ms")
```
=>
class: 1.54ms

**pydantic을 사용했을 때 10배정도 느리다**




**여담**

```cardlink
url: https://github.com/pydantic/pydantic/pull/7288
title: "Fix pydantic-settings to underscore in docs by FacerAin · Pull Request #7288 · pydantic/pydantic"
description: "Change SummaryIn this migration docs, it seems desirable to change &quot;pydantic-settings&quot; to &quot;pydantic_settings&quot; with underscores.Even though the package repo name is &quot;pydan..."
host: github.com
favicon: https://github.githubassets.com/favicons/favicon.svg
image: https://opengraph.githubassets.com/7b543db89d07ef625e070422b0b14e2d53c2baed0edca5f42aec446568a519f2/pydantic/pydantic/pull/7288
```
몰랐는데 오픈소스 기여도 했었다


```cardlink
url: https://pydantic.dev/articles/pydantic-v2
title: "Pydantic V2 Plan | Pydantic"
host: pydantic.dev
favicon: https://pydantic.dev/favicon/favicon-32x32.png
image: https://pydantic.dev/articles/pydantic-v2/og.png
```
Pydantic 커뮤니티에서도 내부를 Rust로 구현하는 V2를 준비하고 있다고 한다.

```cardlink
url: https://pydantic.dev/articles/pydantic-v2-final
title: "Pydantic V2 Is Here! | Pydantic"
host: pydantic.dev
favicon: https://pydantic.dev/favicon/favicon-32x32.png
image: https://pydantic.dev/articles/pydantic-v2-final/og.png
```

더 느려졌다는 제보도 있다

```cardlink
url: https://github.com/pydantic/pydantic/discussions/6748
title: "Pydantic v2 significantly slower than v1 · pydantic/pydantic · Discussion #6748"
description: "I can't post the code as it's internal, but I'll try my best to describe and provide what information I can. We have a heavily nested, big object. Some of the child objects have some custom validat..."
host: github.com
favicon: https://github.githubassets.com/favicons/favicon.svg
image: https://opengraph.githubassets.com/f5aa4c9ba7bf66e275d0a9a1aaad5f14a7d39a3e3a2581642f42b9b0193e6111/pydantic/pydantic/discussions/6748
```

