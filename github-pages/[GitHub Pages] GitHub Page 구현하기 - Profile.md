[Velog로 가기](https://velog.io/@choi-hyk/GitHub-Pages-GitHub-Page-구현하기-Profile)

released at 2025-07-30 19:46:23 KST

updated at 2025-10-27 04:10:40 KST

|[github pages](https://velog.io/tags/github-pages)|
|----|

## 🛠️ GitHub Page 구현하기

저번에 Velog 페이지를 구현한 것과 이어서, 오늘은 GitHub Page를 구현하려 한다. GitHub 페이지에 내 GitHub 프로필과 repo 정보, PRs, Issues를 출력하는 것이 목표이다.

오늘은 일단 프로필 출력을 구현하려고 한다.

---

## 구현 방법

구현 방법은 GitHub에서 제공하는 Public API를 사용하는 것이다. 여기서 PAT(Personal API Token) Token을 발급받아서 Header에 넣어주면 Public API를 시간당 5000회까지 요청이 가능하다. PAT는 앱을 등록하지 않아도 되어서 OAuth 요청을 위한 Access Token과는 완전히 다른 것이다. 단순히 개인 인증을 통해 토큰을 발급받아서 **사용자를 위임하는 OAuth** 기능과는 다르고, 나는 일단 단순히 내 GitHub 정보를 출력하는 것이 목적이기 때문에, PAT를 사용하였다.

<img width="1258" height="336" alt="Image" src="https://github.com/user-attachments/assets/a438dec7-e049-44f7-9312-c93d102bdd2e" />

이렇게 토큰을 발급받고 환경 변수로 서버에 등록을 해주면 된다. 그리고 해당 토큰을 API 요청을 할 때 Header로 넘겨주면 된다(매우 간단).

#### API Server

```js
const buildHeaders = () => {
    const headers: Record<string, string> = {
        Accept: "application/vnd.github+json",
    };
    if (process.env.GITHUB_TOKEN) {
        headers["Authorization"] = `token ${process.env.GITHUB_TOKEN}`;
    }
    return headers;
};

// GET GitHub Profile
app.get("/github/profile", async (req, res) => {
    try {
        const resGithub = await fetch(`https://api.github.com/users/choi-hyk`, {
            headers: buildHeaders(),
        });
        const data = await resGithub.json();
        res.json(data);
    } catch (e) {
        console.error(e);
        res.status(500).json({ error: "Failed to fetch GitHub profile" });
    }
});
```

`buildHeaders`라는 함수로 Headers에 `Accept` 필드를 JSON 객체로 생성하여 `Authorization: token ghp_xxxxx...` 형태의 값으로 넣어주게 된다.

이제 단순히 Header에 해당 값을 같이 넘겨주면 된다.

<img width="1909" height="245" alt="Image" src="https://github.com/user-attachments/assets/e0a2dc33-2cf8-4669-a78f-43e352347561" />

성공적으로 프로필 정보를 JSON 형태로 받아오는 것을 볼 수 있다.

#### Client

```ts
export async function fetchGitHub() {
    const profile = await axios.get(`${API_BASE_URL}/github/profile`);
    const repos = await axios.get(`${API_BASE_URL}/github/repo`);
    return { profile: profile.data, repos: repos.data };
}

export interface Profile {
    login: string;
    avatar_url: string;
    html_url: string;
    name: string;
    company: string;
    location: string;
    public_repos: number;
    followers: number;
    following: number;
}
```

일단 repo 정보를 가져오는 API도 구현을 했지만 해당 글에는 Profile만 설명을 하겠다.

인터페이스로 내가 사용할 정보를 선언하였다.

```ts
function GitHub() {
    const [profile, setProfile] = useState<Profile | null>(null);
    const [repos, setRepos] = useState([]);

    useEffect(() => {
        fetchGitHub().then((data) => {
            const { profile, repos } = data;
            setProfile(profile);
            setRepos(repos);
        });
    }, []);

    return (
        <>
            <MarkdownRenderer page="github" />
            <DivCenteredWrapper>
                <GitHubProfile profile={profile} />
            </DivCenteredWrapper>
        </>
    );
}
```

위에처럼 간단하게 구현을 완료하였다.

---

### 최종 결과

<img width="1919" height="904" alt="Image" src="https://github.com/user-attachments/assets/8445ce65-851b-4729-9dcc-bebaf4eed180" />

API를 요청하고 가져오는데 상당히 시간이 오래 걸리는 것 같다(한 2\~3초 정도 렌더링 시간이 걸림). 최적화하는 방법을 나중에 찾아봐야겠다.

---

### 마무리

오늘은 이렇게 GitHub Public API로 프로필 정보를 가져와서 출력하는 것까지 구현을 완료하였다. 다음에는 repo와 PRs, 그리고 Issues까지 출력하여 GitHub 페이지 구현을 완성해야겠다.


