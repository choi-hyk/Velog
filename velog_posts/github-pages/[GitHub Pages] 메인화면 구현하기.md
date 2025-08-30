[Velog로 가기](https://velog.io/@choi-hyk/GitHub-Pages-메인화면-구현하기)
released at 2025-07-20 19:49:36 KST
updated at 2025-08-15 13:29:26 KST

|[github pages](https://velog.io/tags/github-pages)|
|----|

## 🖥️GitHub Pages 메인화면 구현

github pages의 웹사이트를 어떤식으로 구현을 할까 고민을 해보았는데, 간단하게 내 프로필과 일정 그리고 공부 내용을 정리 할수 있는 공간을 만들어 보기로 하였다. 따라서 구현해야 되는 페이지는 아래와 같다.

- 프로필
- 캘린더
- 스터디

---

### 📁HashRouter

정적 페이지만을 서비스하는 GitHub Pages는 BrowserRouter를 사용할 수 없다. 이유는 BrowserRouter는 브라우저가 새로고침하면 다시 서버를 통해 정적 페이지를 요청을 하게 되는데 이때 GitHub Pages에서는 미리 제공된정적 페이지 만을 사용하기 때문에 오류가 발생해서 404 오류가 발생한다.
따라서 HashRouter라는 라우터를 사용해야한다. HashRouter는 `#` 를 기준으로 URL을 라우팅 해준다. 이때 라우팅 되는 페이지는 리렌더링 되는 정적인 파일만을 취급한다. 새로고침할 경우에는 `#` 이전의 URL만이 실제로 재로딩 된다. 

#### HasRouter url

```
https://choi-hyk.github.io./#/
```

HashRouter는 위와 같이 #를 기준으로 # 이후의 URL을 관리한다.
여기서 SEO는 #가 들어간 URL은 적용이 안된다해서 검색엔진을 사용하지 못하는 단점이 있다고 한다.

---

### 🛠️페이지 구성

<img width="1899" height="912" alt="Image" src="https://github.com/user-attachments/assets/67e54a44-1d1b-4d72-a1f3-fc3c1673ad58" />

HashRouter를 통해 일단 SideBar와 NavBar를 구성하고 SideBar에 필요한 페이지 아이콘을 배치하여 이동이 가능하게 하였다. 그리고 NavBar에는 현재의 페이지 URL을 넣어서 현재 어떤 페이지에 위치하는지 알 수 있도록 하였다.

앞으로 프로필과 캘린더 스터디 페이지를 순서대로 구현을 하고 여러가지 기능을 넣을 생각이다.
