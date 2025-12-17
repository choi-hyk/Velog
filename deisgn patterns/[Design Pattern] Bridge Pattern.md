[Velog로 가기](https://velog.io/@choi-hyk/Design-Pattern-Bridge-Pattern)

released at 2025-08-31 16:51:56 KST

updated at 2025-12-17 18:22:14 KST

|[Bridge Pattern](https://velog.io/tags/Bridge-Pattern)|[Design Pattern](https://velog.io/tags/Design-Pattern)|[Structure Pattern](https://velog.io/tags/Structure-Pattern)|
|----|----|----|

# Bridge Pattern 🌉

이번에는 `Bridge Pattern` 에 대해서 알아보겠다. Bridge Pattern은 말 그대로 클래스와 클래스를 가교(Bridge)라는 관계로 정의하는 패턴이다. 한번 생각해보자, 우리가 어떠한 클래스를 상속을 통해 구현을 할때, 깊이 1에 있는 클래스들은 해당 클래스의 원형을 그대로 따라갈 것이다. 근데 만약에, 부모 클래스가(깊이 0) 새로운 개념의 서브 클래스를 생성한다 생각해보자. 

<img width="610" height="189" alt="Image" src="https://github.com/user-attachments/assets/efba71ae-dd25-430c-b038-8eecff148e32" />

위의 이미지는 GOF 책에서 예시로 든 사용자 인터페이스 툴킷인 `Winodow` 클래스의 **클래스 폭발**을 보여준다. 툴킷인 `Window` 클래스를 사용해서 우리가 각 플랫폼의 특성이 반영된 `XWindow` 와 `PMWindow` 플랫폼을 구현했다고 해보자, 해당 구현만 존재하면 사용하는데는 문제가 없을 것이다. 그런데, `Window` 구현자가 새로운 기능을 담은 `Window` 인 `IconWindow` 를 출시 했다. 그러면 우리는 기존의 `XWindow` 와 `PMWindow` 를 다시 `IconWindow` 에 상속 받아서 해당 `Icon` 기능이 포함된 클래스들을 재정의 해야 한다. 매우 번거롭지 않은가?

그래서 사용되는 패턴이 **Bridge Pattern**이다.

> 구현에서 추상을 분리하여, 이들이 독립적으로 다양성을 가질 수 있도록 합니다.

구현에서 추상을 분리한다는 것은, 구현체와 추상으로 생성된 추가 클래스들을 분리한다는 것이다. 참고로 **Bridge Pattern**은 **핸들/구현부(Handle/Body)** 라는 이름으로도 불린다.

---

## 언제 사용하나? 📌

책에서는 위에서 말한 예시로 Bridge Pattern을 설명한다. 

<img width="576" height="359" alt="Image" src="https://github.com/user-attachments/assets/f9be37fd-6e35-4cd0-bf37-5c98a4402e4b" />

이미지를 보면, `Window` 의 추상 클래스로 `IconWindow`, `TransientWindow` 가 설정되어 있고, `Window`는 `imp` 라는 구현체 인스턴스를 가지게 된다. 이 `imp` 는 `WindowImp` 를 참조하게 된다. `IconWindow`, `TransientWindow`는 기존의 `Winodw` 에서 제공하는 `DrawText()` 와 `DrawRect()` 로 자신들이 제공하는 기능을 구현하고 있다. 여기서 해당 패턴의 핵심이 나오는데, 바로 `WindowImp`는 `DrawRect()`를 4개의 `DevDrawLine()` 으로 구현 중이다. 이것이 **Bridge Pattern** 의 구현부의 역할이다. 구현부는 가장 **저수준의 구현**을 제공하고, 추상부는 해당 구현체들을 활용해서 실질적인 동작을 수행한다. 그리고 이러한 저수준의 구현을 하나의 클래스로 정의하면 해당 클래스의 서브 클래싱을 통해 여러가지 플랫폼에서 활용이 가능하다. 

이렇게 함으로써 얻는 가장 큰 이점은, 기능(추상화 계층)과 플랫폼(구현 계층)을 각각 독립적으로 관리할 수 있다는 점이다. 기능이 늘어날 때마다 모든 플랫폼별 클래스를 다시 작성해야 하는 클래스 폭발 문제를 피할 수 있고, 새로운 플랫폼을 지원하는 것도 훨씬 수월하다.

---

## 구조 🏗️

<img width="600" height="246" alt="Image" src="https://github.com/user-attachments/assets/b7413f5c-919a-422a-a53b-64932b149974" />

구조는 위의 예시를 이해했으면, 바로 파악이 될것이다. 정리하자면, **Bridge Pattern**은 상속으로 인해 **기능 × 플랫폼 조합이 기하급수적으로 늘어나는 문제를 해결하기 위해, 추상 계층과 구현 계층을 분리하고, 이를 가교(imp)로 연결하는 방식**이다. 이 덕분에 기능과 구현을 **분리된 축(axis)**으로 관리할 수 있어 확장성과 유지보수성이 크게 향상된다.

여기서 핵심 포인트는 **추상은 고수준 동작을 정의, 구현은 저수준 세부사항을 담당, 그리고 둘은 런타임에 조합된다 라는 구조**다.

---
## 구현 💻

```cpp
#include <iostream>
#include <memory>
#include <string>
#include <algorithm>

// -------- Primitive --------
struct Point { int x{}, y{}; };

// -------- Implementor --------
class WindowImp {
public:
    virtual ~WindowImp() = default;
    virtual void DeviceRect(int x0, int y0, int x1, int y1) = 0;
    virtual void DeviceText(const char* s, int x, int y) = 0;
};

// -------- Concrete Implementors --------
class XWindowImp : public WindowImp {
public:
    void DeviceRect(int x0, int y0, int x1, int y1) override {
        int x = std::min(x0, x1);
        int y = std::min(y0, y1);
        int w = std::abs(x1 - x0);
        int h = std::abs(y1 - y0);
        std::cout << "[X] Rect (" << x << "," << y << ") w=" << w << " h=" << h << "\n";
    }
    void DeviceText(const char* s, int x, int y) override {
        std::cout << "[X] Text \"" << s << "\" @(" << x << "," << y << ")\n";
    }
};

class PMWindowImp : public WindowImp {
public:
    void DeviceRect(int x0, int y0, int x1, int y1) override {
        int left   = std::min(x0, x1);
        int right  = std::max(x0, x1);
        int bottom = std::min(y0, y1);
        int top    = std::max(y0, y1);
        std::cout << "[PM] Rect L=" << left << " R=" << right
                  << " B=" << bottom << " T=" << top << "\n";
    }
    void DeviceText(const char* s, int x, int y) override {
        std::cout << "[PM] Text \"" << s << "\" @(" << x << "," << y << ")\n";
    }
};

// -------- Abstraction --------
class Window {
public:
    explicit Window(std::unique_ptr<WindowImp> imp) : imp_(std::move(imp)) {}
    virtual ~Window() = default;

    // 고수준 API
    virtual void DrawRect(const Point& p1, const Point& p2) {
        imp_->DeviceRect(p1.x, p1.y, p2.x, p2.y);
    }
    virtual void DrawText(const std::string& s, const Point& at) {
        imp_->DeviceText(s.c_str(), at.x, at.y);
    }
    virtual void DrawContents() = 0; 

protected:
    WindowImp* imp() { return imp_.get(); }

private:
    std::unique_ptr<WindowImp> imp_; 
};

// -------- Refined Abstractions --------
class IconWindow : public Window {
public:
    IconWindow(std::unique_ptr<WindowImp> imp, std::string iconName)
        : Window(std::move(imp)), icon_(std::move(iconName)) {}
    void DrawContents() override {
        DrawText(("ICON:" + icon_), {0, 0});
        DrawRect({0, 0}, {32, 32});
    }
private:
    std::string icon_;
};

class TransientWindow : public Window {
public:
    explicit TransientWindow(std::unique_ptr<WindowImp> imp)
        : Window(std::move(imp)) {}
    void DrawContents() override {
        DrawText("Transient", {8, 16});
        DrawRect({4, 4}, {128, 64});
    }
};

// -------- Client --------
int main() {
    // 런타임에 구현 선택 → 같은 추상도 다른 구현과 조합 가능
    IconWindow w1(std::make_unique<XWindowImp>(), "app.png");
    TransientWindow w2(std::make_unique<PMWindowImp>());

    w1.DrawContents(); // X 구현으로 그리기
    w2.DrawContents(); // PM 구현으로 그리기
    return 0;
}
```
전체 코드는 이렇게 되는데, 책에서 제시한 코드는 기능이 너무 많아서 간단하게 `DrawRect()`와 `DrawText()`만 구현을 했다. 그리고 Refined Abstraction으로 `IconWinodw` 만 구현을 했다.


#### Implementor
```cpp
// -------- Implementor --------
class WindowImp {
public:
    virtual ~WindowImp() = default;
    virtual void DeviceRect(int x0, int y0, int x1, int y1) = 0;
    virtual void DeviceText(const char* s, int x, int y) = 0;
};

// -------- Concrete Implementors --------
class XWindowImp : public WindowImp {
public:
    void DeviceRect(int x0, int y0, int x1, int y1) override {
        std::cout << "[X] Rect (" << x0 << "," << y0
                  << ")-(" << x1 << "," << y1 << ")\n";
    }
    void DeviceText(const char* s, int x, int y) override {
        std::cout << "[X] Text \"" << s << "\" @(" << x << "," << y << ")\n";
    }
};

class PMWindowImp : public WindowImp {
public:
    void DeviceRect(int x0, int y0, int x1, int y1) override {
        std::cout << "[PM] Rect (" << x0 << "," << y0
                  << ")-(" << x1 << "," << y1 << ")\n";
    }
    void DeviceText(const char* s, int x, int y) override {
        std::cout << "[PM] Text \"" << s << "\" @(" << x << "," << y << ")\n";
    }
};
```

Bridge Pattern의 **Implementation(구현부)** 는 `WindowImp`라는 인터페이스를 중심으로 구성된다. 이 클래스는 `DeviceRect`, `DeviceText`와 같이 플랫폼 의존적인 저수준 API(Application Programming Interface)를 정의한다. 그리고 실제 구현은 `XWindowImp`, `PMWindowImp`에서 이루어진다. 예를 들어 `XWindowImp`는 X 윈도우 시스템 호출을, `PMWindowImp`는 프레젠테이션 매니저 호출을 각각 캡슐화한다. 즉, **어떻게 그릴 것인가**라는 부분을 담당하는 것이 바로 구현부이며, 추상부와 독립적으로 교체하거나 확장할 수 있다.


#### Abstraction
```cpp
// -------- Abstraction --------
class Window {
public:
    explicit Window(std::unique_ptr<WindowImp> imp) : imp_(std::move(imp)) {}
    virtual ~Window() = default;

    virtual void DrawRect(const Point& p1, const Point& p2) {
        imp_->DeviceRect(p1.x, p1.y, p2.x, p2.y);
    }
    virtual void DrawText(const std::string& s, const Point& at) {
        imp_->DeviceText(s.c_str(), at.x, at.y);
    }
    virtual void DrawContents() = 0;

protected:
    WindowImp* imp() { return imp_.get(); }

private:
    std::unique_ptr<WindowImp> imp_;
};
```

**Abstraction(추상부)**는 `Window` 클래스가 담당한다. `Window`는 클라이언트에 노출되는 고수준 인터페이스를 정의하며, `DrawRect`, `DrawText` 같은 메서드를 통해 기능을 제공한다. 하지만 직접 그리기를 수행하지 않고, 내부에 `std::unique_ptr<WindowImp>`를 보관해 실제 동작을 구현부에 위임한다. 이렇게 하면 클라이언트는 `Window`의 API만 이용하면 되고, 저수준 동작은 구현부에서 알아서 처리된다.

#### Refined Abstraction
```cpp
// -------- Refined Abstractions --------
class IconWindow : public Window {
public:
    IconWindow(std::unique_ptr<WindowImp> imp, std::string iconName)
        : Window(std::move(imp)), icon_(std::move(iconName)) {}
    void DrawContents() override {
        DrawText(("ICON:" + icon_), {0, 0});
        DrawRect({0, 0}, {32, 32});
    }
private:
    std::string icon_;
};

class TransientWindow : public Window {
public:
    explicit TransientWindow(std::unique_ptr<WindowImp> imp)
        : Window(std::move(imp)) {}
    void DrawContents() override {
        DrawText("Transient", {8, 16});
        DrawRect({4, 4}, {128, 64});
    }
};
```

`IconWindow`와 `TransientWindow` 같은 **Refined Abstraction**은 `Window`를 상속받아 고수준의 행위를 구체화한다. 예를 들어 `IconWindow`는 아이콘을 그리는 동작을 정의하고, `TransientWindow`는 임시 창을 그리는 방식을 정의한다. 하지만 이들도 직접 저수준 연산을 구현하지 않고, `imp()`를 통해 내부의 `WindowImp`에 작업을 위임한다. 이렇게 추상부는 “무엇을 할 것인지”를 정의하고, 구현부는 “어떻게 할 것인지”를 책임지게 되는 구조가 된다. 물론 나는 `IconWindow` 만 구현을 한 상태이다.

#### Client
```cpp
// -------- Client --------
int main() {
    IconWindow w1(std::make_unique<XWindowImp>(), "app.png");
    TransientWindow w2(std::make_unique<PMWindowImp>());

    w1.DrawContents(); // X 플랫폼 구현으로 동작
    w2.DrawContents(); // PM 플랫폼 구현으로 동작
    return 0;
}
```

마지막으로 클라이언트는 실행 시점에 `IconWindow`나 `TransientWindow`를 생성하면서 원하는 구현체(`XWindowImp` 혹은 `PMWindowImp`)를 주입할 수 있다. 이렇게 **런타임 조합(Runtime Composition)** 을 활용하면, 기능 축(추상)과 플랫폼 축(구현)을 완전히 독립적으로 확장할 수 있으며, 기능 × 플랫폼 조합에 따라 모든 클래스를 미리 만들어야 하는 클래스 폭발 문제를 방지할 수 있다.

---

# 마무리 😘

**Bridge Pattern**은 **추상과 구현을 분리해서 독립적으로 확장할 수 있도록 만들어주는 구조적 패턴**이다. 예시에서 보았듯이, **추상화 계층과 구현 계층을 분리해두면 새로운 기능을 추가하더라도 클래스가 불필요하게 늘어나지 않고 훨씬 유연하게 확장할 수 있다**. 즉, 어댑터 패턴이 기존 인터페이스의 불일치를 해결하기 위한 사후적 접근이었다면, **Bridge Pattern은 처음부터 확장을 고려한 선제적 설계 방식**이라고 볼 수 있다.

다음 글에서는 마찬가지로 구조 패턴 중 하나인 `Composite Pattern`을 다뤄볼 생각이다. `Composite Pattern`은 객체들을 **트리 구조로 묶어서 부분-전체 계층을 표현**하는 데 초점이 맞춰져 있다. 즉, **개별 객체와 객체 집합을 동일한 방식으로 다룰 수 있게 해주는 패턴**인데, 이를 통해 복잡한 계층 구조도 단순하게 다룰 수 있는 장점이 있다.

[[참고] Bridge Pattern](https://www.cs.unc.edu/~stotts/GOF/hires/pat4bfso.htm) 