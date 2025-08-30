[Velog로 가기](https://velog.io/@choi-hyk/Design-Pattern-Prototype-Pattern)
released at 2025-08-02 19:08:10 KST
updated at 2025-08-16 13:05:58 KST

|[Creational Pattern](https://velog.io/tags/Creational-Pattern)|[Design Pattern](https://velog.io/tags/Design-Pattern)|
|----|----|

## 🛠️Prototype Pattern

Prototype은 "원형"이라는 의미를 가진다. 이를 토대로 `Prototype` 객체는 원형을 나타낸다는 것을 알 수 있고, `Prototype Pattern`은 원형 객체를 사용하여 객체를 생성하는 패턴임을 알 수 있다.

> 원형이 되는(prototype) 인스턴스를 사용하여 생성할 객체의 종류를 명시하고, 이렇게 만든 견본을 복사해서 새로운 객체를 생성합니다.

즉, 해당 패턴에서는 복사가 매우 중요한 행위임을 알 수 있다. 그렇다면 클라이언트는 해당 객체를 어떨 때 사용할 수 있을까? 만약 원형 객체가 존재해도 해당 객체를 사용하기에 적절하지 않으면 사용하기 힘들 것이다. 따라서 해당 패턴은 클라이언트의 시작점을 기준으로 생각해야 한다. 보통 우리가 패턴을 사용할 때는 해당 패턴의 객체, 여기서는 `Prototype`의 구현을 중점으로 보겠지만 만약 해당 패턴을 사용할 기회가 생기면 우리는 이미 생성된 `Prototype`을 사용할 것이다.

---

### 언제 사용하나?

![이미지](https://github.com/user-attachments/assets/63426eba-b35e-424c-bd11-05c6378ef3b4)

GOF 책에서는 그래픽 편집기에서 노래 편집기 기능을 확장하는 것을 예로 들어 `Prototype Pattern`을 설명한다. 그래픽 편집기에서 노래 편집기 기능을 넣으려면 `GraphicTool`에 `MusicGraphicTool`을 서브 클래스로 생성하면 될 것이다. 그러나 이는 확장성과 유지보수 면에서 매우 좋지 않은 형태가 된다. 따라서 위 그림처럼 `Graphic(Prototype)`에서 제공하는 `WholeNote`, `HalfNote`를 복제하여 사용하는 구조를 보여준다. `GraphicTool`은 `Graphic` 객체 입장에서는 자신의 서브클래스를 복제해서 사용하는 Client이다. 그리고 아마도 `Graphic` 객체는 그래픽 프레임워크 또는 드라이버 제품으로 제공되어 Client들이 사용할 것이다.

---

### 구조

![이미지](https://github.com/user-attachments/assets/31db9050-d74f-4b6d-86c6-cff65e62d721)

`Prototype Pattern`의 구조는 매우 간단하다. 우리는 구현된 `Prototype`을 `clone()`만 사용하면 된다. 따라서 해당 패턴은 구현을 중점으로 보기보다는 다양한 `Prototype`을 조화롭게 클라이언트들이 결합해서 사용하는 것이 중요하다고 볼 수 있다.

---

### 구현 방법

#### 전체 코드

```cpp
#include <iostream>
#include <string>
#include <memory>
using namespace std;

// ---------------------------
// Graphic : Prototype 인터페이스
// ---------------------------
class Graphic {
public:
    virtual ~Graphic() {}
    virtual unique_ptr<Graphic> clone() const = 0;
    virtual void draw(const string& position) const = 0;
};

// ---------------------------
// Staff : 구체 Prototype
// ---------------------------
class Staff : public Graphic {
public:
    unique_ptr<Graphic> clone() const override {
        return make_unique<Staff>(*this);
    }

    void draw(const string& position) const override {
        cout << "오선지를 " << position << " 위치에 그림" << endl;
    }
};

// ---------------------------
// MusicalNote : 추상 클래스
// ---------------------------
class MusicalNote : public Graphic { };

// WholeNote : 구체 Prototype
class WholeNote : public MusicalNote {
public:
    unique_ptr<Graphic> clone() const override {
        return make_unique<WholeNote>(*this);
    }

    void draw(const string& position) const override {
        cout << "온음표를 " << position << " 위치에 그림" << endl;
    }
};

// HalfNote : 구체 Prototype
class HalfNote : public MusicalNote {
public:
    unique_ptr<Graphic> clone() const override {
        return make_unique<HalfNote>(*this);
    }

    void draw(const string& position) const override {
        cout << "2분음표를 " << position << " 위치에 그림" << endl;
    }
};

// ---------------------------
// Tool : Client 역할 추상 클래스
// ---------------------------
class Tool {
public:
    virtual ~Tool() {}
    virtual void manipulate() = 0;
};

// GraphicTool : Client 역할 구체 클래스
class GraphicTool : public Tool {
private:
    unique_ptr<Graphic> prototype;
public:
    GraphicTool(unique_ptr<Graphic> proto) : prototype(move(proto)) {}

    void manipulate() override {
        auto p = prototype->clone();
        p->draw("새로운 위치");
    }
};

// RotateTool : 다른 Tool 예시
class RotateTool : public Tool {
public:
    void manipulate() override {
        cout << "대상을 회전시킴" << endl;
    }
};

// ---------------------------
// 클라이언트 코드
// ---------------------------
int main() {
    // 온음표 도구
    unique_ptr<Tool> wholeNoteTool = make_unique<GraphicTool>(make_unique<WholeNote>());
    wholeNoteTool->manipulate();

    // 2분음표 도구
    unique_ptr<Tool> halfNoteTool = make_unique<GraphicTool>(make_unique<HalfNote>());
    halfNoteTool->manipulate();

    // 오선지 도구
    unique_ptr<Tool> staffTool = make_unique<GraphicTool>(make_unique<Staff>());
    staffTool->manipulate();

    return 0;
}
```

`Prototype`인 `Graphic` 클래스는 `clone()`과 `draw()` 기능을 제공하여 원형 클래스 인터페이스를 제공한다. 나머지 `Staff`, `WholeNote`, `HalfNote`의 내부적인 구현은 제쳐 두고 Client는 이제 해당 클래스들을 `clone()`하여 사용하면 된다.

---

#### Client

```cpp
class Tool {
public:
    virtual ~Tool() {}
    virtual void manipulate() = 0;
};

// GraphicTool : Client 역할 구체 클래스
class GraphicTool : public Tool {
private:
    unique_ptr<Graphic> prototype;
public:
    GraphicTool(unique_ptr<Graphic> proto) : prototype(move(proto)) {}

    void manipulate() override {
        auto p = prototype->clone();
        p->draw("새로운 위치");
    }
};
```

따라서 Client는 `Graphic`을 `clone()`하기 위한 `GraphicTool` 객체를 생성해야 한다. `manipulate()`를 통해 프로토타입을 `clone()`하고 `draw()`하는 것을 볼 수 있다.

---

### 마무리

Prototype 패턴은 객체를 `new`로 직접 생성하지 않고 이미 존재하는 객체를 복제(clone)하여 새로운 객체를 만드는 디자인 패턴으로, **객체 생성 비용이 큰 경우에 효율적이고 다양한 객체를 런타임에 유연하게 사용할 수 있다는 장점**이 있다.
하지만 **깊은 복사와 얕은 복사 문제를 주의해야 하고, 모든 클래스마다 clone 기능을 구현해야 하므로 관리 비용이 늘어난다는 단점**이 있다.
결국 이 패턴은 복잡한 초기화 과정을 거치는 객체나 자주 반복해서 생성해야 하는 객체가 있는 경우에 특히 효과적으로 사용된다.

다음 글에서는 이제 구조 패턴(Structual Pattern)의 `Decorator Pattern` 에 대해서 알아볼 생각이다. 해당 패턴은 주어진 용도에 따라 여러 객체에 서브클래싱을 유연하게 하는 패턴이다.