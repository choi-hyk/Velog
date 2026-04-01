[Velog로 가기](https://velog.io/@choi-hyk/LangChain-Chatbot으로-LangChain-시작하기)

released at 2025-08-12 12:36:42 KST

updated at 2026-04-01 08:26:51 KST

|[AI](https://velog.io/tags/AI)|[LLM](https://velog.io/tags/LLM)|[langChain](https://velog.io/tags/langChain)|
|----|----|----|

## 🚀 LangChain 시작하기

LangChain은 요즘 핫한 AI 애플리케이션 개발을 도와주는 오픈소스 라이브러리이다. 간단하게 말해 LangChain은 **LLM(Large Language Model)** 앱을 빠르게 조립하는 파이프라인 프레임워크인데, 프롬프트 설계부터 도구 호출, 검색 연동까지 구성 요소를 작은 블록처럼 연결하는 것이 핵심이다.

<img width="643" height="432" alt="Image" src="https://github.com/user-attachments/assets/0336d833-4f52-4a0e-a62d-ef1d7fcc6acc" />

위 그림은 LangChain에서 사용하는 도구들을 나눈 것이라 생각하면 된다.
한번 역할을 간단하게 살펴보겠다.

코드 환경에서는 **LangChain을 통해 개발**을 진행하게 된다. 그리고 LangGraph를 통해 HIP(Human In the Loop)이라는 방법으로 고도화 작업이 가능하다고 하는데,
이 부분은 나중에 알아보도록 하자.

또한 **LangSmith를 통해 품질 모니터링과 테스트** 같은 활동이 가능하다.

마지막으로 **LangGraph Platform은 실제 제품화를 위한 API 추출과 Assistant화**를 도와준다고 한다.

나는 일단 LangChain을 통해 코드 환경에서 간단한 실습을 진행하고, 프로젝트화를 통해 기능을 넣을 생각이다.

---

### 💬 Chatbot 만들기

이제 Chatbot 실습을 진행하겠다.

```txt
langchain
langchain-core
langgraph>0.2.27
dotenv
```

먼저 필요한 패키지다.

```bash
pip install -qU "langchain[google-genai]"
```

이 명령은 google-genai를 사용하게 해주는 LangChain 키트를 설치하는 명령어다.
이 명령어는 꼭 별도로 설치해야 한다. 그렇지 않으면 의존성 오류가 발생한다.

```python
import os
from dotenv import load_dotenv

load_dotenv()

key = os.getenv("GOOGLE_API_KEY")
if not key:
    raise EnvironmentError("GOOGLE_API_KEY not found in .env")

from langchain.chat_models import init_chat_model
from langchain_core.messages import HumanMessage, AIMessage

model = init_chat_model("gemini-2.5-flash", model_provider="google_genai")

resp = model.invoke(
    [
        HumanMessage(content="Hello, my name is choihyeok"),
        AIMessage(content="Hello choihyeok! How can I assist you today?"),
        HumanMessage(content="What's my name?"),
    ]
)
print(resp.content)
```

위 코드를 보면 `from langchain.chat_models import init_chat_model`을 통해 Chatbot 모델 기능을 사용 가능하다. 여기에 API 키를 넣어 사용하면 되며, 여기서는 Google-Gemini 2.25-flash 모델을 사용했다.

`invoke`를 통해 모델에 메시지를 삽입하고, `HumanMessage`와 `AIMessage`로 사람과 AI의 대화를 구분한다. 마지막에 `resp.content`를 출력하면 텍스트 응답만 확인할 수 있다.

#### 📌 실행 예시

```bash
Your name is **choihyeok**.
```

---

그런데 우리가 원하는 것은 **프로세스 환경에서 실시간 대화**다.

```python
from langgraph.checkpoint.memory import MemorySaver
from langgraph.graph import START, MessagesState, StateGraph

workflow = StateGraph(state_schema=MessagesState)

def call_model(state: MessagesState):
    response = model.invoke(state["messages"])
    return {"messages": response}

workflow.add_edge(START, "model")
workflow.add_node("model", call_model)

memory = MemorySaver()
app = workflow.compile(checkpointer=memory)

config = {"configurable": {"thread_id": "abc123"}}

query = "Hi! I'm HYK."
input_messages = [HumanMessage(query)]
output = app.invoke({"messages": input_messages}, config)
output["messages"][-1].pretty_print()

query = "What's my name?"
input_messages = [HumanMessage(query)]
output = app.invoke({"messages": input_messages}, config)
output["messages"][-1].pretty_print()
```

`call_model` 메서드를 선언해 state를 설정하고, LangGraph의 State 기능을 통해 대화 상태를 기억하게 한다.

여기서 LangGraph의 강력한 기능을 알 수 있는데, 바로 State로 워크플로를 관리하는 것이다. LangGraph는 하나의 프로세스를 정의해서 해당 모델이 어떠한 상태를 가지고 있는지 정의한다.

```python
workflow = StateGraph(state_schema=MessagesState)
```

해당 코드가 핵심인데, `StateGraph`를 `MessagesState`로 정의하는 워크프로를 생성한다는 의미이다. 이제 모델은 해당 워크플로안에서 유지되면서 메세지를 받게된다.

```python
workflow.add_edge(START, "model")
workflow.add_node("model", call_model)
memory = MemorySaver()
app = workflow.compile(checkpointer=memory)
```

워크플로는 노드와 엣지 형태로 모델의 시작점과 진행 사항을 연결 및 유지 해준다.

위와 같이 구현하면 모델은 대화를 메모리에 저장하며 진행한다. 다만 현재는 프로세스를 종료하면 쓰레드가 정리되어 기록이 사라진다.

```python
config = {"configurable": {"thread_id": "abc123"}}
```

이 코드로 쓰레드를 설정해 대화 세션을 구분할 수 있다.

#### 📌 실행 예시

```
================================== Ai Message ==================================
Hi HYK! It's nice to meet you.
How can I help you today?

================================== Ai Message ==================================
Your name is HYK! You told me that when you first introduced yourself.

================================== Ai Message ==================================
I don't know your name. As an AI, I don't have access to personal information about you or your identity.
How can I help you today?

================================== Ai Message ==================================
Your name is HYK! I remember you told me that when we first started chatting.
```

위 출력에서 보듯, 첫 번째 메시지에서 이름을 알려줬지만 세번째 쓰레드에서는 이름을 모른다.

기본적인 구현은 위와 같으며, 다음에는 **프롬프트 엔지니어링**을 알아보겠다.

[[참고] LangChain Build a Chatbot](https://python.langchain.com/docs/tutorials/chatbot/)

