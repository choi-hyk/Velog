[Velog로 가기](https://velog.io/@choi-hyk/Design-Pattern-Adapter-Pattern)

released at 2025-08-24 17:29:55 KST

updated at 2025-09-26 13:21:31 KST

|[Design Pattern](https://velog.io/tags/Design-Pattern)|[Structure Pattern](https://velog.io/tags/Structure-Pattern)|
|----|----|

# Adapter Pattern 🪛

이번에는 `Adapter Pattern`에 대해서 알아보겠다. GOF 디자인패턴 책에서는 구조패턴을 설명할 때 **Adapter Pattern** 을 제일 먼저 설명한다. **Adapter Pattern**은 말 그대로 기존의 클래스 인터페이스에 다른 라이브러리나 인터페이스를 결합하기 위해 사용하는 패턴이다. 그래서 구조는 매우 직관적이다. 기존에 우리가 사용할 인터페이스와 결합할 인터페이스를 다중 상속 받는 **클래스 어댑터**를 생각해 볼 수 있고, 다른 방법으로는 결합할 인터페이스를 인스턴스로 가지고 있는 **객체 어댑터**를 생각해 볼 수 있다

> 클래스의 인터페이스를 사용자가 기대하는 인터페이스 형태로 적응(변한)시킵니다. 서로 일치하지 않는 인터페이스를 갖는 클래스들을 함께 동작시킵니다.

---

## 언제 사용하나? 📌

책에서는 어댑터 패턴을 `Shape`라는 그래픽을 관리하는 클래스에 `TextView` 기능을 결합하는 예제로 설명을 한다.

<img width="626" height="236" alt="Image" src="https://github.com/user-attachments/assets/a31f7502-6532-41d8-92a3-caadc49ebae5" />

위의 그림은 **객체 어댑터를** 표현하고 있다. 그 이유는 `TextShape`가 `TextView`를 상속하지 않고 포함(Composition) 하고 있기 때문이다. 즉, TextShape 안에 `TextView` 인스턴스를 멤버 변수로 두고, `Shape`의 인터페이스를 구현하면서 내부적으로 `TextView`의 기능을 호출해주는 방식이다.

반면에 **클래스 어댑터** 방식이라면 `TextShape`가 `Shape`를 상속함과 동시에 `TextView`도 상속받아야 한다. 즉, 다중 상속을 이용해서 `TextView` 기능을 바로 가져오는 구조이다. 하지만 이렇게 하면 유연성이 떨어지고, 언어 제약(자바는 다중 상속 불가) 때문에 현실적으로 잘 안 쓰이는 경우가 많다.

---

## 구조 🏗️

#### 클래스 어댑터
<img width="543" height="197" alt="Image" src="https://github.com/user-attachments/assets/37a23640-e4f8-4e9b-9439-b43edd57c22c" />

#### 객체 어댑터
<img width="524" height="197" alt="Image" src="https://github.com/user-attachments/assets/67f292f1-5c1f-495a-bccd-59f669cf7ad6" />

구조는 매우 간단하다. 클래스 어댑터는 상속(Inheritance) 을 이용해서 구현하고, 객체 어댑터는 합성(Composition) 을 이용해서 구현한다. 즉, 클래스 어댑터는 이미 존재하는 클래스를 직접 상속받아 새로운 인터페이스를 맞추는 방식이고, 객체 어댑터는 기존 클래스를 멤버 변수로 두고 그 객체의 기능을 위임(delegate)하는 방식이다.

클래스 어댑터는 상속을 쓰는 만큼 **기존 클래스의 세부 구현에 강하게 묶인다.** 대신 **성능상 조금 더 단순하고 직접적이다.**

객체 어댑터는 **합성을 쓰기 때문에 더 유연하고, 다른 클래스와도 쉽게 조합할 수 있다. 다형성을 활용하기에도 적합하다.**

정리하면, **"빠르고 단순하게"라면 클래스 어댑터**, **"유연하고 확장성 있게"라면 객체 어댑터**를 쓰는 게 맞다. 

---

## 구현 💻

```cpp
#include <iostream>
#include <string>
#include <memory>

using namespace std;

struct Point { int x{}, y{}; };
struct Size  { int w{}, h{}; };
struct Rect  { int x1{}, y1{}, x2{}, y2{}; };

ostream& operator<<(ostream& os, const Rect& r) {
    return os << "Rect{(" << r.x1 << "," << r.y1 << ") ~ (" << r.x2 << "," << r.y2 << ")}";
}

class Manipulate;
class TextManipulator;

class Shape{
    public:
        ~Shape()  = default;
        virtual void boundingBox() const = 0;
        virtual unique_ptr<Manipulate> createManipulate() const = 0;
};

class Manipulate {
    public:
        Manipulate() = default;               
        virtual ~Manipulate() = default;
        virtual void manipulate() const {
            std::cout << "Shape 조작\n";
        }
};

class TextManipulator : public Manipulate
{
    public:
        void manipulate() const override {
            std::cout << "TextShape 조작\n";
        }
};

class Line : public Shape
{
    public:
        Line(Point p1, Point p2) : p1_(p1), p2_(p2) {}
        
        void boundingBox() const override {
            Rect r{
                min(p1_.x, p2_.x),
                min(p1_.y, p2_.y),
                max(p1_.x, p2_.x),
                max(p1_.y, p2_.y)
            };
            std::cout << "[Line] boundingBox = " << r << "\n";
        }

        unique_ptr<Manipulate> createManipulate() const override {
            return make_unique<Manipulate>();
        }
    
    private:
        Point p1_{}, p2_{};
};

class TextView{
    public: 
        virtual ~TextView() = default;
        Point getOrigin() const { return origin_; }
        Size  getExtent() const { return extent_; }
        
        virtual bool isEmpty() const = 0;
    
    protected:
        void setOrigin(Point p) { origin_ = p; }
        void setExtent(Size s)  { extent_ = s; }
        
    private:
        Point origin_{0, 0};
        Size  extent_{0, 0};
};

class TextShape : public Shape, private TextView
{
    public:
        TextShape(Point origin, Size extent, bool empty = false) : empty_(empty) {
            setOrigin(origin);
            setExtent(extent);
        }

    void boundingBox() const override {
        Point o = getOrigin();
        Size  s = getExtent();
        Rect r{o.x, o.y, o.x + s.w, o.y + s.h};
        cout << "[TextShape] origin=(" << o.x << "," << o.y
            << "), extent=(" << s.w << "," << s.h << ") -> boundingBox = "
            << r << "\n";
        }
        
        unique_ptr<Manipulate> createManipulate() const override {
            return make_unique<TextManipulator>();
        }
        
        bool isEmpty() const override {
            return empty_;
        }
    
    private:
        bool empty_{false};
};

int main() {
    unique_ptr<Shape> s1 = make_unique<Line>(Point{10, 5}, Point{2, 20});
    s1->boundingBox();
    s1->createManipulate()->manipulate();

    unique_ptr<Shape> s2 = make_unique<TextShape>(Point{100, 200}, Size{50, 20});
    s2->boundingBox();
    s2->createManipulate()->manipulate();

    return 0;
}
```

책에서 예제로 든 `Shape`에 `TextView`를 결합하는 **클래스 어댑터**이다. `Shape` 는 2개의 기능을 제공하는데 `Shape`를 생성하면 경계선 박스를 만드는 함수 `boundingBox()` 그리고 `Shape`를 이동시키거나 조작하는 조작기를 생성하는 `createManipulate()` 이 2가지의 기능을 제공한다. 이때 기존에 원래 존재하는 `Line`은 `Shape`의 기능을 그대로 상속받아 구현하고 있다. 우리는 `TextShape`라는 어댑터를 통해 `TextView`를 `Shape`에서 사용할 수 있도록 하는 것이 목표이다.

#### Adaptee 
```cpp
class TextView{
    public: 
        virtual ~TextView() = default;
        Point getOrigin() const { return origin_; }
        Size  getExtent() const { return extent_; }
        
        virtual bool isEmpty() const = 0;
    
    protected:
        void setOrigin(Point p) { origin_ = p; }
        void setExtent(Size s)  { extent_ = s; }
        
    private:
        Point origin_{0, 0};
        Size  extent_{0, 0};
};
```
`TextView`는 3개의 기능이 존재하는데, 자신의 위치와 크기를 알려주는`getOrigin()`, `getExtent()` 두가지 기능과 텍스트가 채워져 있는지 아닌지를 알려주는 `isEmpty()`가 있다. 따라서 Target인 `Shape` 가 제공하는 두가지 기능인  `boundingBox()` 와 `createManipulator()`를 연동하기 위해서 기존의 `TextView`의 기능을 적절히 조합해서 만들거나 아예 새로운 코드를 넣어서 기능을 연동시켜야 한다.

#### Adapter
```cpp
class TextShape : public Shape, private TextView
{
    public:
        TextShape(Point origin, Size extent, bool empty = false) : empty_(empty) {
            setOrigin(origin);
            setExtent(extent);
        }

    void boundingBox() const override {
        Point o = getOrigin();
        Size  s = getExtent();
        Rect r{o.x, o.y, o.x + s.w, o.y + s.h};
        cout << "[TextShape] origin=(" << o.x << "," << o.y
            << "), extent=(" << s.w << "," << s.h << ") -> boundingBox = "
            << r << "\n";
        }
        
        unique_ptr<Manipulate> createManipulate() const override {
            return make_unique<TextManipulator>();
        }
        
        bool isEmpty() const override {
            return empty_;
        }
    
    private:
        bool empty_{false};
};
```

`TextShape`는 말한 것 처럼 다중상속을 통해 `Shape` 와 `TextView`를 받고 있다. 여기서 중요한 점이 Adaptee인 TextView는 Private로 해야 한다. 이유는 당연히 Target이 Adaptee를 Adater를 통해 사용할 때 내부의 구조를 알 필요가 없기 때문이다. `boundingBox()`를 보면 `TextView`의 `getOrigin()` 와 `getExtent()`를 사용해서 위치와 크기를 얻고 경계 박스를 구현하는 것으로 연동을 완료했다. 그런데 `createManipulator()`는 기존의 기능으로 연동이 불가능 하므로 새로운 `TextManipulator`를 생성해서 연동해야 한다.

#### TextManipulator
```cpp
class TextManipulator : public Manipulate
{
    public:
        void manipulate() const override {
            std::cout << "TextShape 조작\n";
        }
};
```

이렇게 만든 `TextManipulator`를 통해 완벽히 연동이 되었다. 이제 클라이언트는 기존에 Shape를 이용하는 방식으로 TextView를 이용가능하다.

#### Client
```cpp
int main() {
    unique_ptr<Shape> s1 = make_unique<Line>(Point{10, 5}, Point{2, 20});
    s1->boundingBox();
    s1->createManipulate()->manipulate();

    unique_ptr<Shape> s2 = make_unique<TextShape>(Point{100, 200}, Size{50, 20});
    s2->boundingBox();
    s2->createManipulate()->manipulate();

    return 0;
}
```

`s1`으로 `Line`을 만들고 `boundingBox()` 와 `createManipulate()`를 사용하고 있다. 그리고 `s2`로 `TextShape`를 만들고 똑같이 `boundingBox()` 와 `createManipulate()`를 사용하고 있다. 이렇게 완벽히 연동이 되었다. 

이번에는 객체 어댑터는 어떻게 구현하는지 알아보자.

#### Adapter
```cpp
class TextShape : public Shape{
public: 
    TextShape(shared_ptr<TextView> tv) : tv_(std::move(tv)) {}

    void boundingBox() const override {
        Point o = tv_->getOrigin();
        Size  s = tv_->getExtent();
        Rect r{o.x, o.y, o.x + s.w, o.y + s.h};
        cout << "[TextShape(ObjectAdapter)] origin=(" << o.x << "," << o.y
            << "), extent=(" << s.w << "," << s.h << ") -> boundingBox = "
            << r << "\n";
    }
    
    unique_ptr<Manipulate> createManipulate() const override {
        return make_unique<TextManipulator>();
    }

    bool empty() const { return tv_->isEmpty(); }

private:
    shared_ptr<TextView> tv_;
};
```

`TextShape`는 `TextView`를 공유 포인터로 생성하면서 생성된다. 따라서 `TextView`를 합성하여 인스턴스로 가지고 있다.

#### Client
```cpp
int main() {
    unique_ptr<Shape> s1 = make_unique<Line>(Point{10, 5}, Point{2, 20});
    s1->boundingBox();
    s1->createManipulate()->manipulate();

    auto tv = make_shared<SimpleTextShape>(Point{100, 200}, Size{50, 20});
    unique_ptr<Shape> s2 = make_unique<TextShape>(tv);
    s2->boundingBox();
    s2->createManipulate()->manipulate();

    return 0;
}
```
따라서 먼저 `TextShape`를 생성한 다음 `Shape`에 주입을 해야 한다. 만약에 `TextView` 여러개의 서브클래스로 다양한 기능이 있다고 해보자. 


```cpp
class SimpleTextView : public TextView {
public:
    explicit SimpleTextView(Point origin, Size extent, bool empty = false)
        : empty_(empty)
    {
        setOrigin(origin);
        setExtent(extent);
    }

    bool isEmpty() const override { return empty_; }

private:
    bool empty_{false};
};
```

이렇게 `SimpleTextView`라는 TextView의 기능을 확장해주는 서브클래스를 바로 주입이 가능하다. 그러면 우리는 객체 어댑터를 통해 TextShape를 여러가지 형태로 만들 수 있을 것이다. 이것이 클래스 어댑터에는 없는 객체 어댑터의 장점이다.

---
## 마무리

어댑터 패턴은 서로 다른 인터페이스를 가진 클래스들을 연결해주는 역할을 한다고 보면 된다. 클래스 어댑터는 상속으로, 객체 어댑터는 합성으로 풀어내는데, 결국 상황에 따라 어떤 방식을 선택할지가 달라진다. 내가 글에서 보여준 것처럼, `Shape`와 `TextView`를 연동할 때도 두 가지 방식 모두 동작은 되지만, 유연성과 확장성을 생각하면 **객체 어댑터 쪽이 좀 더 현실적이**라고 할 수 있다.

다음 글에서는 구조 패턴 중에서 `Bridge Pattern`을 소개할 생각이다. 브리지 패턴은 이름처럼 **추상과 구현을 분리해서 독립적으로 확장할 수 있게** 만들어주는 패턴인데, 어댑터 패턴과 비교하면 더 일반화된 구조를 갖는다. 즉, 인터페이스 불일치를 해결하는 게 목적이었던 어댑터와 달리, **브리지는 애초에 확장 가능성을 열어두는 구조 설계**에 초점이 맞춰져 있다

[[참고] Adapter Pattern](https://www.cs.unc.edu/~stotts/GOF/hires/pat4afso.htm)