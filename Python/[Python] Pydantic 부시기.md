[Velog로 가기](https://velog.io/@choi-hyk/Python-Pydantic-부셔버리기)

released at 2025-09-11 00:11:21 KST

updated at 2025-09-11 02:01:34 KST

|[FastAPI](https://velog.io/tags/FastAPI)|[Pydantic](https://velog.io/tags/Pydantic)|[asgi](https://velog.io/tags/asgi)|[python](https://velog.io/tags/python)|
|----|----|----|----|

# Pydantic ✨

오늘은 `Pydantic`에 대해서 알아보려고 한다. 최근에 오픈소스로 이루어진 프로젝트들을 보면, 백엔드 서버를 `FastAPI`를 사용하는 경우가 많은데, Pydantic은 `FastAPI`에서 데이터 스키마를 정의하고 데이터 직렬화/역직렬화를 위해 많이 사용하는 라이브러리이다. 최근 회사에서 오픈소스를 활용한 프로젝트를 유지보수하는 업무를 하고 있는데, 해당 프로젝트가 나는 별로 사용해본 적이 없는 `Svelte`를 프론트로 사용 중이고 백엔드는 `FastAPI` 기반의 **Monolithic** 아키텍처를 구성하고 있다. `Pydantic`은 얼핏보면 간단해 보이지만, 수 많은 ASGI 코드들과 정의되어 있는 스키마를 보면 어지러워 질때가 있다. 그래서 해당 라이브러리에 익숙해질 필요를 느껴서 이렇게 정리를 해보려고 한다.

찾아보니 `Pydantic`은 처음에는 **Samuel Colvin** 이라는 사람이 2018년 쯤에 Python 환경에서 타입 힌트화를 통한 데이터 무결성 보장과 타입 직렬화/역직렬화를 지원하기 위해 만들었다고 한다. ~~GPT 피셜~~. 나중에 FastAPI에서 공식적으로 Pydantic을 스키마 라이브러리로 채택하면서 널리 쓰이게 됐다고 한다. 특히, 데이터 스키마 정의를 통해 API 기반의 백엔드 서버의 라우터 문서화 **(OpenAPI/Swagger)** 를 자동화 하는 점이 큰 장점이다.  

---
## 사용법 🛠️

거두절미하고 바로 사용법을 알아보겠다. Pydantic을 써보면서 느낀점은 사용자 입맛대로 강력한 데이터 강제성을 주입 시킬 수 있다. 참고로 사용한 `Pydantic` 버전은 2.9.2 이다.

### BaseModel

```python
class User(BaseModel):
    id: int = Field(
        default_factory=lambda: int(uuid.uuid4()),
        description="사용자의 고유 ID",
    )
    name: str = Field(
        ..., min_length=1, max_length=20, description="사용자의 이름 (1~20자)"
    )
    email: str = Field(..., description="사용자의 이메일 주소")
    age: Optional[int] = Field(None, ge=0, description="사용자의 나이 (0 이상)")

    def __str__(self):
        return f"User(id={self.id}, name='{self.name}', email='{self.email}', age={self.age})"

    def to_model_dump(self):
        return self.model_dump()

    @classmethod
    def from_model_dump(cls, data):
        return cls.model_validate(data)

    @model_validator(mode="before")
    def check_email(cls, values):
        email = values.get("email")
        if email and "@" not in email:
            raise ValueError("Invalid email address")
        return values

```

이제 위의 코드를 보면 좀 어지러워 질텐데, 일단 `User` 스키마만 살펴보자.

```python
class User(BaseModel):
    id: int 
    name: str
    email: str 
    age: int
```
위의 스키마를 최대한 간단하게 정의하면 이렇게 작성할 수 있다. 먼저 `BaseModel` 은 `Pydantic`에서 해당 클래스가 스키마라는 것을 정의해주는 기본 클래스이다. 해당 클래스를 상속 함으로서 `User` 는 `Pydantic` 의 데이터 검증과 직렬화/역직렬화를 사용 가능하다. 

```python
user = User(id="123", name="Alice", email="user@example.com, age="25")
print(user) 
```
위 처럼 `User`를 정의했다고 생각해보자, 현재 `id` 와 `age` 는 `int` 형인데 `str` 형이 할당 되어있다. 마치 `JavaScript` 의 타입 캐스팅 처럼 `Pydantic` 은 바꿀 수 있는 타입은 알아서 바꿔 준다. 위의 경우에는 문제 없이 `int` 형으로 바뀔 것이다. 그러나 만약 "one two three" 같은 것이 할당되어 있으면, `ValidationError`를 발생 시킨다.

### Field

```python
class User(BaseModel):
    id: int = Field(
        default_factory=lambda: int(uuid.uuid4()),
        description="사용자의 고유 ID"
    )
    name: str = Field(
        ..., min_length=1, max_length=20, description="사용자의 이름 (1~20자)"
    )
    email: str = Field(
        ..., description="사용자의 이메일 주소"
    )
    age: Optional[int] = Field(
        None, ge=0, le=150, description="사용자의 나이 (0~150)"
    )
```

이번에는 `Field`에 대해서 알아보자. `Field`는 일종의 데이터 명세서로 단순히 타입만 지정했을 때보다 훨씬 세밀하게 제약조건과 메타데이터를 설정할 수 있게 해준다. 위 코드를 보면, 모든 필드가 `description` 을 통해 필드 설명을 제공 중이다. 이 값은 문서화가 되었을 때, API 설명 부분에 자동을 할당된다.

각 필드를 살펴보면, `id`의 `default_factory` 를 볼 수 있는데, 해당 인자는 해당 필드를 동적으로 값을 생성한다는 의미이다. `default` 도 있는데, 해당 값은 동적이 아니라 정해진 값을 생성해주는 인자이다. 참고로 밑에 처럼 `Field`를 사용 안하고, `default` 선언도 가능하다.

```python
class User(BaseModel):
    id: int = 10
```

다음으로는 `...` 을 볼 수 있는데, 해당 값은 필수 인자라는 뜻이다. 따라서 해당 스키마를 정의할 때, 해당 값들을 할당하지 않고 정의하면 `ValidationError` 가 발생한다. 

그 밖에도 여러가지 인자가 있는데, 밑에 표로 정리한 것을 살펴보면 이해가 편할 것이다.

#### Field 주요 인자 정리

| 인자                            | 설명                     | 예시                                            |
| ----------------------------- | ---------------------- | --------------------------------------------- |
| **default**                   | 기본값 지정                 | `Field(0)`                                    |
| **default\_factory**          | 동적으로 기본값 생성 (함수 실행 결과) | `Field(default_factory=lambda: uuid.uuid4())` |
| **... (Ellipsis)**            | 필수(required) 필드 지정     | `Field(...)`                                  |
| **title**                     | 필드 제목 (문서화용)           | `Field(..., title="User ID")`                 |
| **description**               | 필드 설명 (문서화용)           | `Field(..., description="사용자의 고유 ID")`        |
| **gt / ge**                   | 숫자 크기 제한 (>) / (≥)     | `Field(..., gt=0)`                            |
| **lt / le**                   | 숫자 크기 제한 (<) / (≤)     | `Field(..., le=100)`                          |
| **min\_length / max\_length** | 문자열 길이 제한              | `Field(..., min_length=1, max_length=20)`     |
| **pattern**                   | 정규식 패턴 검증              | `Field(..., pattern=r"^[a-z0-9]+$")`          |
| **alias**                     | 입력 받을 때 다른 키 이름 허용     | `Field(..., alias="user_id")`                 |
| **deprecated**                | 필드가 더 이상 쓰이지 않음을 표시    | `Field(..., deprecated=True)`                 |
| **examples**                  | API 문서에 예시 값 표시        | `Field(..., examples=["alice@example.com"])`  |

### Typing

`Pydantic` 은 `typing` 모듈의 정의 타입들을 사용하는데, 대표적으로 `Optional`이 있다.

```python
class User(BaseModel):
    id: int = Field(..., description="사용자 ID (필수)")
    name: str = Field(..., min_length=1, max_length=20, description="사용자 이름")
    age: Optional[int] = Field(None, ge=0, le=150, description="나이 (없으면 None)")
    phone: Union[str, int, None] = Field(
        None, description="전화번호 (문자열 또는 숫자 허용, 없으면 None)"
    )
    role: Literal["admin", "user", "guest"] = Field(
        "user", description="권한 (admin, user, guest 중 하나)"
    )
    tags: List[str] = Field(default_factory=list, description="사용자 태그 목록")
    preferences: Dict[str, str] = Field(
        default_factory=dict, description="사용자 환경설정"
    )
```
이런식으로 타입 정의가 가능한데, 참고로 `Union` 보다는 간단하게 파이프 연산자를 사용하는 것을 추천한다. 다른 타입도 많은데, `FastAPI` 데이터 스키마에서는 이 정도면 사용하는 것 같다.

### Method

`Pydantic` 은 기본 `Method` 기능을 제공한다. 오픈소스 코드에서도 이러한 기본 함수를 적극적으로 활용하고 있어서, 반드시 알아둬야 된다.


#### 1. **`__str__`**

→ 객체를 print 했을 때 사람이 읽기 좋은 문자열 반환

```python
from pydantic import BaseModel

class User(BaseModel):
    id: int
    name: str
    email: str

    def __str__(self):
        return f"User(id={self.id}, name={self.name}, email={self.email})"

u = User(id=1, name="Alice", email="alice@example.com")
print(u)  
# 출력: User(id=1, name=Alice, email=alice@example.com)
```

#### 2. **`model_dump()`**

→ 객체 → dict 직렬화

```python
data = u.model_dump()
print(data)
# 출력: {'id': 1, 'name': 'Alice', 'email': 'alice@example.com'}
```

#### 3. **`model_validate()`**

→ dict → 객체 (검증 포함)

```python
user_dict = {"id": 2, "name": "Bob", "email": "bob@example.com"}
u2 = User.model_validate(user_dict)
print(u2)
# 출력: User(id=2, name=Bob, email=bob@example.com)
```


위의 함수들을 통해 `Pydantic`의 핵심 기능인, 데이터 검증과 **직렬화/역직렬화**를 간편하게 적용 가능하다.

#### 4. **`@model_validator`**

→ 모델 생성 시 비즈니스 규칙 검증

```python
from pydantic import model_validator

class User(BaseModel):
    id: int
    name: str
    email: str

    @model_validator(mode="before")
    def check_email(cls, values):
        email = values.get("email")
        if email and "@" not in email:
            raise ValueError("Invalid email address")
        return values

# 올바른 입력
User(id=3, name="Charlie", email="charlie@example.com")

# 잘못된 입력 → 예외 발생
User(id=4, name="Dave", email="invalid-email")
# ValueError: Invalid email address
```
해당 함수는 `model_validate` 기능을 제공한다는 의미로 데코레이터로 정의 가능하다. 옆에 `(mode="before")` 는 스키마가 정의되기 전에 실행되는 함수라는 뜻이다. 따라서 이러한 검증 함수를 여러가지 만들 수가 있다.

### Model Nested

이제 `Pydantic`의 가장 강력한 기법이라 볼 수 있는 중첩을 알아보자.

```python
class ProjectConfig(BaseModel):
    owner: User
    members: List[User]
```

위의 방식 처럼 중첩을 사용해서 상위 스키마를 제공이 가능하다. 당연한 기능 같지만, `Pydantic`의 `BaseModel` 은 `Dict` 타입을 위에서 살펴본 모델 검증 과정을 통해 객체형으로 바꿔준다. 

만약 위의 스키마대로 `Dict` 타입을 정의했다고 해보자

```python
config = {
    "owner": {
        "name": "Alice",
        "email": "alice@example.com"
    },
    "members": [
        {"name": "Bob", "email": "bob@example.com"},
        {"name": "Charlie", "email": "charlie@example.com"}
    ]
}
```
위에서 정의된 `config` 에서 `members[0]` 를 살펴보려면 `config["members"][0]` 으로 접근이 가능하다.

```python
project = ProjectConfig.model_validate(config)
```
이제 `BaseModel`의 `model_validate` 로 변환을 해보자. 그러면 `project`는 객체가 되어서 
`config.members[0]` 으로 접근이 가능하게 된다. 수 많은 `FastAPI`를 사용한 오픈소스 에서는 이러한 형태로 강력한 타입 설정을 하여, 관리를 하고있다. `FastAPI` 에서는 `resquest` 와 `response` 에 스키마를 설정하여, 자동으로 `JSON`이 직렬화 된 데이터를 객체 형태로 받게된다. 

---

## FastAPI

```python
from typing import List
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

class User(BaseModel):
    id: int 
    name: str 
    email: str 
    age: Optional[int]

class ProjectConfig(BaseModel):
    owner: User
    members: List[User]
    
class ProjectResponse(BaseModel):
    owner_name: str
    member_count: int

@app.post("/projects", response_model=ProjectResponse)
async def create_project(project: ProjectConfig):
    return ProjectResponse(
        owner_name=project.owner.name,
        member_count=len(project.members)
    )
```

위의 코드를 보면, `create_project`에서 `project` 를 `ProjectConfig` 로 받고 있다. 만약에 클라이언트가 `Dict` 형태로 인자를 보내게 되면, 위의 포스트 라우터는 자동으로 `project`를 검증 및 변환하여 `ProjectConfig` 로 만들어 준다.

경로 옆 `response_model` 은 응답 타입도 정해주는 설정이다. 반환 값으로 `ProjectResponse` 스키마대로 반환을 하고 있다. 클라이언트는 해당 응답을 받으면, `Dict` 형태로 받게 된다. 이렇게도 쓸 수 있다.

```python
@app.post("/projects", response_model=ProjectResponse)
async def create_project(project: ProjectConfig):
    return {
        "owner_name": project.owner.name,
        "member_count": len(project.members)
    }
```
`FastAPI` 에서 `Dict` 형으로 반환을 해도, 자동으로 스키마를 감지해준다. 컨벤션에 맞게 두가지를 조율해서 사용하면 될 것 같다. 보통의 API에서는 클라이언트는 항상 `JSON` 으로 직렬화 된 데이터를 보내므로, 이러한 데이터를 검증하고 좀 더 쉽게 관리가 가능하다. 

참고로 상속 기능이 있긴 하지만, 상속 기능은 사용하면 너무 복잡해져서 많이 보진 못한 것 같다. 그래서 설명은 넘어가겠다.

---

## 마무리 😁

오늘은 `Pydantic`에 대해서 알아보았다. 처음에는 그냥 단순한 타입 정의 라이브러리라 생각하고, 찾아보지 않았다가, 실수나 타입 불일치 오류를 많이 보게 되었는데, 이번 기회에 제대로 알아보고 작업을 할 수 있을 것 같다. 기회가 되면, 회사에서 사용하는 스텍이나 라이브러리들을 하나씩 정리해서 학습을 해야겠다.

