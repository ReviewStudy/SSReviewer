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