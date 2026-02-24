[Velog로 가기](https://velog.io/@choi-hyk/GitHub-Pages-React로-배포하기)

released at 2025-07-18 19:35:59 KST

updated at 2026-02-24 13:34:32 KST

|[github pages](https://velog.io/tags/github-pages)|
|----|

## GitHub Pages가 페이지를 배포하는 방법

GitHub Pages는 기본적으로 정적 페이지인 html을 배포해준다.index.html이라는 루트 페이지를 진입점으로 사용하며, CSR을 통한 동작이 가능하다. 그렇다는 이야기는 React를 활용해서 간단한 SPA 동작을 지원하는 웹 페이지도 구현 가능하다는 이야기이다. 여기서 고려해야 되는 점은 React는 컴포넌트 형태의 jsx를 취급해야 되므로 깃허브 저장소에 빌드 결과물을 저장해주는 장소를 새로 지정해 줘야 한다.

나는 빌드파일 용 브랜치를 새로 생성해서 배포를 하는 방식을 선택하였다.

---

### 🛠️ 빌드파일 배포 설정

#### 설치
```bash
npm install gh-pages --save-dev
```


먼저 GitHub Pages에 빌드파일을 배포해주는 툴인 gh-pages를 설치한다.

#### package.json에 homepage 설정

```json
"homepage": "https://choi-hyk.github.io"
```
React 패키지 설정 파일인 `package.json`에 homepage 필드에 자신의 GitHub Pages URL을 설정한다.

#### vite 환경설정

```js
import { defineConfig } from "vite";
import react from "@vitejs/plugin-react";

export default defineConfig({
    base: "/",
    plugins: [react()],
});
```

또한 vite를 사용한다면 이렇게 `vite.config.js`에서 base URL위치를 설정해줘야 한다.


#### scripts 추가

```json
"scripts": {
  "predeploy": "npm run build",
  "deploy": "gh-pages -d build"
}
```

이제 배포를 하기 위한 스크립트를 설정한다.

---

### 🖱️배포하기

<img width="1029" height="275" alt="Image" src="https://github.com/user-attachments/assets/6f703458-f885-42ab-8374-d0322733588c" />


배포를 할 경우 GitHub repo에 gh-pages 브랜치가 생성되며 빌드 파일이 안에 저장된다.

<img width="1920" height="1020" alt="Image" src="https://github.com/user-attachments/assets/57e2e157-0202-4a7f-908f-ac4da8e065f6" />

URL에 들어가면 배포가 성공적으로 이루어진 것을 볼 수 있다.
