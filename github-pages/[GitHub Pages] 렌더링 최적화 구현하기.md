[Velog로 가기](https://velog.io/@choi-hyk/GitHub-Pages-렌더링-최적화-구현하기)

released at 2025-07-31 19:39:10 KST

updated at 2026-03-05 06:26:09 KST

|[github pages](https://velog.io/tags/github-pages)|
|----|

## 🛠️ 렌더링 최적화 구현하기

오늘은 지난 시간에 이야기한 렌더링 최적화 기법을 적용해 보았다. 이전에 구현한 GitHub 페이지에서 프로필을 렌더링하는 데 2\~3초가 걸려 불편함이 있었는데, 처음에는 `Context`로 모든 데이터를 fetch하도록 할 수도 있지만 나는 페이지마다 fetch를 진행하기를 원했다 그래서 `Page` 마다 fetch함수를 구현했지만 `Route`로 이동할 때마다 fetch가 다시 실행되는 문제가 있었다. 

따라서 캐싱이나 다른 기법으로 이러한 fetch 동작을 최적화할 필요가 생겼고, 추가로 페이지별 markdown 데이터도 캐싱해야겠다고 생각했다.

---

### 구현 방법

React에서 사용할 수 있는 캐싱 기법에는 여러 가지가 있는데, 대표적으로 **`localStorage`를 사용하여 브라우저에 직접 저장하는 방식**과 **`SWR (stale-while-revalidate)` 라이브러리를 사용하는 방식**이 있다.

`SWR`을 사용하면 Route를 통해 컴포넌트가 다시 렌더링되어도 라이브러리에서 제공하는 hook으로 **데이터 변화를 감지하고 캐싱을 관리**해 준다. 캐싱된 데이터를 먼저 사용해 화면을 빠르게 렌더링하고, 백그라운드에서 fetch를 다시 진행해 데이터 변화를 감지한다. 또한 `{ data, isLoading, error }` 같은 상태 변수를 함께 제공해 편리하게 사용할 수 있다.

나는 `SWR`을 활용하여 API 호출에 대한 캐싱을 구현했다.

---

#### SWR 예제

```ts
import useSWR from "swr";
import useSWRMutation from "swr/mutation";
import { fetchEvents, fetchGitHub, fetchVelog, createEvent, ping } from "./api";

// ping
export function usePing() {
    return useSWR("ping", ping, { revalidateOnFocus: false });
}

// Calendar
export function useEvents() {
    return useSWR("events", fetchEvents, { revalidateOnFocus: false });
}

export function useCreateEvent() {
    return useSWRMutation(
        "events",
        async (key, { arg }: { arg: Parameters<typeof createEvent>[0] }) => {
            return await createEvent(arg);
        }
    );
}

// GitHub
export function useGitHub() {
    return useSWR("github-data", fetchGitHub, {
        revalidateOnFocus: false,
        dedupingInterval: 1000 * 60 * 5,
    });
}

// Velog
export function useVelog() {
    return useSWR("velog", fetchVelog, { revalidateOnFocus: false });
}
```

위 코드에서 `useSWR` hook으로 fetch 함수들을 등록하고 있다. 옵션으로 `{ revalidateOnFocus: false, dedupingInterval: 1000 * 60 * 5 }` 를 지정했는데, `revalidateOnFocus`는 **Route로 화면이 다시 포커스될 때 fetch를 다시 할지 여부**를 나타낸다. 위에서는 모두 `false`로 설정해 다시 fetch하지 않도록 했다. `dedupingInterval`은 **같은 key의 요청이 중복될 경우 일정 시간 동안 fetch를 생략하고 캐시된 데이터를 사용**하도록 하는 설정이다.

---

```ts
import { MarkdownRenderer } from "../../components/markdown/MarkDown";
import { GitHubProfile } from "../../components/github/GitHub";
import { DivCenteredWrapper } from "./GitHub.styles";
import { useGitHub } from "../../useApi";

function GitHub() {
    const { data, error, isLoading } = useGitHub();

    if (error) return null;
    if (isLoading) return null;
    if (!data) return null;

    const { profile } = data;

    return (
        <>
            <MarkdownRenderer page="github" />
            <DivCenteredWrapper>
                <GitHubProfile profile={profile} />
            </DivCenteredWrapper>
        </>
    );
}

export default GitHub;
```

컴포넌트에서는 위와 같이 `useGitHub` hook을 호출해 `{ data, error, isLoading }` 상태를 받아오고, 데이터가 준비되면 화면에 렌더링하도록 구성했다. 이 방식으로 API 호출이 캐싱되어 렌더링이 최적화된다.

---

### Custom Caching

마크다운 렌더링 캐싱은 React 전역 객체(`Map`)을 이용해 간단하게 구현했다. 현재 마크다운 렌더링은 page 단위로 이루어지므로 page 별로 데이터를 캐싱하여 불필요한 fetch를 줄였다.

```ts
const markdownCache = new Map<string, string>();

function MarkdownRenderer({ page }: { page: string }) {
    const [markdown, setMarkdown] = useState("");

    useEffect(() => {
        if (markdownCache.has(page)) {
            setMarkdown(markdownCache.get(page)!);
            return;
        }
        fetch(`/markdown/${page}.md`)
            .then((res) => res.text())
            .then((text) => {
                markdownCache.set(page, text);
                setMarkdown(text);
            })
            .catch((err) => console.error("파일 읽기 실패:", err));
    }, [page]);

    return (
        <DivCenteredWrapper>
            <ReactMarkdown rehypePlugins={[rehypeHighlight]}>
                {markdown}
            </ReactMarkdown>
        </DivCenteredWrapper>
    );
}
```

`const markdownCache = new Map<string, string>();` 로 전역 캐시를 선언하고, `page` 별로 데이터를 저장해 관리하도록 했다.

---

### 마무리

이렇게 API 호출은 `SWR`로, 마크다운은 `Map`을 이용한 단순 캐싱으로 렌더링 속도를 최적화했다. 다음 시간에는 GitHub 페이지를 마무리하고 **GitHub Pages**의 중간 점검을 진행할 예정이다.
