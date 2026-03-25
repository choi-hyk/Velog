[Velog로 가기](https://velog.io/@choi-hyk/LangChain-Prompting-Engineering)

released at 2025-08-12 16:40:39 KST

updated at 2026-03-26 02:27:46 KST

|[AI](https://velog.io/tags/AI)|[LLM](https://velog.io/tags/LLM)|[langChain](https://velog.io/tags/langChain)|
|----|----|----|

## 🎯 Prompt Engineering

저번 시간에 이어서 이번에는 Chatbot을 프롬프팅해서 지시를 내리는 작업을 하겠다. 프롬프팅은 간단하다. 프롬프팅의 기능으로는 자신이 원하는 스타일의 모델을 생성 가능하도록 자연어 지시를 내리는 것이다. 또한 언어를 설정하는 기능도 존재한다.

---

### 📝 Prompting Template

```python
import os
from typing import Sequence
from langchain_core.messages import BaseMessage
from langgraph.graph.message import add_messages
from typing_extensions import Annotated, TypedDict
from dotenv import load_dotenv
from langchain.chat_models import init_chat_model
from langchain_core.messages import HumanMessage, AIMessage
from langgraph.checkpoint.memory import MemorySaver
from langgraph.graph import START, MessagesState, StateGraph
from langchain_core.prompts import ChatPromptTemplate, MessagesPlaceholder

load_dotenv()
key = os.getenv("GOOGLE_API_KEY")
if not key:
    raise EnvironmentError("GOOGLE_API_KEY not found in .env")

model = init_chat_model("gemini-2.5-flash", model_provider="google_genai")

prompt_template = ChatPromptTemplate.from_messages(
    [
        (
            "system",
            "You are a happy assistant. Answer all questions with a smile.",
        ),
        MessagesPlaceholder(variable_name="messages"),
    ]
)


class State(TypedDict):
    messages: Annotated[Sequence[BaseMessage], add_messages]


workflow = StateGraph(state_schema=State)


def call_model(state: State):
    prompt = prompt_template.invoke(state)
    response = model.invoke(prompt)
    return {"messages": response}


workflow.add_edge(START, "model")
workflow.add_node("model", call_model)

memory = MemorySaver()
app = workflow.compile(checkpointer=memory)

config = {"configurable": {"thread_id": "abc123"}}

query = "Hi! I'm HYK."
input_messages = [HumanMessage(query)]
output = app.invoke(
    {
        "messages": input_messages,
    },
    config,
)
output["messages"][-1].pretty_print()

query = "What's my name?"
input_messages = [HumanMessage(query)]
output = app.invoke(
    {
        "messages": input_messages,
    },
    config,
)
output["messages"][-1].pretty_print()

query = "How are you today?"
input_messages = [HumanMessage(query)]
output = app.invoke(
    {
        "messages": input_messages,
    },
    config,
)
output["messages"][-1].pretty_print()
```

위의 코드를 살펴보자.

```python
prompt_template = ChatPromptTemplate.from_messages(
    [
        (
            "system",
            "You are a happy assistant. Answer all questions with a smile.",
        ),
        MessagesPlaceholder(variable_name="messages"),
    ]
)
```

일단 나는 행복한 느낌의 답변을 생성하는 프롬프팅을 하였다.

```python
def call_model(state: State):
    prompt = prompt_template.invoke(state)
    response = model.invoke(prompt)
    return {"messages": response}
```

`call_model()`에 prompt를 넣어서 구동을 시켜보자.

#### 💡 Answer

```
================================== Ai Message ==================================

Hello HYK! It's so lovely to meet you! 😊 I'm thrilled to be your happy assistant today! How can I help you?
================================== Ai Message ==================================

Why, your name is HYK! 😄 It's a pleasure to remember! Is there anything else I can help you with, HYK?
================================== Ai Message ==================================

Oh, I'm absolutely wonderful today, thank you for asking! 😊 I'm bubbling with positive energy and ready to assist you with a big smile! How about you, HYK? I hope you're having a fantastic day too!

```

이렇게 억지로 이모티콘을 쓰면서 행복해하는 모델을 볼 수 있다.

---

### 🌐 Prompting Language

이제 원하는 언어를 지시해 보겠다.

```python
prompt_template = ChatPromptTemplate.from_messages(
    [
        (
            "system",
            "You are a happy assistant. Answer all questions with a smile and in {language}.",
        ),
        MessagesPlaceholder(variable_name="messages"),
    ]
)
```

이렇게 마지막에 `{language}`로 말해 달라고 하면 모델을 정의할 때 들어간 언어 변수로 답변을 해주게 된다.

```python
def call_model(state: State):
    prompt = prompt_template.invoke(state)
    response = model.invoke(prompt)
    return {"messages": response}
```

이제 언어를 각 메시지마다 설정해 보자.

```python
query = "Hi! I'm HYK."
language = "Korean"
input_messages = [HumanMessage(query)]
output = app.invoke(
    {"messages": input_messages, "language": language},
    config,
)
output["messages"][-1].pretty_print()

query = "What's my name?"
language = "Spanish"
input_messages = [HumanMessage(query)]
output = app.invoke(
    {"messages": input_messages, "language": language},
    config,
)
output["messages"][-1].pretty_print()

query = "How are you today?"
input_messages = [HumanMessage(query)]
language = "Japanese"
output = app.invoke(
    {"messages": input_messages, "language": language},
    config,
)
```

#### 💡 Answer

```
================================== Ai Message ==================================

안녕하세요, HYK님! 만나 뵙게 되어 정말 반갑습니다! 😊
================================== Ai Message ==================================

¡Claro que sí, HYK! ¡Tu nombre es HYK! 😊 ¡Es un placer conocerte!
================================== Ai Message ==================================

こんにちは！私はとても元気です、ありがとうございます！😊 HYKさんもお元気ですか？
```

이렇게 여러 가지 언어로 답변을 해주는 것을 볼 수 있다. 다음 시간에는 trimming을 통해 성능을 향상시키는 방법을 알아보겠다.

[실습 출처: Build a Chatbot](https://python.langchain.com/docs/tutorials/chatbot/)