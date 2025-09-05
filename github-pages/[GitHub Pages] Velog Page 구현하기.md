[Velog로 가기](https://velog.io/@choi-hyk/GitHub-Pages-Velog-Page-구현하기)

released at 2025-07-29 18:08:36 KST

updated at 2025-09-05 15:06:26 KST

|[github pages](https://velog.io/tags/github-pages)|
|----|

## 🛠️ Velog Page 구현하기

현재 사이트의 사이드바에 study 페이지가 존재하는데, 여기에 **Velog**, **GitHub**, **Study** 이 3개의 페이지를 추가하여 관리할 계획이다.
Velog 아이콘은 나중에 만들기로 하고, 우선 study 페이지에 Velog 기능부터 구현해 보았다.

---

### 구현 방법

Velog는 **RSS(Really Simple Syndication)** 형태로 글 정보를 제공한다.
RSS는 웹 페이지의 정보를 XML 형태로 제공하는 표준 데이터 형식으로, 뉴스 사이트나 블로그 등에서 많이 사용된다.
이를 통해 사람들은 별도의 API 없이도 공개된 글 데이터를 손쉽게 가져와 활용할 수 있다.

Velog는 공식 API를 제공하지 않는다. 따라서 번거롭게 API를 구현할 필요 없이, **RSS URL로 XML 데이터를 가져와 파싱**하면 글 목록을 쉽게 가져올 수 있다.

<img width="1918" height="969" alt="Image" src="https://github.com/user-attachments/assets/f51abee4-c426-4144-875f-e7898a4c8b36" />

위 이미지는 내 Velog RSS URL에 직접 접속한 화면이다.

처음에는 클라이언트에서 바로 RSS URL을 fetch로 가져오려고 했으나, **CORS 정책** 때문에 접근할 수 없었다.
그래서 EC2에서 실행 중인 서버에 라우터를 추가해, 서버를 통해 RSS 데이터를 받아오는 방식을 사용했다.

---

### 서버 코드

```ts
import axios from "axios";
import Parser from "rss-parser";
import TurndownService from "turndown";

const parser = new Parser();
const turndownService = new TurndownService();

app.get("/velog", async (req, res) => {
    try {
        const url = `https://v2.velog.io/rss/choi-hyk`;
        const xml = await axios.get<string>(url);
        const feed = await parser.parseString(xml.data);

        const posts = feed.items.map((item, index, array) => {
            const html = (item["content:encoded"] ||
                item.content ||
                item.contentSnippet ||
                "") as string;
            const markdown = turndownService.turndown(html);

            const rawTitle = item.title ?? "";
            const tagMatch = rawTitle.match(/^\[(.*?)\]\s*(.*)$/);

            return {
                id: array.length - 1 - index,
                tag: tagMatch ? tagMatch[1] : null,
                title: tagMatch ? tagMatch[2] : rawTitle,
                link: item.link,
                pubDate: item.pubDate,
                description: markdown,
            };
        });

        res.json(posts);
    } catch (err) {
        console.error(err);
        res.status(500).json({ error: "Failed to fetch Velog RSS" });
    }
});
```

여기서 `url`에 `https://v2.velog.io/rss/choi-hyk`를 지정해 RSS XML 데이터를 가져오고,
`axios`, `rss-parser`, `turndown`을 이용해 XML을 파싱하고 HTML을 Markdown으로 변환하여 JSON 형태로 응답한다.

---

### JSON 변환 로직

```ts
const posts = feed.items.map((item, index, array) => {
    const html = (item["content:encoded"] ||
        item.content ||
        item.contentSnippet ||
        "") as string;
    const markdown = turndownService.turndown(html);

    const rawTitle = item.title ?? "";
    const tagMatch = rawTitle.match(/^\[(.*?)\]\s*(.*)$/);

    return {
        id: array.length - 1 - index,
        tag: tagMatch ? tagMatch[1] : null,
        title: tagMatch ? tagMatch[2] : rawTitle,
        link: item.link,
        pubDate: item.pubDate,
        description: markdown,
    };
});
```

RSS 데이터에는 태그 정보가 따로 없어서, 글 제목 앞에 `[GitHub Pages]`와 같이 직접 적어둔 분류명을 사용해 태그를 추출하는 방식으로 처리했다.
React에서 key 값으로 사용할 **id**도 여기서 생성했다.

<img width="1892" height="967" alt="Image" src="https://github.com/user-attachments/assets/619ab9b9-6049-416f-930b-c7bb7559ac43" />

위 이미지처럼 API를 통해 JSON 형태로 글 데이터를 가져오는 데 성공했다.

이제 이 데이터를 클라이언트에 연동해서, Velog 글을 선택하면 **오른쪽 영역에 마크다운으로 글 내용을 표시하고, 태그 버튼으로 분류별 필터링**이 가능하게 만들었다.

#### Client

```tsx
import { useEffect, useState } from "react";
import {
    ButtonTag,
    DivMarkdownWrapper,
    DivPostInit,
    DivPostTag,
    DivPostTitle,
    DivPostWrapper,
    DivTagContainer,
    DivVelogWrapper,
} from "./Study.styles";
import { fetchVelog, Post } from "../../api";
import {
    MarkDownPostRenderer,
    MarkdownRenderer,
} from "../../components/markdown/MarkDown";
import moment from "moment";

function Study() {
    const [posts, setPosts] = useState<Post[]>([]);
    const [selectedTag, setSelectedTag] = useState<string | null>(null);
    const [selectedPost, setSelectedPost] = useState<Post | null>(null);

    useEffect(() => {
        fetchVelog().then((posts) => {
            setPosts(posts);
            setSelectedPost(posts[0] || null);
        });
    }, []);

    const tags = Array.from(new Set(posts.map((p) => p.tag || "기타")));

    const grouped = posts.reduce((acc, post) => {
        const key = post.tag || "기타";
        if (!acc[key]) acc[key] = [];
        acc[key].push(post);
        return acc;
    }, {} as Record<string, Post[]>);

    const visiblePosts =
        selectedTag && grouped[selectedTag] ? grouped[selectedTag] : posts;

    return (
        <>
            <MarkdownRenderer page="study" />
            <DivTagContainer>
                <ButtonTag
                    onClick={() => setSelectedTag(null)}
                    $active={selectedTag === null}
                >
                    전체
                </ButtonTag>
                {tags.map((tag) => (
                    <ButtonTag
                        key={tag}
                        onClick={() => setSelectedTag(tag)}
                        $active={selectedTag === tag}
                    >
                        {tag}
                    </ButtonTag>
                ))}
            </DivTagContainer>
            <DivVelogWrapper>
                <DivPostWrapper>
                    {visiblePosts.map((post) => (
                        <DivPostInit
                            key={post.id}
                            onClick={() => setSelectedPost(post)}
                        >
                            <DivPostTag>{post.tag}</DivPostTag>
                            <DivPostTitle $active={selectedPost === post}>
                                {post.title}
                            </DivPostTitle>
                        </DivPostInit>
                    ))}
                </DivPostWrapper>
                <DivMarkdownWrapper>
                    <h5
                        style={{
                            color: "var(--third-text-color)",
                            paddingLeft: "40px",
                        }}
                    >
                        {moment(selectedPost?.pubDate).format(
                            "YYYY-MM-DD HH:mm"
                        )}
                    </h5>
                    <h4
                        style={{
                            color: "var(--third-text-color)",
                            paddingLeft: "40px",
                        }}
                    >
                        {selectedPost?.link && (
                            <a
                                href={selectedPost.link}
                                target="_blank"
                                rel="noopener noreferrer"
                                style={{
                                    color: "var(--fourth-text-color)",
                                    textDecoration: "underline",
                                }}
                            >
                                Velog로 이동하기
                            </a>
                        )}
                    </h4>
                    <h1
                        style={{
                            color: "var(--secondary-text-color)",
                            paddingLeft: "40px",
                        }}
                    >
                        <span
                            style={{
                                color: "var(--tertiary-text-color)",
                            }}
                        >
                            [{selectedPost?.tag}]
                        </span>
                        <span> {selectedPost?.title}</span>
                    </h1>
                    {selectedPost && (
                        <MarkDownPostRenderer post={selectedPost} />
                    )}
                </DivMarkdownWrapper>
            </DivVelogWrapper>
        </>
    );
}

export default Study;
```
CSS를 구성하는데 애를 먹었지만 성공적으로 완성하였다~~(Gpt는 신이야)~~.

---

### 최종 결과

<img width="1912" height="912" alt="Image" src="https://github.com/user-attachments/assets/309808b3-2c97-4ede-9f2c-32099d02cb03" />

태그로 글을 분류하고, 클릭하면 오른쪽에서 글을 바로 볼 수 있는 UI가 완성되었다.

---

### 마무리

Velog의 글을 RSS를 이용해 성공적으로 연동하였다.
다음에는 **GitHub API**를 활용해서 내 GitHub 활동 정보를 보여주는 페이지를 구현할 예정이다.


