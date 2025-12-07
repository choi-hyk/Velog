[Velog로 가기](https://velog.io/@choi-hyk/GitHub-Pages-GitHub-Page-구현하기-with-Cursor-AI)

released at 2025-08-04 22:11:10 KST

updated at 2025-12-07 06:30:09 KST

|[AI](https://velog.io/tags/AI)|[github pages](https://velog.io/tags/github-pages)|
|----|----|

## 🛠️ GitHub Page 구현하기 with Cursor AI

이번 시간에는 최종적으로 GitHub Page 구현을 마치려고 한다. 그런데 역시 끈기가 부족해서 구현하는 게 상당히 귀찮게 느껴졌다. 특히 CSS로 렌더링을 구현하는 게 짜증이 났다. 그래서 이번에는 온전히 Cursor AI를 통해 프로젝트를 분석하고 적절한 코드를 생성하려고 한다.

---

### Vibe Coding

올해 가장 핫한 개발 트렌드는 **Vibe Coding**일 것이다. 학부생 수준의 개발자들은 AI를 적극적으로 활용해 개발하겠지만, 사용하면서 양심의 가책을 느낄 수도 있다(나만 그런가?). 현직 개발자들의 이야기를 들어보면, 이제 AI를 적극 활용해 업무 효율을 극대화하는 것이 매우 중요하다고 한다. 물론 나도 인턴십 경험이 겨우 2개월이라 실질적인 현업 경험은 부족하지만, 이번에 Cursor AI로 Vibe Coding과 유사한 작업물을 만들어 보려고 한다.

---

### Cursor AI

나는 주로 **VS Code IDE**를 사용한다. Cursor AI는 완전히 VS Code 위에서 작동하는 AI 통합 개발 환경으로, LLM(대규모 언어 모델, Large Language Model)을 탑재했다. VS Code의 Copilot과 달리 전체 프로젝트를 분석해 더 수준 높은 코드를 생성할 수 있다.

Cursor AI는 다음과 같은 모델을 지원한다:

* **OpenAI**

  * o3-pro (GPT-3.5 Pro)
  * GPT-4.1 (o4)
  * GPT-4 Turbo
* **Google**

  * Gemini 2.5 Pro
* **Anthropic**

  * Claude Sonnet 4
  * Claude Opus 4
* **xAI**

  * Grok 3 Beta
* **DeepSeek**

  * DeepSeek V3.1
* **Cursor 자체 모델**

  * cursor-small (경량화 버전)

사용자는 자신이 보유한 API 키로 원하는 모델을 지정할 수 있다. 나는 무료 플랜에서 **GPT-3.5** 모델을 사용했다. Cursor AI의 강점은 전체 프로젝트 단위에서 자연어 프롬프트만으로 원하는 결과물을 얻을 수 있다는 점이다.

---

### 구현 과정

![Cursor AI 프롬프트 예시](https://github.com/user-attachments/assets/1e40c8b7-6427-48db-abae-ba7a3e6d9adb)

위와 같이 대략적인 프롬프트만 작성해도, 숙련된 사용자가 아니어도 매우 자연스럽고 정확한 코드를 생성해 준다. 나는 이미 구현해 둔 `Velog.tsx`의 스타일과 컴포넌트를 기준으로 요청하자, 거의 동일한 스타일로 코드를 받아낼 수 있었다.

---

### 최종 결과

![최종 GitHub Page 화면](https://github.com/user-attachments/assets/da280393-2c6f-4bed-8bb2-fb21b6ad0622)

프로젝트 내 전역 색상, 보더, 컴포넌트 스타일을 유사하게 구현한 결과를 확인할 수 있다. 앞으로는 이런 CSS 디자인 작업을 Cursor AI에 전적으로 맡길 생각이다.

다음 시간에는 GitHub Pages 중간 점검을 해 보고 잠시 쉬어갈 예정이다. 이후에는 **GitHub Codespaces** 서버와 개인 톡비서 같은 AI 챗봇 시스템을 구현할 계획이다. 그다음 프로젝트로는 AI와 **Godot 엔진**을 결합한 간단한 게임 개발을 생각 중이며, 틈틈이 AI 공부도 진행해 정리할 예정이다.
