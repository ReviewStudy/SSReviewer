> JWT(JSON Web Token)는 인증과 정보를 안전하게 전달하는 데 사용하는 **토큰(Token) 기반 인증 방식**

JWT는 세 부분으로 구성됨.
> 헤더(Header).페이로드(Payload).서명(Signature)

**Header**
- 어떤 알고리즘으로 서명헀는지 정보
```
{
  "alg": "HS256",
  "typ": "JWT"
}
```

**Payload**
- 사용자의 정보(예: ID, 권한 등)를 담고 있음
```
{
  "user_id": 123,
  "role": "admin",
  "exp": 1710000000
}
```

**Signature**
- **토큰이 변조되지 않았는지 검증**하는 역할.
- 비밀 키를 사용해서 생성됨.

**장점**
- **세션 저장 필요 없음** (서버에서 상태를 저장하지 않아도 됨 → 확장성이 좋음) (Stateless)
- **빠른 인증** (매 요청마다 DB 조회 없이 검증 가능)
- **다양한 서비스에서 사용 가능** (예: OAuth)

**단점**
- 한번 발급된 JWT는 수정이 어려움
- JWT가 유출되면 누구나 사용할 수 있음
	- HTTPS 사용
	- 짧은 만료 시간 설정
	- [[Refresh Token]] 활용

> 액세스 토큰이 만료되었을 때, 로그인을 다시 하지 않아도 새로운 액세스 토큰을 받을 수 있도록 해주는 역할

**필요한 이유**
- JWT는 만료 시간이 짧게 설정되는 경우가 많음
- 이때 사용자가 계속해서 로그인을 반복하는 것은 힘듦
- Refresh Token을 통해 자동으로 새로운 Access Token을 발급함

**동작 방식**
1. 사용자가 로그인하면 Access Token과 Refresh Token을 발급함
2. 사용자는 Access Token을 통해 API 요청을 보냄
3. Acess Token이 만료되면 Refresh Token을 서버에 보냄
4. 서버가 Refresh Token을 검증하고, 새로운 Access Token을 발급함



**장점**
- 로그인 유지 가능
- 보안 강화 (Access Token이 짧은 만료 시간을 가지게 할 수 있음)

**단점**
- Refresh Token이 털리면 답 없음
- 추가적인 서버 로직 (Refresh Token 발급 및 검증)


**생성 Tip**
- 길고 안전한 난수 값으로
- **서버에서 저장 & 검증 (DB or Redis)**
	- **유출되거나 갱신이 필요할 때 파기 가능**
	- **Stateless가 깨지긴 함**
- HTTPS 사용
- 기기별 Refresh Token 관리
- 주기적으로 갱신


**Rotate Refresh Token**
**방법 21: Refresh Token을 저장**

- Refresh Token을 **DB 또는 Redis에 저장하고, Access Token은 Stateless하게 유지**.
- Refresh Token이 유출되면 **서버에서 무효화 가능**.
- **완전한 Stateless는 아니지만, 보안성과 확장성을 확보**.

**방법 2: Rotate Refresh Token**
1. 클라이언트가 **Refresh Token을 서버에 보냄**.
2. 서버는 **새로운 Refresh Token과 Access Token을 발급**.
3. 기존 Refresh Token을 **즉시 폐기**.
4. 클라이언트는 **새로운 Refresh Token을 사용**

FastAPI에서는 어떻게 구현할 수 있을까?

common/auth.py
```python
SECRET_KEY = "THIS_IS_A_SECRET"
ALGORITHM = "HS256"

oauth2_scheme = OAuth2PasswordBearer(tokenUrl="/users/login")

class Role(StrEnum):
    USER = "USER"
    ADMIN = "ADMIN"
    
@dataclass
class CurrentUser:
    id: str
    role: Role

def create_access_token(payload: dict, role: Role, expires_delta: timedelta = timedelta(hours=6)):
    expire = datetime.utcnow() + expires_delta
    payload.update({"exp": expire, "role": role})
    encoded_jwt = jwt.encode(payload, SECRET_KEY, algorithm=ALGORITHM)
    return encoded_jwt

def decode_access_token(token: str):
    try:
        payload = jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
        return payload
    except JWTError:
        raise HTTPException(status_code=status.HTTP_401_UNAUTHORIZED, detail="Invalid token")

def get_current_user(token: Annotated[str, Depends(oauth2_scheme)]):
    payload = decode_access_token(token)
    user_id = payload.get("user_id")
    role = payload.get("role")
    if not user_id or not role or role != Role.USER:
        raise HTTPException(status_code=status.HTTP_403_FORBIDDEN, detail="Not enough permissions")
    return CurrentUser(id=user_id, role=Role(role))
    
def get_admin_user(token: Annotated[str, Depends(oauth2_scheme)]):
    payload = decode_access_token(token)
    role = payload.get("role")
    if not role or role != Role.ADMIN:
        raise HTTPException(status_code=status.HTTP_403_FORBIDDEN, detail="Not enough permissions")
    return CurrentUser(id="ADMIN_USER_ID", role=Role(role))
```

user_contoller
```python
@router.post("/login")
@inject
def login(form_data: Annotated[OAuth2PasswordRequestForm, Depends()], user_service: UserService = Depends(Provide[Container.user_service])):
    access_token = user_service.login(email=form_data.username, password=form_data.password)
    return {"access_token": access_token, "token_type": "bearer"}



@router.put("", response_model=UserResponse)
@inject
def update_user(
    current_user: Annotated[CurrentUser, Depends(get_current_user)],
    body_data: UpdateUserBody,
    user_service: UserService = Depends(Provide[Container.user_service]),
):
    user = user_service.update_user(current_user.id, body_data.name, body_data.password)
    return user