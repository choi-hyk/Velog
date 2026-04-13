[Velog로 가기](https://velog.io/@choi-hyk/GitHub-Pages-deploy-구성하기)

released at 2025-07-28 20:13:34 KST

updated at 2026-04-13 09:25:20 KST

|[github pages](https://velog.io/tags/github-pages)|
|----|

## 🚀 deploy 구성하기

GitHub Pages에 배포하는 방식은 여러 가지가 있는데, 먼저 로컬에서 빌드된 결과물을 직접 명령어로 배포하는 방법이 있다.

```bash
npm run build
```

```bash
npm run deploy
```

```json
    "scripts": {
        "dev": "vite",
        "build": "vite build",
        "lint": "eslint .",
        "preview": "vite preview",
        "predeploy": "npm run build",
        "deploy": "gh-pages -d dist"
    },
```

이렇게 scripts를 설정하고 위의 명령어를 실행하면 빌드된 `dist` 폴더를 `github.io` 레포지토리의 `gh-pages` 브랜치로 푸시할 수 있다.
그러면 GitHub에서 자동으로 배포 전용 `gh-pages` 브랜치를 감지해 해당 dist 파일을 기준으로 빌드 및 배포를 진행한다.

나 역시 이 방법을 사용하고 있었는데, 현재 백엔드 서버를 EC2 위에서 운영하고 있어서 **이제는 이 프로젝트도 배포를 자동화하고 싶다**는 생각이 들었다.

---

### 🖥️ Node.js PM2 배포

현재 EC2에서는 MySQL을 Docker 컨테이너로 실행하고 있다. 서버도 Docker 이미지를 만들어 컨테이너로 배포하는 방법이 있지만, 이번에는 다른 접근을 해보고 싶었다.

그래서 **PM2(Process Manager 2)** 를 사용하여 Node.js 서버를 EC2에서 배포하고 관리하도록 구성했다. PM2를 사용하면 백엔드 프로세스를 **데몬 프로세스** 형태로 구동할 수 있다.

```bash
echo "[1] 로컬 빌드 시작"
npx tsc

echo "[2] dist, package.json 전송"
scp -o StrictHostKeyChecking=no -i $KEY_PATH -r dist $SERVER_USER@$SERVER_HOST:$SERVER_PATH
scp -o StrictHostKeyChecking=no -i $KEY_PATH package*.json $SERVER_USER@$SERVER_HOST:$SERVER_PATH

echo "[3] 서버 내 명령 실행"
ssh -tt -o StrictHostKeyChecking=no -i $KEY_PATH $SERVER_USER@$SERVER_HOST << EOF
  set -e
  cd $SERVER_PATH
  npm ci --omit=dev
  pm2 restart choi-hyk.github.io-api || pm2 start dist/app.js --name choi-hyk.github.io-api
EOF
```

위 스크립트를 보면 `scp` 명령어로 빌드된 dist 파일과 package.json 파일을 서버로 전송한 뒤,

```bash
pm2 restart choi-hyk.github.io-api || pm2 start dist/app.js --name choi-hyk.github.io-api
```

이 명령어로 구동 중인 pm2 앱을 재시작하거나, 없으면 새로 시작한다.
하지만 이렇게 하면 **매번 로컬에서 직접 실행해야 하는 번거로움**이 생긴다. 그래서 **GitHub Actions를 이용해 main 브랜치에 push될 때 자동으로 배포되는 스크립트**를 구성하게 되었다.

---

### ⚙️ GitHub Actions deploy

GitHub에서는 EC2 배포 템플릿을 제공해 쉽게 CI/CD 환경을 구성할 수 있다.

<img width="898" height="229" alt="Image" src="https://github.com/user-attachments/assets/fc1c6873-bcf3-4920-909d-fb40e20c74cb" />

이처럼 `deploy`라는 **환경(Environment)** 을 설정해주면 로컬 환경 변수처럼 사용할 수 있다.

```yml
name: Deploy to EC2

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: deploy 

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 20

      - name: Install dependencies
        run: npm ci

      - name: Build TypeScript
        run: npx tsc

      - name: Print valuables for debugging
        run: |
          echo "EC2_HOST is: ${{ vars.EC2_HOST }}"
          echo "EC2_USER is: ${{ vars.EC2_USER }}"
      - name: Configure SSH
        run: |
          mkdir -p ~/.ssh
          echo "${{ secrets.EC2_KEY }}" > ~/.ssh/id_rsa
          chmod 600 ~/.ssh/id_rsa
          ssh-keyscan -H ${{ vars.EC2_HOST }} >> ~/.ssh/known_hosts
      - name: Deploy to EC2
        run: |
          rsync -avz --delete dist package*.json ${{ vars.EC2_USER }}@${{ vars.EC2_HOST }}:/home/ubuntu/choi-hyk.github.io/choi-hyk.github.io.server/
          ssh ${{ vars.EC2_USER }}@${{ vars.EC2_HOST }} "
            cd /home/ubuntu/choi-hyk.github.io/choi-hyk.github.io.server &&
            npm ci --omit=dev &&
            pm2 restart choi-hyk.github.io-api || pm2 start dist/app.js --name choi-hyk.github.io-api
          "
```

처음에는 복잡해 보이지만 동작 원리를 이해하면 간단하다.
`name`과 `on`은 Actions의 이름과 트리거(언제 실행할지)를 정하는 부분이고,

```yml
deploy:
    runs-on: ubuntu-latest
    environment: deploy 
```

이 부분은 해당 Job이 실행될 환경을 지정한다. 특히 **`environment: deploy`** 로 지정해주면 GitHub의 Environment 탭에 설정한 **secrets**와 **variables** 값을 그대로 사용할 수 있다.
`secrets`는 `secrets.XXX`, `variables`는 `vars.XXX` 로 참조 가능하다.

이후 steps에서는 다음 순서로 진행된다.

1. **Checkout**
   `actions/checkout` 으로 main 브랜치 코드를 가져온다.
2. **Node.js 세팅**
   `actions/setup-node` 로 Node.js 버전을 맞춘다.
3. **의존성 설치 및 빌드**
   `npm ci`, `npx tsc` 로 설치와 빌드를 수행한다.
4. **SSH 접속 설정**
   secrets에 저장한 키(`EC2_KEY`)를 \~/.ssh/id\_rsa로 저장하고 권한을 조정한다.
5. **EC2로 전송 및 pm2 재시작**
   `rsync`로 dist와 package.json을 서버로 전송한 뒤, pm2를 재시작하거나 처음 실행한다.


이제 main 브랜치에 push만 하면 자동으로 **EC2에 새로운 빌드가 배포**된다.
수동으로 scp와 pm2를 실행하던 불편함을 없앨 수 있게 된 것이다.

---

### ✅ 마무리

이번에 GitHub Actions를 적용하면서 **프론트엔드는 GitHub Pages**, **백엔드는 EC2 + PM2**로 각각 자동화 배포되는 구조를 만들 수 있었다.
이제는 main 브랜치에 코드를 푸시하는 것만으로도 배포가 완료된다.

앞으로 테스트, 코드 스타일 체크, Docker 이미지 빌드 같은 단계도 추가하여 **점점 더 정교한 CI/CD 파이프라인**으로 확장해볼 예정이다.

