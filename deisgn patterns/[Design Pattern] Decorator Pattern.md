[Velog로 가기](https://velog.io/@choi-hyk/Design-Pattern-Decorator-Pattern)

released at 2025-08-15 19:26:47 KST

updated at 2025-10-05 21:28:32 KST

|[Design Pattern](https://velog.io/tags/Design-Pattern)|[Structure Pattern](https://velog.io/tags/Structure-Pattern)|
|----|----|

# Decorator Pattern 🎨

오랜만에 디자인패턴 글을 써보는데, 최근에 IPP로 회사에 가서 이것저것 하고 정신이 없어서 글 쓰는 것을 잊고 있었다.

앞으로는 일주일에 한번은 디자인패턴 글을 쓸 생각이다. 어쨌든 저번 프로토타입 패턴을 마지막으로 **생성패턴은 전부 정리를 완료**했고, 오늘부터는 **장식자 패턴을 시작으로 구조 패턴을 차례대로 정리**해보겠다.

**Decorator Pattern**은 이름에서 알 수 있다시피, **어떤 객체를 장식을 하는 패턴**이다. 참고로 장식자 패턴을 포함한 **Structure Patterns**는 **여러 개의 객체로 이루어진 구조를 정의**해주는 패턴이다. 간단하게 **보편적인 설계도를 정의한 것**이라 보면 된다. 따라서 코드를 작성할 때, **클래스나 함수를 정의하고 객체의 생명주기를 관리하는 방법에는 생성패턴**이 사용된다면, **전체적인 구조를 정의하고, 하나의 모듈로 동작하는 기능을 구현할 때는 구조패턴**을 사용할 경우가 생길 것이다.

다시 돌아가서 장식자 패턴은 **객체에 동적으로 새로운 책임(기능)을 추가하는 방식**으로, **상속의 대안**으로 사용된다. 이 패턴은 **기존 객체의 구조를 변경하지 않고 기능을 확장**할 수 있다.

> **객체에 동적으로 새로운 책임을 추가할 수 있게 합니다. 기능을 추가하려면, 서브클래스를 생성하는 것보다 융통성 있는 방법을 제공합니다**

GOF 책에서 보면 많은 패턴들이 **서브클래스를 생성하는 것을 대체하고 효율적으로 기능을 추가하기 위해 고안**된 것임을 알 수 있다.
그렇다면 여기서 **동적**은 무엇을 뜻하는 것일까?

여기서 동적은 실제로 컴퓨터공학에서 말하는 **`Dynamic`**을 의미한다.
만약 객체를 서브클래스로 기능과 책임을 만들 경우, **컴파일 타임이나 빌드 타임에 "정적"으로 기능과 책임을 담당하는 클래스**를 생성해야 한다. 하지만 장식자 패턴은 **동적으로 실제 런타임 환경에서 이러한 추가 기능 클래스를 추가 가능**하다. 또한 장식자 패턴은 **`Wrapper`**라고도 불리는데, **객체를 감싸서 추가적인 기능이나 책임을 부여하는 구조** 때문에 이렇게 불린다.

**책임을 부여하는 것**이 장식자 패턴에서 가장 중요한 점인데, **장식받는 객체는 자신의 기능만 신경 쓰면 되고**, 나머지 장식을 하는 객체들의 구현과 기능은 신경 쓸 필요가 없다. 따라서 **`Decorator`가 사용되는 순간 장식받는 객체는 `Decorator`의 멤버 변수로 들어가서 기능 호출만 받으면 된다.**

---

## 언제 사용하나? 📌

책에서는 장식자 패턴을 **`TextView` 컴포넌트를 감싸서 기능을 `BorderDecorator`와 `ScrollDecorator`로 예시**를 들었다.

<img width="524" height="220" alt="Image" src="https://github.com/user-attachments/assets/cd5a7c25-28c9-46ef-84a9-6485f3defdc8" />  

해당 이미지를 보면, 기존의 **`TextView`에 `BorderDecorator`와 `ScrollDecorator`로 감싸서 컴포넌트를 이루는 것**을 나타낸다.

<img width="578" height="312" alt="Image" src="https://github.com/user-attachments/assets/d910b2ca-ca2e-4e82-8739-85a9a61d5f73" />  

위 사진을 보면 **동적으로 어떻게 기능을 추가하는지** 이해가 될 것이다. 바로 **`VisualComponent`라는 클래스가 `TextView`를 서브클래스로 가지고 있는데, 해당 클래스가 `Decorator`라는 클래스를 서브클래싱**하여 관리를 한다.

**`Decorator`는 `Draw()`로 원하는 컴포넌트를 그리면 된다.** 여기서 중요한 점이 있는데, **`TextView`는 이러한 `Decorator`들을 알 필요가 없다.** `TextView`를 정의하고, 만약 테두리를 그리고 싶으면 **`TextView`를 `Decorator`에 넘겨주고 해당 클래스에서 장식을 해준다.**

따라서 **`TextView`는 자신의 기능인 텍스트뷰 그리기만 신경**을 쓰면 된다.

---

## 구조 🏗️

<img width="645" height="285" alt="Image" src="https://github.com/user-attachments/assets/b0938fb6-00ba-417f-9dfe-d8db2d85ff14" />  

**`ConcreteDecorator`들은 `Operation()`으로 자신만의 기능과 함께 `ConcreteComponent`의 `Operation()`을 호출**할 것이다. 이렇게 **`ConcreteComponent`는 그저 자신의 기능만 호출하고 추가 기능 확장에 대해서는 신경 쓸 필요가 없다.**

---

## 구현 💻

```cpp
#include <iostream>
#include <memory>

class VisualComponent {
public:
    virtual void Draw() = 0;
    virtual ~VisualComponent() {}
};

class TextView : public VisualComponent {
public:
    void Draw() override {
        std::cout << "기본 텍스트 뷰 그리기" << std::endl;
    }
};

class Decorator : public VisualComponent {
protected:
    std::unique_ptr<VisualComponent> _component;
public:
    Decorator(std::unique_ptr<VisualComponent> component)
        : _component(std::move(component)) {} 
    void Draw() override {
        if (_component) {
            _component->Draw();
        }
    }
};

class BorderDecorator : public Decorator {
private:
    int _width;
    void DrawBorder() {
        std::cout << "테두리 그리기" << std::endl;
    }
public:
    BorderDecorator(std::unique_ptr<VisualComponent> component, int width)
        : Decorator(std::move(component)), _width(width) {}

    void Draw() override {
        Decorator::Draw();
        DrawBorder();
    }
};

class ScrollDecorator : public Decorator {
private:
    void DrawScroll() {
        std::cout << "스크롤바 그리기" << std::endl;
    }
public:
    ScrollDecorator(std::unique_ptr<VisualComponent> component)
        : Decorator(std::move(component)) {}

    void Draw() override {
        Decorator::Draw();
        DrawScroll();
    }
};

int main() {
    auto textView = std::make_unique<TextView>();
    std::cout << "\n--- 기본 TextView ---" << std::endl;
    textView->Draw();
    
    auto textViewWithBorder = std::make_unique<BorderDecorator>(std::move(textView), 1);
    std::cout << "\n--- 테두리 추가된 TextView ---" << std::endl;
    textViewWithBorder->Draw();

    auto textViewWithBoth = std::make_unique<ScrollDecorator>(
        std::make_unique<BorderDecorator>(
            std::make_unique<TextView>(), 1));
    std::cout << "\n--- 테두리와 스크롤 모두 추가된 TextView ---" << std::endl;
    textViewWithBoth->Draw();

    return 0;
}
```

위의 코드에서 중요하게 볼 점은, 바로 **`Decorator`들이 `VisualComponent* _component;`로 장식할 객체인 `TextView`를 멤버 변수로 받는 것**이다. 이를 통해서 **`TextView`는 만약 `Decorator`를 사용하고 싶지 않으면, 해당 객체들을 만들 필요가 없다.**

다시 상기시키자면, **장식자 패턴에서 가장 중요한 점은 바로 책임 전가**이다.

---

## 마무리

Decorator 패턴은 기존 객체의 구조를 변경하지 않고, 런타임에 동적으로 새로운 기능과 책임을 부여할 수 있는 디자인 패턴이다. 상속 대신 객체를 감싸는 방식(Wrapper)을 사용하여 기능을 확장하므로, 필요할 때만 선택적으로 기능을 조합할 수 있고 클래스 폭발 문제를 피할 수 있다는 장점이 있다. 하지만 장식이 중첩될수록 구조가 복잡해지고, 디버깅이 어려워질 수 있으며, 너무 많은 데코레이터가 사용되면 유지보수 비용이 증가할 수 있다는 단점이 있다. 결국 이 패턴은 **기능 확장이 빈번하고, 유연한 구조가 필요한 UI 컴포넌트나 모듈성 높은 시스템**에서 특히 효과적으로 사용된다.

다음 글에서는 **구조 패턴** 중 **Adapter Pattern**에 대해 알아볼 예정이다. Adapter 패턴은 호환되지 않는 인터페이스를 가진 클래스들이 함께 동작할 수 있도록 연결하는 패턴으로, 기존 코드를 수정하지 않고 새로운 환경에 맞출 수 있다는 장점이 있다.

[[참고] Decorator Pattern](https://www.cs.unc.edu/~stotts/GOF/hires/pat4dfso.htm)


