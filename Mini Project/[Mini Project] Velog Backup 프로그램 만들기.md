[Velog로 가기](https://velog.io/@choi-hyk/Mini-Project-Velog-Backup-프로그램-만들기)

released at 2025-08-30 23:36:55 KST

updated at 2025-10-27 06:23:18 KST

|[Python Package Index](https://velog.io/tags/Python-Package-Index)|[graphql](https://velog.io/tags/graphql)|[mini project](https://velog.io/tags/mini-project)|[pypi](https://velog.io/tags/pypi)|[python deploy](https://velog.io/tags/python-deploy)|[velog](https://velog.io/tags/velog)|
|----|----|----|----|----|----|

# 📦 Velog Backup 프로그램 만들기

오늘은 갑자기 미니 프로젝트를 하고 싶어져서, Velog 포스트들을 백업해주는 프로그램을 만들려고 한다. 갑자기 미니 프로젝트를 하는 이유는 딱히 없다. 그냥 해보고 싶어졌다. 프로그램 목적은 사용자의 이름을 환경변수로 주면 GrphQL로 Velog 정보를 가져와서 시리즈 별 디렉토리에 포스트들을 저장하는 방식이다.

그리고 **Python Package Index** 를 통해 배포를 해보고 **GitHub Actions** 로 자동화 까지 가능하도록 구현할 생각이다.

---

## 🔍 GraphQL

내가 GitHub Pages를 만들 때, 그때도 Velog 포스트들을 가져와서 GitHub Pages에 출력 해주는 API를 만들었다. 그떄는 RSS를 사용해서 가져왔다. RSS는 근데 시리즈랑 프로필에 대한 상세한 정보가 없어서, 단순하게 내가 제목에 대괄호로 자체 태그를 만드는 걸 이용해서 포스트들의 시리즈를 구분했다. 

이번 프로젝트는 다른 사람들이 전부 사용 가능하도록 GraphQL을 활용해서 시리즈를 추출할 생각이다. 참고로 GraphQL은 **Facebook(현 Meta)** 가 2012년 개발해서, 2015년 공개한 API 쿼리 언어라고 한다. 클라이언트가 필요한 데이터만 정확히 요청할 수 있도록 설계된 데이터 질의 언어인데, 서버와 클라이언트 간 데이터 통신을 더 유연하고 효율적으로 만들어 준다...

이번에 처음 써보는데, 클라이언트가 원하는 데이터만 응답해주는 것이 특징이다. 내가 느낀 건, GraphQL은 마치 "필요한 만큼만 담아오는 주문표" 같은 느낌이다. REST API에서는 `/posts` 요청하면 정해진 형식대로 모든 데이터가 쏟아지는데, GraphQL은 `title`이나 `tags`만 원하면 그것만 딱 주고, `series`까지 원하면 그것도 같이 준다. 그래서 불필요한 데이터 전송이 줄고, 필요한 관계형 데이터도 한 번에 가져올 수 있다. 대신 스키마랑 쿼리를 직접 설계해야 해서, 초반에는 좀 낯설고 복잡하게 느껴질 수도 있다. 하지만 익숙해지면 데이터 흐름이 훨씬 깔끔해지고, 특히 내가 이번에 시리즈 정보까지 정리해서 가져오려는 것처럼, RSS보다 훨씬 세밀하게 제어할 수 있는 게 장점이다.

```python
import requests

ENDPOINT = "https://v2.velog.io/graphql"

def gql(query: str, variables: dict | None = None) -> dict:
    """
    GraphQL 쿼리를 실행하는 함수

    Args:
        query (str): GraphQL 쿼리 문자열
        variables (dict | None, optional): 쿼리 변수

    Returns:
        data["data"] (dict): GraphQL 응답 데이터
    """
    payload = {"query": query, "variables": variables or {}}
    res = requests.post(ENDPOINT, json=payload, timeout=15)
    res.raise_for_status()
    data = res.json()
    if "errors" in data:
        msgs = "; ".join(e.get("message", "") for e in data["errors"])
        raise RuntimeError(f"GraphQL 오류: {msgs}")
    return data["data"]
```

위의 코드는 Velog의 정보를 가져오는 GraphQL 실행 함수이다. 저기 `payload` 에 내가 원하는 데이터의 정보를 넣게 되면, 해당 정보를 응답해 준다.

### QUERY
```python
PROFILE_QUERY = """
query UserProfile($username: String!) {
    user(username: $username) {
        id
        username
        profile {
            display_name
            thumbnail
        }
    }
}
"""

LIST_QUERY = """
query Posts($username: String!, $limit: Int!, $cursor: ID) {
    posts(username: $username, limit: $limit, cursor: $cursor) {
        id
        url_slug
    }
}
"""

DETAIL_QUERY = """
query ReadPost($username: String!, $slug: String!) {
    post(username: $username, url_slug: $slug) {
        id
        url_slug
        title
        thumbnail
        tags
        series { name }
        released_at
        updated_at
        is_markdown
        body
        likes
    }
}
"""
```

앞에서 말한 것 처럼 GraphQL에서는 Route가 없고, 클라이언트가 무슨 payload를 보내느냐에 따라 오는 응답이 달라진다. 나는 Velog 사용자의 프로필과, 모든 포스트 정보, 그리고 각 포스트의 컨텐츠 3개가 필요하다. 프로필은 `PROFILE_QUERY`를 통해 요청이 가능했다. 간단하게 Velog 유저 이름을 보내면 프로필 정보를 보내준다. 다음은 `LIST_QUERY`를 통해서 모든 포스트 정보를 가져왔다. 해당 쿼리가 제일 복잡한데, 이유는 GraphQL은 리스트를 요청할때 한번에 요청이 가능한 한도가 정해져 있어서 `cursor` 와 `limit`로 메세지 큐를 보내는 것처럼 잘라서 받아야 한다. 그래서 `cursor` 와 `limit` 가 `LIST_QUERY`를 보면 설정되어 있다. 마지막으로 `DETAIL_QUERY`는 `url_slug` 라는 `LIST_QUERY`에서 가져온 포스트들의 url로 해당 포스트의 컨텐츠를 가져온다. 이렇게 모든 정보를 가져오면 이제 간단하다. 각 시리즈들을 폴더로 만들고, 해당 폴더 안에 시리즈에 해당하는 포스트들을 md파일로 생성하면 된다. 매우 고맙게도 GraphQL은 응답을 md파일로 해줘서 매우 편했다. ~~RSS를 사용할때는 html형식을 md로 바꿔야해서 짜증이 났다.~~

---

## ⚙️ pyproject.toml

이제 해당 프로젝트를 빌드를 하고, 빌드 파일을 배포해보겠다. 파이썬은 배포 환경이 매우 잘 되어 있는데 빌드를 **PyPI**에 업로드 하며 우리가 흔히 파이썬 패키지를 다운 받을 때 사용하는 `pip install`이 가능하다. 

파이썬에서 패키지를 배포할 때는 `pyproject.toml` 파일을 작성해야 한다. 이게 일종의 **패키지 설정서** 역할을 하는데, 프로젝트 이름부터 버전, 의존성, 빌드 방식까지 전부 여기에 정의한다. 내가 작성한 항목들을 하나씩 보면 이렇다:

```bash
[project]
name = "velog_sync"
version = "0.1.0"
description = "Velog 글을 Markdown으로 백업 (시리즈별 폴더) — velog_sync PyPI 패키지 실행"
readme = "README.md"
requires-python = ">=3.10"
authors = [{ name = "choi-hyk", email = "blindlchoil@gmail.com" }]
license = { file = "LICENSE" }   
classifiers = [
  "License :: OSI Approved :: MIT License",
]
dependencies = [
  "requests>=2.32.0",
  "tzdata>=2024.1"   
]

[project.scripts]
velog-sync = "velog_sync:main"

[tool.setuptools]
py-modules = ["velog_sync"]

[build-system]
requires = ["setuptools>=68", "wheel"]
build-backend = "setuptools.build_meta"
```

---

### [project]

#### name: `"velog_sync"`  
- PyPI에 올라갈 패키지 이름. `pip install velog-sync` 할 때 쓰이는 이름이다.
- 참고로 언더바 ( _ ) 는 하이픈 ( - )으로 바뀐다
#### version: `"0.1.0"`  
- 패키지 버전. SemVer(주버전.부버전.패치버전) 규칙을 따른다.  

#### description: 패키지 간단 설명.  

#### readme: `"README.md"`  
- PyPI 페이지에 표시될 문서.  

#### requires-python: `">=3.10"`  
-  파이썬 최소 버전. 여기서는 Python 3.10 이상만 지원하도록 했다.  

#### authors: 작성자 정보. 이름과 이메일을 적을 수 있다. 

#### license = { file = "LICENSE" }   
- 라이선스 파일을 명시해준다.

#### classifiers
- 라이선스의 종류를 명시해준다.

#### dependencies:  
- `requests>=2.32.0`: HTTP 요청용 라이브러리  
- `tzdata>=2024.1`: 타임존 데이터용 라이브러리
- 패키지를 설치할 때 자동으로 같이 설치된다.

---

### [project.scripts]

```bash
velog-sync = "velog_sync:main"
```
이 프로젝트가 단일 파이썬 파일(velog_sync.py)로 구성되어 있다는 걸 명시한다. 패키지 디렉토리 구조가 아니라 .py 모듈을 main 함수로 실행하면 위와 같이 적는다. 함수는 본인이 알아서 설정 가능하다.

---

### [build-system]

```bash
requires = ["setuptools>=68", "wheel"]
build-backend = "setuptools.build_meta"
````

빌드할 때 어떤 툴을 사용할지 지정한다. `setuptools`와 `wheel`이 필요하다고 정의했고, `setuptools.build_meta`를 빌드 백엔드로 사용한다고 명시했다. 이 설정 덕분에 `python -m build` 명령으로 `.tar.gz`와 `.whl` 빌드 파일을 만들 수 있다.

여기서 중요한 건, `build-system.requires`에 적은 패키지들이 실제 실행 환경에 필요한 건 아니라는 점이다. 이건 어디까지나 **빌드 과정에서만 필요한 도구**라서, 패키지를 설치하는 사람 입장에서는 신경 쓸 필요가 없다. 그리고 `setuptools.build_meta`는 일종의 빌드 엔진 역할을 하는데, `pip install .` 같은 명령을 실행했을 때 내부적으로 `build_wheel`, `build_sdist` 같은 함수를 호출해서 배포 파일을 만들어준다.

---

## 🚀 PyPI 배포하기

배포를 하려면 PyPI에 계정을 만들고, Token을 받아서 등록을 해야 한다.

![](https://velog.velcdn.com/images/choi-hyk/post/cc358957-8460-4e96-a68a-22a30a7ac5cd/image.png)

이제 해당 토큰을 자신의 로컬에 등록을 하면 된다.

```
이 토큰을 사용하세요.
이 API 토큰을 사용하려면:

__token__에 사용자 이름을 설정합니다
pypi- 접두사를 포함하여 비밀번호를 토큰 값으로 설정하세요
예를 들어, 프로젝트를 PyPI에 업로드하기 위해 Twine을 사용하는 경우, $HOME/.pypirc 파일을 다음과 같이 설정하세요:

[pypi]
  username = __token__
  password = TOKEN
```

위의 설정을 보고 로컬에 등록을 하면 로컬에서 배포가 가능하다. 먼저 빌드를 통해 코드를 배포 가능한 형태인 `.tar.gz`, `.whl`로 만들어야 한다.

### 1. 빌드

`python -m build` 를 실행하면 dist/ 디렉토리에 아래와 같은 파일이 생긴다.
- velog_sync-0.1.0.tar.gz (소스 배포본)
- velog_sync-0.1.0-py3-none-any.whl (휠 파일)


---

### 2. 업로드

이제 twine을 사용해서 PyPI에 업로드한다:

```bash
twine upload dist/*
```

여기서 `.pypirc` 파일에 등록해둔 토큰이 자동으로 사용된다. 업로드가 성공하면 PyPI 패키지 페이지에 바로 반영된다. 

---

### 3. 설치 확인

업로드가 끝나면 실제로 잘 올라갔는지 pip로 설치해본다:

```bash
pip install velog-sync
```

설치가 잘 되고, 내가 지정한 `velog-sync` 명령어까지 정상 실행되면 배포 완료다.

---

## 🤖 GitHub Actions 배포


```yaml
name: Publish to PyPI   # 워크플로우 이름 (GitHub Actions 탭에 표시됨)

on:
    push:
        tags: ["v*"]    # 태그가 v로 시작하는 커밋이 push될 때 실행됨 (예: v0.1.0, v1.0.0)

jobs:
    pypi-publish:
        name: Upload release to PyPI  # 잡 이름
        runs-on: ubuntu-latest        # 실행 환경: 최신 Ubuntu GitHub Runner 사용

        permissions:
            contents: read            # 리포지토리 컨텐츠 읽기 권한
            id-token: write           # OIDC(OpenID Connect) 토큰 발급 권한 → PyPI에 인증용

        steps:
            # 1. 코드 체크아웃
            - uses: actions/checkout@v4
              # GitHub Actions 런너에 현재 레포지토리 코드 가져오기

            # 2. Python 설치
            - uses: actions/setup-python@v5
              with:
                  python-version: "3.12"   # 파이썬 3.12 환경 구성

            # 3. 빌드 단계
            - name: Build
              run: |
                  python -m pip install --upgrade pip  # pip 최신화
                  pip install build                    # build 패키지 설치
                  python -m build                      # pyproject.toml 기반으로 dist/에 빌드 산출물 생성

            # 4. PyPI 업로드
            - name: Publish to PyPI
              uses: pypa/gh-action-pypi-publish@release/v1
              with:
                  skip-existing: true  # 이미 업로드된 파일이 있으면 스킵(중복 업로드 방지)
```

위와 같이 구성이 가능한데, 살펴볼 점은 태그랑 인증 방법이다. GitHub Actions는 태그 설정을 통해 배포 자동화가 이루어진다.  예를 들어 `git tag v0.1.0` 을 하게 되면, 바뀐 버전이 해당 액션으로 자동 배포가 이루어진다.

다음은 PyPI의 인증 방식인데, 기존에 로컬에서는 Token을 발급받아서, 배포를 하였는데, PyPI는 GitHub Actions와 같이 자동화 툴들을 위해  **PyPI Trusted Publisher**라는 방법을 제공한다. 예전처럼 `.pypirc`에 비밀번호 저장하는 게 아니라, GitHub OIDC(OpenID Connect) 토큰을 이용해서 **PyPI Trusted Publisher**로 인증한다. 즉, GitHub 저장소와 PyPI 계정을 연결해두면 비밀번호/토큰 노출 없이 안전하게 배포 가능하다.  **PyPI Trusted Publisher** 를 사용하려면 자신의 PyPI 계정에 해당 GitHub repo를 등록하면 된다.

<img width="838" height="217" alt="Image" src="https://github.com/user-attachments/assets/678f20aa-f48b-470c-a03c-3005dca06da4" />

난 이렇게 등록을 하였다. 

실행을 할때는 패치된 버전의 코드와 `pyproject.toml` 의 버전을 올리고 push와 push tag를 해줘야 한다. 참고로 `git tag` 명령어를 통해 tag를 등록하고 기존의 푸쉬 방법 처럼 `git push origin v0.1.0` 과 같은 방법으로 배포를 해줄 수 있다. 이때 주의할 점은 반드시 패치된 버전의 코드와 `pyproject.toml` 의 버전을 푸쉬해 놓은 상태여야 한다.

---

## 🔄 GitHub Actions로 velog-sync 자동화 하기

이제 로컬 배포와 GitHub Actions 배포도 구성을 하였으니, 실제로 사용자들이 쓸 수 있도록 GitHub Actions의 yml 파일을 제공하면 된다. 로컬에서 사용할 사람은 로컬에서 실행해서 백업을 진행하면 되고 나는 사용자들이 매일 03:00 시에 자동으로 Velog 포스트들을 GitHub repo에 업로드 되도록 yml 파일을 구성하였다.

```yml
name: velog-sync (daily KST 03:00)

on:
    schedule:
        - cron: "0 18 * * *" # 매일 03:00 KST
    workflow_dispatch: {}

permissions:
    contents: write

jobs:
    sync:
        runs-on: ubuntu-latest
        environment: velog_sync
        steps:
            - name: Checkout
              uses: actions/checkout@v4

            - name: Set up Python
              uses: actions/setup-python@v5
              with:
                  python-version: "3.11"

            - name: Install velog-sync
              run: |
                  python -m pip install --upgrade pip
                  pip install velog-sync

            - name: Run velog-sync
              env:
                  VELOG_USERNAME: ${{ vars.VELOG_USERNAME }}
              run: velog-sync

            - name: Configure Git
              run: |
                  git config user.name "github-actions[bot]"
                  git config user.email "41898282+github-actions[bot]@users.noreply.github.com"

            - name: Rebase with remote main
              run: |
                  git pull --rebase --autostash origin main

            - name: Commit if changed
              env:
                  TZ: Asia/Seoul
              run: |
                  if [ -n "$(git status --porcelain)" ]; then
                    DATE_KST="$(date +'%Y-%m-%d %H:%M:%S %Z')"
                    git add -A
                    git commit -m "chore: velog sync @ ${DATE_KST}"
                    git push
                  else
                    echo "No changes to commit."
                  fi
```

yml 파일에서는 내가 만든 패키지인 `velog-sync`를 다운받고 해당 패키지를 사용해서 등록한 유저 환경변수를 통해 GitHub에 업로드 해준다.

<img width="846" height="824" alt="Image" src="https://github.com/user-attachments/assets/3196d68c-5232-404d-806f-31739d6b9677" />

배포가 완료된 모습이다. 아래 링크에서 확인 가능하다.
[https://github.com/choi-hyk/Velog](https://github.com/choi-hyk/Velog)

---

# 🏁 마무리

오늘은 velog-sync라는 패키지를 만들고 배포까지 해보았는데, repo를 확인하고 이슈가 등록되면 개선해 나갈 생각이다. 그리고 지금은 Velog 가 조회수를 보여주는 API가 없지만, access_token을 통해 조회수를 확인 가능하다고 들었다. 그래서 해당 패키지에 access_token을 등록하여 조회수를 확인하는 기능을 넣고 싶다. 해당 패키지는 아래 링크에서 확인 가능하고, 이슈가 있으면 언제든지 등록을 해주길 바란다.

[[PyPI] velog-sync](https://pypi.org/project/velog-sync/)
[[GitHub] velog-sync](https://github.com/choi-hyk/velog_sync)




