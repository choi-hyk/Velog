[Velog로 가기](https://velog.io/@choi-hyk/Design-Patterns-Singleton-Pattern)

released at 2025-07-22 18:04:01 KST

updated at 2025-12-21 02:58:23 KST

|[Creational Pattern](https://velog.io/tags/Creational-Pattern)|[Design Pattern](https://velog.io/tags/Design-Pattern)|
|----|----|


## 🛠️ Singleton Pattern

컴퓨터공학에서 용어를 정의할 때는 보통 그 개념을 가장 잘 설명할 수 있는 단어를 쓰려고 한다. 디자인 패턴도 마찬가지다. 처음 보면 막막하고 헷갈리기 쉬운데, 이름과 역할을 연관지어 생각하면 훨씬 쉽게 이해된다.

`Singleton Pattern`은 말 그대로 프로그램 전체에서 **하나만 존재해야 하는 객체**를 만들 때 사용하는 패턴이다. 따라서 객체를 생성하는 `생성 패턴(Creational Pattern)`이다. 참고로 먼저 생성 패턴을 차례대로 살펴볼 생각이다.

---

### 언제 사용하나?

이런 구조는 다양한 곳에서 자주 등장한다. 예를 들어:

* **Logger**: 로그는 한 군데서 모아야 함
* **ConfigManager**: 설정값은 전역에서 하나만 존재해야 함
* **DB 연결 풀**: 커넥션 풀을 여러 개 만들면 낭비
* **InputManager, AudioManager**: 게임에서 공용으로 관리

---

### 구조

<img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdna%2Fbcx3J6%2FbtsBipDsE9Y%2FAAAAAAAAAAAAAAAAAAAAANRPyiM8CNXzg25E0ZwzlLhxsAjHMNUIBj3RYhddFf8C%2Fimg.webp%3Fcredential%3DyqXZFxpELC7KVnFOS48ylbz2pIh7yKj8%26expires%3D1753973999%26allow_ip%3D%26allow_referer%3D%26signature%3D0Qe3sSy4W03tNBxGnGpcDadO9js%253D">

보통 Singleton 클래스는 자기 자신을 가리키는 `instance` 포인터를 내부에 가지고 있다. 외부에서는 `getInstance()`를 호출해서 접근하는 구조고, 새로운 객체 생성을 막고 기존 객체를 반환하게 된다.
이 Singleton 객체는 클라이언트가 직접 뭔가를 하기보다는 시스템 전체를 대표하는 역할로 많이 쓰인다.

---

### 장점

* **리팩토링이 쉬움**
  나중에 이 객체를 더 이상 싱글톤으로 쓰고 싶지 않을 때 구조를 바꾸기 편하다.

* **전역 변수보다 훨씬 안전**
  전역 객체는 의존성도 높고 유지보수도 어렵지만, Singleton은 전역적으로 관리하면서도 더 안전하게 쓸 수 있다.

---

### 기본 구현

```cpp
#include <iostream>
using namespace std;

class Singleton {
public:
    static Singleton* getInstance() {
        if (_instance == nullptr) {
            _instance = new Singleton();
        }
        return _instance;
    }

    void doSomething() {
        cout << "싱글톤 객체 동작 중" << endl;
    }

private:
    Singleton() {
        cout << "Singleton 생성됨" << endl;
    }

    Singleton(const Singleton&) = delete;
    Singleton& operator=(const Singleton&) = delete;

    static Singleton* _instance;
};

Singleton* Singleton::_instance = nullptr;

int main() {
    Singleton* s1 = Singleton::getInstance();
    s1->doSomething();

    Singleton* s2 = Singleton::getInstance();
    s2->doSomething();

    cout << "주소 비교: " << s1 << " == " << s2 << endl;
}
```

* `getInstance()`에서 `_instance`가 `nullptr`이면 객체를 생성하고, 이미 있으면 그걸 그대로 반환한다.
* 생성자를 `private`으로 막아서 `new Singleton()`처럼 직접 생성하는 걸 방지한다.
* 복사 생성자와 대입 연산자를 `delete` 해서 복사로 다른 인스턴스를 만드는 것도 막는다.

---

### Thread-Safety 문제

하지만 위 코드는 멀티스레드 환경에서는 안전하지 않다.
여러 스레드가 동시에 `getInstance()`를 호출하면, **싱글톤 객체가 두 개 생길 수도 있다.**

이를 막기 위해 **mutex**와 **DCL(Double-Checked Locking)** 기법을 쓴다:

```cpp
#include <iostream>
#include <mutex>
using namespace std;

class Singleton {
private:
    Singleton() {
        cout << "Singleton 생성됨" << endl;
    }

    Singleton(const Singleton&) = delete;
    Singleton& operator=(const Singleton&) = delete;

    static Singleton* _instance;
    static mutex mtx;

public:
    static Singleton* getInstance() {
        if (_instance == nullptr) {
            lock_guard<mutex> lock(mtx);
            if (_instance == nullptr) {
                _instance = new Singleton();
            }
        }
        return _instance;
    }

    void doSomething() {
        cout << "싱글톤 객체에서 메서드 호출됨" << endl;
    }
};

Singleton* Singleton::_instance = nullptr;
mutex Singleton::mtx;
```

* 첫 번째 `if (_instance == nullptr)`는 빠르게 통과하기 위한 check
* mutex를 잠그고 한 번 더 확인한 후 객체를 생성

---

### 최신 C++ 방식: `std::call_once`

```cpp
#include <mutex>

class Singleton {
public:
    static Singleton* getInstance() {
        call_once(_flag, []() {
            _instance = new Singleton();
        });
        return _instance;
    }

private:
    Singleton() {}
    static Singleton* _instance;
    static once_flag _flag;
};
```

* `call_once()`는 `_flag`를 기준으로 단 한 번만 해당 람다를 실행한다.
* 훨씬 안전하고 코드도 깔끔해진다.

---

### 마무리

`Singleton Pattern`은 진짜 하나만 존재해야 하는 경우에 쓰는 강력한 도구다.
하지만 남발하면 **테스트 어려움, 의존성 증가, 결합도 상승** 같은 부작용도 있으니 **정말 필요한 경우에만 쓰는 게 핵심**이다.

다음에 살펴볼 패턴은 `Factory Method Pattern`이다. 해당 패턴은 객체 생성 로직을 직접 쓰는 대신, 객체 생성을 서브클래스에 위임하는 방식을 사용한다. 싱글톤이 **"객체를 하나만 만들자"**는 패턴이라면, 팩토리 메서드는 **"객체 생성을 유연하게 만들자"**는 개념이다.

`Factory Method Pattern`을 적절한 예제와 함께 살펴볼 생각이다.
