[Velog로 가기](https://velog.io/@choi-hyk/GitHub-Pages-백엔드-서버-구축하기)

released at 2025-07-27 17:10:48 KST

updated at 2025-08-15 13:33:29 KST

|[github pages](https://velog.io/tags/github-pages)||----|

## 🛠️ 백엔드 서버 구축하기

정적 페이지만 서비스하는 GitHub Pages를 확장하기 위해 백엔드 서버를 구축하려고 한다.

서버는 **Node.js**를 사용할 예정이다. AWS 프리 티어를 사용했는데, 이상하게 **VS Code Remote SSH**로 EC2 서버에 접속을 시도하면 처음 30초 동안은 접속이 되다가, EC2 서버 자체가 먹통이 되는 오류가 생겼다. 예전에 AWS 실습을 진행할 때는 이러한 문제가 없었는데, 갑자기 생겨서 당황했다.
일단 서버는 로컬에서 구현한 후 나중에 이미지로 EC2에 올리는 방식으로 진행했다.

또한 **DB는 MySQL을 Docker**로 구동할 예정이다.

---

### Node.js 구현

간단하게 구현하기 위해 Node.js를 사용하였다.

#### `app.ts`

```js
import express from "express";
import cors from "cors";
import pool from "./db";

const app = express();
const PORT = 3000;

app.use(cors());
app.use(express.json());

app.get("/ping", (req, res) => {
    res.json({ message: "pong", time: new Date() });
});

app.get("/events", async (req, res) => {
    const [rows] = await pool.query("SELECT id, title, start, end FROM events");
    res.json(rows);
});

app.post("/events", async (req, res) => {
    const { title, start, end } = req.body;
    await pool.query(
        "INSERT INTO events (title, start, end) VALUES (?, ?, ?)",
        [title, start, end]
    );
    res.status(201).json({ message: "created" });
});

app.listen(PORT, () => {
    console.log(`API Server running on http://localhost:${PORT}`);
});
```

보시면 알겠지만 일단 `app.ts`에 라우터들을 전부 넣었다. 간단하게 ping을 확인하는 라우터부터, 일정을 조회하고 등록하는 라우터를 구현하였다.

---

### MySQL

Docker를 너무 오랜만에 사용해서 꺼둔 상태였는데, 다시 Docker Desktop을 실행하니 초기화면에서 멈추었다.
그래서 Docker를 삭제했다가 재부팅하는 등 여러 방법을 시도한 끝에 정상 실행이 되었다.

어쨌든 MySQL 이미지를 실행시키고 `schedule_db`라는 데이터베이스에 `events` 테이블을 생성해 테스트 일정을 몇 개 등록하였다.

<img width="565" height="932" alt="Image" src="https://github.com/user-attachments/assets/03e90321-10e4-4984-94f6-286481700238" />

---

### HTTPS 이슈

이제 배포된 GitHub Pages 상에서 **퍼블릭 IP**로 API 요청을 보내면 일정이 캘린더에 출력될 것이다.
그런데 여기서 생각하지 못한 문제가 생겼다. 바로 **GitHub Pages는 HTTPS 트래픽만 사용**한다는 점이다.
사실 당연한 것이지만 구현 중에는 이를 잊고 있었다.

---

#### Nginx

HTTPS를 서비스하기 위해서는 **Nginx를 통해 외부 HTTPS 트래픽을 받아 내부 서버로 HTTP 트래픽으로 전달**해야 한다.

```
[Client]  <-- HTTPS -->  [Nginx/로드밸런서]  <-- HTTP -->  [내부 서버]
```

이 구조를 **Reverse Proxy(역방향 프록시)** 라고 하며,
이 과정에서 **SSL/TLS Termination(SSL 종료)** 가 이루어진다.
즉, 외부(클라이언트 ↔ Nginx)에서는 SSL/TLS를 통한 암호화/복호화를 처리하고,
내부 서버에서는 HTTP로 가벼운 트래픽만 처리할 수 있도록 한다.

---

#### 개인 DNS

또한 HTTPS를 사용하려면 **DNS 등록이 필요**하다.
HTTPS에서 사용하는 **TLS 인증서는 도메인 기반**이기 때문에, 브라우저는 **트래픽을 요청하는 도메인과 인증서에 등록된 도메인이 일치하는지** 확인한다.
CA(인증기관)는 인증서 발급 과정에서 **DNS를 기반으로 소유권 검증**을 한다.

나는 DNS를 AWS의 **Route 53** 서비스를 사용해서 등록하였다.

<img width="1503" height="242" alt="Image" src="https://github.com/user-attachments/assets/5b97be57-78a8-4ff5-94c4-856da59a75ee" />

앞에 레코드 `api`를 설정해 최종적으로 `api.choi-hyk.com` 이라는 DNS를 사용하기로 했다.

---

#### Certbot

이번에 DNS를 등록하며 찾아보니, **Certbot**이라는 도구를 사용하면 **Nginx에 도메인 등록과 SSL/TLS 인증서 발급·갱신을 자동화**할 수 있었다.

* 주요 기능

  * 도메인 검증 (HTTP-01 또는 DNS-01)
  * 인증서 발급
  * Nginx 설정 자동 변경 (SSL 적용)
  * 자동 갱신 (cron 또는 systemd 타이머를 통한 60\~90일 주기)

```bash
sudo certbot --nginx -d api.choi-hyk.com
```

위 명령어를 실행하면, 인증기관이 `api.choi-hyk.com`에 등록된 IP로 HTTP 요청을 보내 **도메인 소유권을 확인**한다.
인증이 통과되면 인증서를 발급하고 Nginx 설정에 자동 반영한다.

```
[Let's Encrypt CA]
      |
      |  (HTTP로 접속)
      v
http://api.example.com/.well-known/acme-challenge/토큰
      |
      v
 [Route 53 DNS] → 3.35.30.190
      |
      v
 [EC2 (Nginx + Certbot)]
      |
      v
 토큰 응답 → 인증 완료 → 인증서 발급
```

---

#### 연동 확인

<img width="770" height="129" alt="Image" src="https://github.com/user-attachments/assets/835fcce5-d46d-4cbe-8f17-5e86658e312b" />

`https://api.choi-hyk.com/events`로 접속하면 정상적으로 JSON이 출력되는 것을 볼 수 있다.

<img width="1067" height="623" alt="Image" src="https://github.com/user-attachments/assets/e2e6306d-0588-42a3-a9be-4f6f1bf4ac77" />

GitHub Pages에서도 JSON 데이터가 정상적으로 출력되는 것을 확인했다.

---

### 마무리

이렇게 기존에 생각했던 것보다 복잡했지만, **GitHub Pages와 백엔드 서버 연동을 완료**했다.
다음 단계는 클라이언트에서 일정을 등록하고, 세부 내용을 출력하는 기능을 구현하는 것이다.
