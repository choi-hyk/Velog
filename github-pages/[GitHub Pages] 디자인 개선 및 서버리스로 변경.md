[Velog로 가기](https://velog.io/@choi-hyk/GitHub-Pages-디자인-개선-및-서버리스로-변경)

released at 2025-12-28 17:28:39 KST

updated at 2026-01-14 23:11:33 KST

|[github pages](https://velog.io/tags/github-pages)|
|----|

# 🖥️ Github Pages 리펙토링

아주 오랜만에 **Github Pages** 글을 작성하는 것 같다. 이번 글에는 최근에 내가 진행한 GitHub Pages 디자인 개선과 서버리스 구조로 변경한 작업을 정리해보려고 한다.

## 🛠️ Serverless...

원래는 백엔드를 하나 두어서, 일정과, Velog 데이터 및 Github 데이터를 가져오도록 구성을 해두었다. 그런데  AWS 프리티어로 구동 중이다 보니 관리도 너무 힘들고, 서버가 먹통이 되거나 이런 현상이 많았다. 그래서 과감하게 일정 페이지를 삭제하고 단순히 렌더링 타임에 Velog 및 Github 데이터를 호출하도록 바꾸었다. 구매해둔 DNS는 다른 프로젝트에서 사용할 예정이다.


## 📌 Velog 데이터 가져오기

GitHub 데이터는 단순히 내 계정의 정보와 public repo 정보를 가져오면 되므로, API를 호출하면 된다. 그런데 문제는 Velog 데이터를 가져오는 것이었다. Velog는 RSS로 가져오거나, GraphQL로 가져오는 방식 두개가 있는데, 상세한 정보를 가져오고 싶으면 GraphQL을 가져와야 한다. 나는 글의 태그들도 가져오기를 원해서 GraphQL 방식을 채택하였다. **그런데 Velog의 GraphQL은 브라우저 호출을 허용하지 않기에 다른 방법을 찾아야 했다.**
 
나는 예전에 구성한 [VelogSync](https://github.com/choi-hyk/velog_sync) 프로젝트로 내 Github repo에 Velog 글들을 자동 백업을 하고 있다.

관련글
- [[Mini Project] Velog Backup 프로그램 만들기](https://velog.io/@choi-hyk/Mini-Project-Velog-Backup-%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%A8-%EB%A7%8C%EB%93%A4%EA%B8%B0)

따라서 해당 repo의 정보를 가져오는 방식으로 구성을 하였다. 간단하게 해결이되어서 다행이지만, 일단 시간당 60회 호출 제한이 있긴 해서, 완전하지는 않다. 이 부분은 나중에 개선하기로 하겠다.

## 디자인 개선

디자인 개선은 Codex CLI의 힘을 빌러서 진행했다. Codex CLI에 최근 트렌드에 맞춰서 디자인 개선을 진행하도록 프롬프팅을 하고 작업을 하였다. 


### Profile Page
<img width="1920" height="1080" alt="Image" src="https://github.com/user-attachments/assets/0ae752ff-b615-45a4-8497-e3dfa2d3bdc4" />

### Github Page
<img width="1920" height="1080" alt="Image" src="https://github.com/user-attachments/assets/3610afc2-e17a-426e-bf2b-1898164f4509" />

### Velog Page
<img width="1920" height="1080" alt="Image" src="https://github.com/user-attachments/assets/fe236b1f-be49-4568-a3c7-9c9670c965cd" />
<img width="1920" height="1080" alt="Image" src="https://github.com/user-attachments/assets/17fa2bfb-e8e2-4326-9959-30cb2c141abc" />

**Codex CLI**를 써보니 나같은 디자인 잼병이나 프론트에 익숙하지 않은 주니어 개발자 혹은 백엔드 개발자한테 매우 좋은 툴인 것 같다. 백엔드를 구현할 때도 사용을 하지만, 프론트를 구현할 때 강력한 기능을 제공하는 것 같다. 로그인 페이지나 이번에 구성한 프로필 혹은 기타 카드로 이루어진 매우 심플한 페이지를 구성할 때, 시간 단축과 세련된 디자인을 제시해준다. **Codex CLI**에 대해서는 따로 글로 작성해서 MCP 도구 세팅과 유용한 사용법을 정리할 생각이다.

## 😁 마무리

이렇게 서버리스로 바꾸고 디자인 개선을 진행해보니, 한결 마음이 편해진 것 같다. AWS도 빨리 정리해서 지금 진행중인 HippoBox 프로젝트에서 해당 DNS와 서버를 사용하도록 바꿔야 겠다.


**Github Page**
[choi-hyk.github.io](https://choi-hyk.github.io./#/profile)



