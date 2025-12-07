[Velog로 가기](https://velog.io/@choi-hyk/GitHub-Pages-마크다운-컴포넌트-구현하기)

released at 2025-07-21 18:40:54 KST

updated at 2025-12-07 06:18:24 KST

|[github pages](https://velog.io/tags/github-pages)|
|----|

## Ⓜ️마크다운 컴포넌트

우스갯소리로 정리를 잘하면 실력이 좋은 개발자라는 소리가 있다. 자신이 개발한 내용을 자신 뿐만 아니라 다른 사람이 보아도 이해하기 쉽게 정리한 사람이야 말로 개발에 적합하다는 소리이다. 이러한 문서 정리용 언어를 경량 마크업 언어라고 하는데 대표적으로 `MarkDown` 과 `Wiki` 언어가 있는 것 같다.

나는 `MarkDown`에 익숙하므로 내 웹 사이트에 마크다운을 출력해주는 컴포넌트를 구현하려고 한다.

---

### 구현방법

당연하게도 마크다운을 html 상에서 출력해주는 라이브러리는 존재한다. 나는 `react-markdown` 이라는 라이브러리를 사용하였다. 

구현방법은 페이지마다 md 파일을 생성하여 GitHub Pages에 같이 배포하는 방식을 사용하였다.

<img width="159" height="46" alt="Image" src="https://github.com/user-attachments/assets/441a3fcb-4d27-4cb4-a3ba-705733b83df7" />

<img width="1019" height="310" alt="Image" src="https://github.com/user-attachments/assets/b436a1df-75f5-40dd-b70f-998acf6415c3" />

**makrdown 폴더가 같이 배포된 모습**

이제 컴포넌트는 라이브러리를 사용해서 페이지마다 존재하는 md 파일을 입력해주면 된다.

```ts
import { useEffect, useState } from "react";
import ReactMarkdown from "react-markdown";

export default function MarkdownRenderer({ page }: { page: string }) {
    const [markdown, setMarkdown] = useState("");

    useEffect(() => {
        fetch(`/markdown/${page}.md`)
            .then((res) => res.text())
            .then((text) => setMarkdown(text))
            .catch((err) => console.error("파일 읽기 실패:", err));
    }, []);

    return (
        <ReactMarkdown>{markdown}</ReactMarkdown>
    );
}

```

page Props에 `profile`이 들어가게 되면 `markdown/profile.md`를 `ReactMarkdown`을 통해 출력하게 된다.

마지막으로 스타일링을 통해 조정을 해주면 최종 구현이 된다.

```ts
import { useEffect, useState } from "react";
import { DivCenteredWrapper } from "./MarkDown.styles";
import ReactMarkdown from "react-markdown";
import rehypeHighlight from "rehype-highlight";
import "highlight.js/styles/github-dark.css";

export default function MarkdownRenderer({ page }: { page: string }) {
    const [markdown, setMarkdown] = useState("");

    useEffect(() => {
        fetch(`/markdown/${page}.md`)
            .then((res) => res.text())
            .then((text) => setMarkdown(text))
            .catch((err) => console.error("파일 읽기 실패:", err));
    }, []);

    return (
        <DivCenteredWrapper>
            <ReactMarkdown rehypePlugins={[rehypeHighlight]}>
                {markdown}
            </ReactMarkdown>
        </DivCenteredWrapper>
    );
}
```

`highlights` 라이브러리에서 github-dark 스타일을 적용하였다. 또한 `DivCenteredWrapper` 라는 마크다운 Wrapper를 적용하여 좀더 깔끔하게 보이게 하였다.

---

### 😘프로필 화면

<img width="1902" height="910" alt="Image" src="https://github.com/user-attachments/assets/c3b27a75-cee2-4d81-b87a-3658a7ef5a40" />
