[Velog로 가기](https://velog.io/@choi-hyk/LangChain-Managing-Conversation-History)

released at 2025-08-12 17:00:59 KST

updated at 2026-02-13 10:06:14 KST

|[AI](https://velog.io/tags/AI)|[LLM](https://velog.io/tags/LLM)|[langChain](https://velog.io/tags/langChain)|
|----|----|----|

## 🗂️ Managing Conversation History

이번에는 저번에 이어서 대화 맥락을 유지시켜 주는 trimmer 기능에 대해서 알아보겠다. Chatbot은 지금 하는 대화와 이전에 나눈 대화도 기억해서 사용자와 대화해야 한다. 이것을 가능하게 해주는 것이 trimmer 함수이다.

---

### ✂️ trim\_messages

```python
trimmer = trim_messages(
    max_tokens=512,
    strategy="last",
    token_counter=model,
    include_system=True,
    allow_partial=False,
    start_on="human",
)

messages = [
    SystemMessage(content="you're a good assistant"),
    HumanMessage(content="hi! my name is HYK"),
    AIMessage(content="hi! HYK!"),
    HumanMessage(content="My favorite color is blue."),
    AIMessage(content="nice color!"),
    HumanMessage(content="My favorite movie is DarkKnight."),
    AIMessage(content="nice movie!"),
    HumanMessage(content="whats 2 + 2"),
    AIMessage(content="4"),
    HumanMessage(content="thanks"),
    AIMessage(content="no problem!"),
    HumanMessage(content="having fun?"),
    AIMessage(content="yes!"),
]

trimmer.invoke(messages)
```

위의 코드는 메시지 트리머를 정의한 코드이다. 토큰을 충분히 크게 주어서 이전 대화를 최대한 많이 기억할 수 있도록 하였다. 만약 토큰을 적게 할당하면 이전에 나눈 많은 양의 대화를 잊을 것이다. `messages` 변수는 Chatbot에게 메시지를 주입하기 위해 설정한 배열이다.

이를 통해 Chatbot은 해당 대화 내용을 기억하고 있게 된다.

```python
def call_model(state: State):
    trimmed_messages = trimmer.invoke(state["messages"])
    prompt = prompt_template.invoke(
        {"messages": trimmed_messages, "language": state["language"]}
    )
    response = model.invoke(prompt)
    return {"messages": response}
```

이렇게 모델을 정의할 때, `trimmed_messages = trimmer.invoke(state["messages"])`를 삽입해 준다.

```python
query = "What's my name?."
language = "Korean"
input_messages = messages + [HumanMessage(query)]
output = app.invoke(
    {"messages": input_messages, "language": language},
    config,
)
output["messages"][-1].pretty_print()

query = "What's my favorite color?"
language = "English"
input_messages = messages + [HumanMessage(query)]
output = app.invoke(
    {"messages": input_messages, "language": language},
    config,
)
output["messages"][-1].pretty_print()

query = "내가 가장 좋아하는 영화는?"
input_messages = messages + [HumanMessage(query)]
language = "Korean"
output = app.invoke(
    {"messages": input_messages, "language": language},
    config,
)
```

마지막으로 `input_messages = messages + [HumanMessage(query)]`를 통해 정의한 메시지를 주입해 주면 된다.

#### 💬 answer

```
================================== Ai Message ==================================

당신의 이름은 HYK입니다.
================================== Ai Message ==================================

Your favorite color is blue.
================================== Ai Message ==================================

당신이 가장 좋아하는 영화는 다크 나이트입니다.
```

이렇게 trimmer까지 구현을 완료했고, 다음 시간에는 RAG(Retrieval Augmented Generation)의 개념에 대해서 알아보겠다.

[[참고] LangChain Build a Chatbot](https://python.langchain.com/docs/tutorials/chatbot/)