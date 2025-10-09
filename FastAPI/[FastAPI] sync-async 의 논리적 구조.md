[Velog로 가기](https://velog.io/@choi-hyk/Python-syncasync-의-논리적-구조)

released at 2025-10-09 17:58:43 KST

updated at 2025-10-09 21:21:14 KST

|[FastAPI](https://velog.io/tags/FastAPI)|[Sync/Async](https://velog.io/tags/Sync/Async)|[WAS](https://velog.io/tags/WAS)|[asgi](https://velog.io/tags/asgi)|[concurrency](https://velog.io/tags/concurrency)|[python](https://velog.io/tags/python)|[동기/비동기](https://velog.io/tags/동기/비동기)|[동시성](https://velog.io/tags/동시성)|
|----|----|----|----|----|----|----|----|

# 🖥️ 시작하기에 앞서...
이제 회사에서 본격적으로 개발 일을 시작한지 1달 반이 다되간다. 현재 `Svelte`와 `FastAPI` 기반의 Monolothic 구조의 프로젝트를 유지보수 하고 있는데, 해당 프로젝트에서 이해가 안되는 부분이 매우 많이 있다. 특히 `FastAPI`의 `coroutine`을 통한 라우터 설정에 애를 먹고 있는데, 규모가 큰 오픈소스다 보니까, 어떤 기준으로 해당 함수는 `async`를 통해 코루틴 처리를 하였는지, 어떤 함수는 일반 함수 정의를 통해 `thread pool`로 관리하는지 이해가 안되고 있다. 

아마도 나 같은 신입 개발자들은 이러한 동시성 관리가 익숙치 않을 것이다. 신입 개발자들은 다른 WAS 프레임워크들 또한 이러한 동시성 관리, 더 나아가 `Python` 기반이 아닌 `Spring Boot` 같은 다른 언어 진영의 병렬 처리 같은 물리적 구조를 고려한 프로그래밍을 경험할 기회가 없다. 본인이 개인 프로젝트나, 백엔드 서비스를 개발한 경험이 있어도 실무에서 요구하는 것 과는 분명히 차이가 있을 것이다. 

물론 현재 나는 `FastAPI`를 사용 중이어서, 고 수준의 병렬 처리는 고려할 상황이 아니지만, 개발을 하면서, 성능 개선을 위해 동시성 강화같은 이슈를 처리를 하면 벽이 느껴진다... 심지어 이러한 동시성 관리와 성능 개선을 위해 `ThreadPool`을 사용해 블록킹 함수를 추가를 하면, 이러한 수정이 오히려 전체 프로젝트의 성능에 어떠한 영향을 미칠지 모르겠는 경우가 허다하다. 그래서 이번 시리즈에서는 `ASGI` 기반의 서버인 `FastAPI`가 어떤식으로 동시성을 관리하는지와 다른 서버 프레임워크들과 비교를 통해 어떤 식으로 요청을 받고, 처리하는지 정리를 해보려고 한다. 

---

## 🛠️ Blocking vs Non-Blocking

알아보기 전에 `Blocking`과 `Non-Blocking`에 대해서 자세히 알아볼 필요가 있다. 많은 사람들이 오해를 하는 것이 `Blocking` 함수와 `Non-Blocking` 함수의 구분 방법이다. 두 함수는 개발자가 저수준의 구현을 통해서 `Blocking`, `Non-Blocking`을 설정을 하는 것이 아닌, 기존에 존재하는 라이브러리나 구문을 통해서 설정된다. 또한 기본적인 함수들은 동기 실행을 가정한다. 여기서 가장 오해하는 부분이 동기와 `Blocking` 그리고 비동기와 `Non-Blocking`의 관계이다. 앞에서 이야기한 **"기본적인 함수들은 동기 실행을 가정한다"** 의 의미는 기본적인 함수들은 모두 동기로 처리된다는 의미이다. 당연한 이야기지만, 여기에 `Blocking`과 `Non-Blocking`을 고려해보자.

### CPU bound vs I/O bound

`Blocking`은 **I/O 바운드** 를 통한 **쓰레드의 대기 상태**를 의미한다. 그러면 **I/O 바운드**가 아닌 **CPU 바운드**를 생각해보자 만약 개발자가 루프문을 $O(n^3)$ 의 시간복잡도 동안 실행한다고 해보자. 이 상황에서도 `Blocking` 이라고 할 수 있는가? 루프문을 실행하는 동안 같은 쓰레드 내의 다른 함수들은 실행되지 않지만, **분명 해당 쓰레드는 실행 중**이다. 이러한 쓰레드 내의 작업을 **CPU 바운드**라고 한다. 뭔가 거창하게 설명했지만, 그냥 일반적인 함수 실행이다... 

### 비동기와 Non-Blocking의 논리적 실행

그러면 다시 `Blocking` 함수로 돌아가서 **I/O 바운드**는 정확히 무엇을 의미를 할까? 우리가 서버 환경에서 클라이언트로부터 요청을 받고, 데이터베이스에서 사용자의 정보를 확인한다고 해보자.

```python
def get_user_from_db(username: str):
    conn = sqlite3.connect("users.db")
    cursor = conn.cursor()
    cursor.execute("SELECT username, password FROM users WHERE username = ?", (username,))
    user = cursor.fetchone()
    conn.close()
    return user

@app.post("/login/blocking")
def login_blocking(username: str, password: str):
    user = get_user_from_db(username)
    if not user or user[1] != password:
        raise HTTPException(status_code=401, detail="Invalid credentials")
    return {"message": f"Welcome, {username}!"}
```
위의 함수는 동기 상태로 정의된 `FastAPI` 라우터이다. 위의 `get_user_from_db()`에서 내부 **sqlite db**에서 DML을 실행 중이다. 이때, `cursor.execute("SELECT username, password FROM users WHERE username = ?", (username,))` 는  CPU 바운드인가 I/O 바운드인가? 답은 **I/O 바운드**이다. 해당 함수를 실행을 하면, 현재 쓰레드는 추가적인 작업이 필요한지를 생각해보면, 전혀 아니다. **현재 쓰레드에서는 단순히 해당 함수가 끝나길 기다릴 것이다. 즉 `Waiting` 상태가 된다.** 그리고 **CPU는 해당 DML을 수행하고 있는 DBMS가 점유**할 것이다. 그리고 작업이 끝나면, 함수를 반환하고, 다시 현 쓰레드를 실행할 것이다. 다시 말해, **서버의 스레드는 멈춰 있고, DBMS 프로세스가 디스크에서 데이터를 읽거나 쓰는 작업을 수행**하는 것이다. 작업이 완료되면 DBMS는 결과를 반환하고, 커널은 대기 중이던 쓰레드를 깨워 이전의 함수 실행 지점부터 코드를 이어서 수행한다. 즉, 코드 상으로는 함수가 멈춰 있는 것처럼 보이지만, 실제로는 쓰레드가 CPU를 전혀 사용하지 않고, 외부 자원(디스크)의 응답을 기다리는 **I/O Bound + Blocking** 상황이 발생한 것이다. 그러면 생각해보자 여기서 어떻게 성능을 개선 할 수가 있을까?

위의 코드처럼 라우터에 1개의 요청 또는 1개의 `Blocking` 함수만 있으면 별로 상관이 없을 것이다. 이번에는 `Blocking` 함수가 여러 개가 있다고 생각해보자. 

```python
def get_user_from_db(username: str):
    conn = sqlite3.connect("users.db")
    cursor = conn.cursor()
    time.sleep(1)
    cursor.execute("SELECT username, password, info, history FROM users WHERE username = ?", (username,))
    user = cursor.fetchone()  
    conn.close()
    return user

@app.post("/login/blocking")
def login_blocking(username: str, password: str):
    user = get_user_from_db(username)
    if not user or user[1] != password:
        raise HTTPException(status_code=401, detail="Invalid credentials")
    return {"message": f"Welcome, {username}!"}


@app.post("/info/blocking")
def get_user_info_blocking(username: str):
    user = get_user_from_db(username)
    if not user:
        raise HTTPException(status_code=404, detail="User not found")
    return {"username": user[0], "info": user[2]}


@app.post("/history/blocking")
def get_user_history_blocking(username: str):
    user = get_user_from_db(username)
    if not user:
        raise HTTPException(status_code=404, detail="User not found")
    return {"username": user[0], "history": user[3]}
```
위의 함수에 사용자 3명이 동시다발적으로 3개의 요청을 각각 보낸다고 생각해보자. 

> `FastAPI` 에서는 일반 def 요청은 쓰레드 풀의 개별적인 쓰레드 워커에서 실행된다. 해당 부분은 다음 글에서 자세히 설명을 하고 지금은 단일 쓰레드에서 실행되는 것으로 가정 하겠다.

사용자1이 `login_blocking()`, 사용자2가 `get_user_info_blocking()` 그리고 사용자3이 `get_user_history_blocking()`을 순서대로 요청을 보냈다고 가정하자. 또한 DML 실행시간은 1초라고 가정하자. 그리고 **Task Queue**에는 실행 프로세스의 2개의 쓰레드가 Task로 들어간다고 가정하자. **요청을 받는 메인 쓰레드**와, **요청을 처리하는 워커 쓰레드** 2개로 가정하자. 간단히 생각해보면, 이미 실행 중인 프로세스에 추가적인 작업이 쌓이는 것을 생각하면 된다. 메인 워커 쓰레드를 $W$이라 하겠다. 또한 각 Task Queue에는 쓰레드 단위로 Task가 들어간다고 가정하겠다. 

OS 수준에서는 1개의 프로세스와 1개의 단일 쓰레드를 **Task Queue**에서 실행 중이지만, 프로세스 관점에서는 프로세스에 추가적인 작업이 쌓이고 있다. 사용자1의 `login_blocking()`가 들어오는 순간 현재 쓰레드는 약 1초간 `Waiting` 상태가 될 것이다. `Waiting` 이 되는 1초 동안 나머지 2개의 요청을 받았다고 가정하자. 또한 2개의 코어로 병렬 처리가 된다고 가정하자.

<img width="817" height="749" alt="Image" src="https://github.com/user-attachments/assets/ad66382d-5aaa-421e-a2ef-d9a71f8e1318" />

위의 다이어그램은 Request를 받았을 때, 요청과 단일 쓰레드가 어떻게 처리되는지 보여준다. 현재 Blocking 함수는 `time.sleep()`, `cursor.execute()` 가 존재한다.  또한 $T_n$에서 $n$ 은 초 단위라고 가정을 하겠다. `accept queue`는 생소할텐데, listen 상태의 소켓을 지니고 있는 프로세스가 보유하는 큐로 아직 애플리케이션 레벨에서 `accept()` 호출로 가져가지 않은 연결들이 일시적으로 쌓여 있는 공간이다. 위의 그림을 보면, 당연하게도 $R_2$, $R_3$는 $R_1$이 처리되어야지 순서대로 처리될 것이다. 그러면 약 $T_4$에 모든 처리가 완료 될 것이다. 2개의 코어가 존재해도 $W$ 가 waiting이 되어 있으면, DB process가 실행 중일때 나머지 코어에 $W$를 실행하지 못할 것이다.

그러면 여기서 개선을 어떻게 할까? `FastAPI` 는 ASGI 기반이다. 이 말은 모든 요청을 async 인터페이스로 받는 웹 서버를 가정하는 프레임워크란 뜻이다. 위의 다이어그램을 봤을 때, 비동기로 처리를 한다고 하면, $R_1$ 처리 중에 다른 요청을 처리 할 수 있게 해야 한다. 여기서 사용하는 것이 바로 `async` 함수 내의 `await` 구문이다. 그리고 해당 구문으로 `Non-Blocking`의 진정한 의미를 알 수 있는데, 바로 **실행 제어권을 반납하는 것**이다. 매우 간단하다. `async` 함수 내에서 `await`를 만나면 해당 함수를 `Non-Blocking` 으로 실행하겠다는 의미이다. 그럼 여기서 드는 생각이, 결국에는 `FastAPI` 같은 ASGI 기반의 웹서버는 애플리케이션 수준에서 비동기를 지원하는 것이다. 즉 **단일 쓰레드의 코루틴 내에서 비동기를 지원하여 동시성을 강화하는 것**이다. 위의 다이어그램이 비동기로 처리될때, 차이점은 **I/O 바운드 작업시에 $W$를 Waiting 상태에 빠지지 않도록하는 Non-Blocking 처리**만 존재한다. 그러면 이게 어떻게 가능할까? 단일 쓰레드 내에서도 스케줄러같은 실행 처리를 도와주는 로직이 있는 것일까?

### Coroutine

Coroutine이 바로 이 비동기 처리를 구현하는 기법이다. OS는 Context switch같은 기법을 통해 동시성을 강화한다. 그럼 OS가 비동기를 처리한다고 할 수 있을까? 절대 아니다. 이유는 **동시성 강화는 비동기 처리가 아니기 때문이다. 하지만 비동기 처리는 동시성을 강화하는 기법 중 한가지이다**. OS가 동시성을 강화하는 이유는 여러가지의 작업을 효울적으로 처리하기 위해서다. 이를 통해 사용자는 작업이 동시에 이루어지는 환상을 만들어준다. **하드웨어적인 관점으로 자원을 최소한으로 사용하여 가장 효율적인 스케줄을 통해 프로세스를 관리하는 것** 이것이 목적이다. 그러면 Coroutine을 통한 비동기 처리는 무엇이 목적일까? 말한대로 동시성 강화가 목적이다. 하지만 하드웨어적인 관점에서 굳이 애플리케이션 수준에서 자원의 효율성같은 요소를 신경쓰지는 않을 것이다. 주요 목적은 위에서 말한 것 처럼, **I/O 바운드 작업시에 코루틴 쓰레드를 Waiting 상태에 빠지지 않도록하는 Non-Blocking 처리를 하는 것이다.** 이를 통해 Waiting이라는 요소를 제외하고 실행이 가능하다. 그리고 이것은 **Event Loop를 통해 단일 쓰레드의 Call Stack의 Task switch로 이루어진다.** 이는 OS 수준의 Context switch와 비슷한 기법이다. Event Loop 는 CPU 스케줄러 그리고 Task Switch는 Context switch로 비유할 수 있을 것이다. 

> Coroutine은 OS처럼 물리적인 스케줄링을 수행하는 것이 아난, 단일 스레드 내부에서 실행 흐름을 논리적으로 전환(switch)하여 동시성을 달성하는 방식

>OS의 Context switch가 커널이 직접 개입하여 CPU 레지스터, 프로그램 카운터, 스택 포인터 등 하드웨어 상태를 저장하고 복원하는 무거운 전환이라면, Coroutine의 Task switch는 단지 함수의 실행 위치와 로컬 상태를 저장하고 이벤트 루프가 다음 코루틴을 재개(resume)하는 가벼운 사용자 레벨 전환이라 할 수 있다.

따라서 **Coroutine은 커널이 아닌 애플리케이션 레벨에서 구현된 경량화된 동시성 메커니즘**이며, Context switch의 하드웨어적 문맥 교환에 대응되는 소프트웨어적 제어 흐름 교환(Control-flow switching) 이라고 할 수 있다.

위의 다이어그램을 통해 비동기 처리가 구현된 다이어그램을 살펴보겠다. 이를 위해서 `async` 함수에서 `time.sleep()` 은  `await asyncio.sleep()` 로 바꾸고, DML 함수도 비동기 처리를 해야 한다. 이렇게 비동기 처리가 완료되면 다이어그램을 아래와 같을 것이다.

<img width="1016" height="754" alt="Image" src="https://github.com/user-attachments/assets/c1df682b-aa75-42fe-be77-4b4bfe25b7f1" />

asyncio sleep 과 cursor.execute 같은 I/O 바운드가 실행되면, Event Loop는 등록된 다른 코루틴을 실행한다. 즉 위의 그림에서 coroutine1 이 asyncio sleep을 통해 대기 상태에 들어가면  Event Loop는 Accept Queue에서 바로 $R_2$를 가져와서 coroutine2로 실행을 한다. 이때 오해를 하면 안되는 것이, **병렬 실행이 절대 아니란 점**이다. 위에 그림만 보면 오해를 할 수도 있지만, coroutine이 실행되는 로직은 기존에 실행 중이던 coroutine이 I/O 바운드로 인해 대기 상태에 들어갔을때만, 실행이 되는 **동시성 강화**이다. 따라서 위에서 OS 수준의 Context switch로 비유한 이유가 바로 이러한 Event Loop를 통한 coroutine 실행 관리 로직 때문이다. 이러한 동시성 강화를 통해 약 $T_{1.5}$ 에 모든 실행이 완료되는 것을 볼 수 있다. 

> 비동기 코루틴은 단일 스레드 내에서 오직 하나의 Call Stack 위에서만 실행되며, 동시에 여러 coroutine이 CPU를 점유하는 일은 없다.

OS 수준에서는 단순히 $W$를 실행하고 있으면 되고 이러한 비동기 처리는 애플리케이션 수준의 쓰레드 내부에서 전부 이루어지는 추상화가 ASGI 아키텍처의 철학이다. 

---

## 😘 마무리

**비동기 모델에서의 동시성 강화는 OS 수준의 선점형 스케줄링(preemptive scheduling) 이 아닌, Event Loop를 중심으로 한 협력형(Cooperative) 스케줄링** 에 의해 이루어진다. Event Loop는 OS의 CPU 스케줄러에 대응되는 역할을 수행하며, coroutine 간 전환(Task Switching)은 커널 수준의 Context Switch 대신 사용자 레벨에서 수행되는 **가벼운 실행 흐름 전환(Control-flow switching) 으로 처리된다.** 이번 글에서는 FastAPI를 비롯한 비동기 처리가 논리적으로 어떻게 이루어지는 지와 OS 수준과 애플리케이션 수준에서 헷갈리지 않도록 설명을 해보았다. 다음 글에서는 FastAPI의 비동기 처리 로직을 코드를 통해 알아보고 일반 def 선언은 어떻게 처리되는지 그리고 lifespan을 통한 coroutine 처리를 심도있게 다뤄보겠다.


