[Velog로 가기](https://velog.io/@choi-hyk/Design-Pattern-Abstract-Factory-Pattern)

released at 2025-07-27 18:53:38 KST

updated at 2025-11-04 10:53:21 KST

|[Creational Pattern](https://velog.io/tags/Creational-Pattern)|[Design Pattern](https://velog.io/tags/Design-Pattern)|
|----|----|

## 🛠️ Abstract Factory Pattern

`Abstract Factory Pattern`은 생성 패턴(Creational Pattern) 중에서 **제품군(Product family)** 을 생성할 때 사용하는 패턴이다.
팩토리 메서드 패턴이 **하나의 객체 생성 책임**만 서브클래스에 넘긴다면, 추상 팩토리 패턴은 **서로 연관된 여러 객체(제품군)를 한 번에 묶어서 생성**할 수 있도록 해준다.

이 패턴은 특히 **룩앤필(Look & Feel)** 이나 **OS 플랫폼에 따라 UI 구성 요소를 통일감 있게 생성**해야 할 때 강력하게 쓰인다.

---

### ✅ 핵심 개념

* **서브클래스에게 객체 생성 책임을 위임**한다는 점은 Factory Method와 같음
* 하지만 단일 객체가 아닌 **여러 제품군을 한 번에 생성**하도록 묶어서 관리
* 클라이언트는 **제품군의 종류만 결정하고, 구체적인 객체 생성 코드는 팩토리에 위임**

---

### 언제 사용하나?

대표적인 예시가 **UI 테마/플랫폼**이다.
GOF 책에서는 "룩앤필" 관점으로 모티프 테마와 프레젠테이션 매니저의 차이점을 두어 설명한다.

> 모티프는 X 윈도우 환경에서 UI 위젯의 스타일/룩앤필을 제공하는 GUI 툴킷이고, 프레젠테이션 매니저는 OS/2 운영체제의 전체 GUI 시스템 (윈도우 시스템 그 자체)

 라고한다...
 
 책에서는 위의 과거 운영체제에 사용된 대표적인 두가지 테마로 `Abstract Factory Pattern` 를 설명한다.



#### Presentation Manager
<img src="https://upload.wikimedia.org/wikipedia/en/thumb/0/08/Os2-1.1-desktop.png/500px-Os2-1.1-desktop.png">

#### Motif Theme
<img src="https://upload.wikimedia.org/wikipedia/commons/thumb/5/58/Plan_Open_Motif_screenshot.png/500px-Plan_Open_Motif_screenshot.png">

보이는 것처럼 두개의 제품 Motif와 Presentation Manager은 사용자 인터페이스인데, 테마가 다르다. 따라서 공통적인 Widget을 서브 클래스에 위임하여 결합도를 낮출 필요가 있다.

> 추상 팩토리(Abstract Factory) 패턴은 서로 연관되거나 비슷한 성격을 가진 두 개 이상의 제품(클래스)이 함께 동작해야 할 때, 이들을 하나의 '제품군'으로 묶어서 일관성 있게 생성하는 책임을 가진다.

---

### 구조

<img src="https://scvgoe.github.io/img/abstract_factory_structure.png">

추상 팩토리 패턴은 다음과 같은 4가지 요소로 구성된다:

1. **AbstractFactory** : 제품군을 생성하는 인터페이스
2. **ConcreteFactory** : 실제 제품군 생성 구현체
3. **AbstractProduct** : 제품의 추상 인터페이스
4. **ConcreteProduct** : 구체적인 제품 클래스

팩토리 메서드 패턴과 큰 차이는, **팩토리 메서드가 하나의 제품만 생성하는 데 비해 추상 팩토리는 여러 제품군을 다룬다는 점**이다.

위의 그림을 보면 제품군을 생성하는 `AbstractFactory` 객체는 `CreateProductA()`, `CreateProductB()` 라는 메소드를 인터페이스로 가지고 있다. `ConcreateFactory` 는 

위에서 예시로 든 Motif와 Presentation Manager는 `Factory`가 될 것이고 `Prodcut` 는 각 테마에서 제공하는 `ScrollBar` 또는 `Window` 같은 위젯 컴포넌트가 될것이다. 두 테마 모두 유사한 위젯을 제공하므로 하위 클래스인 `ConcreteFactory`에게 생성 메소드를 위임하여 결합도를 낮춰준다. 그리고 `ConcreteFactory`는 구체적인 생성 메소드로 구현된 `AbstractProduct` 의 `ConcreteProduct` 생성을 하게 된다.

따라서 `AbstratProduct` 가 많아져도 `AbstractFactory`에 손쉽게 이식이 가능하다. 그리고 `ConcreteFactory`가 많아져도 `ConcreteProduct`에 해당 `Factory` 의 제품을 구현하여 쉽게 확장이 가능하다.
 
---

### 기본 구현

저번에 살펴본 `Factory Method Pattern`에서 사용한 예제인 Document 예제를 활용하겠다. `Microsoft`와 `Mac` Factory 두개로 구현하고, 각 제품은 `TextDocument`와 `SpreadSheet` 두개로 구현하겠다.

```cpp
#include <iostream>
#include <memory>
using namespace std;

// ===== Abstract Product =====
class Document {
public:
    virtual void open() = 0;
    virtual void save() = 0;
    virtual void close() = 0;
    virtual ~Document() = default;
};

// ===== Concrete Product (MS) =====
class MsTextDocument : public Document {
public:
    void open() override  { cout << "[MS] 텍스트 문서를 엽니다.\n"; }
    void save() override  { cout << "[MS] 텍스트 문서를 저장합니다.\n"; }
    void close() override { cout << "[MS] 텍스트 문서를 닫습니다.\n"; }
};

class MsSpreadsheetDocument : public Document {
public:
    void open() override  { cout << "[MS] 스프레드시트 문서를 엽니다.\n"; }
    void save() override  { cout << "[MS] 스프레드시트 문서를 저장합니다.\n"; }
    void close() override { cout << "[MS] 스프레드시트 문서를 닫습니다.\n"; }
};

// ===== Concrete Product (Mac) =====
class MacTextDocument : public Document {
public:
    void open() override  { cout << "[Mac] 텍스트 문서를 엽니다.\n"; }
    void save() override  { cout << "[Mac] 텍스트 문서를 저장합니다.\n"; }
    void close() override { cout << "[Mac] 텍스트 문서를 닫습니다.\n"; }
};

class MacSpreadsheetDocument : public Document {
public:
    void open() override  { cout << "[Mac] 스프레드시트 문서를 엽니다.\n"; }
    void save() override  { cout << "[Mac] 스프레드시트 문서를 저장합니다.\n"; }
    void close() override { cout << "[Mac] 스프레드시트 문서를 닫습니다.\n"; }
};

// ===== Abstract Factory =====
class AbstractFactory {
public:
    virtual unique_ptr<Document> createTextDocument() = 0;
    virtual unique_ptr<Document> createSpreadsheetDocument() = 0;
    virtual ~AbstractFactory() = default;
};

// ===== Concrete Factories =====
class MicrosoftFactory : public AbstractFactory {
public:
    unique_ptr<Document> createTextDocument() override {
        return make_unique<MsTextDocument>();
    }
    unique_ptr<Document> createSpreadsheetDocument() override {
        return make_unique<MsSpreadsheetDocument>();
    }
};

class MacFactory : public AbstractFactory {
public:
    unique_ptr<Document> createTextDocument() override {
        return make_unique<MacTextDocument>();
    }
    unique_ptr<Document> createSpreadsheetDocument() override {
        return make_unique<MacSpreadsheetDocument>();
    }
};

// ===== Client =====
int main() {
    // 1. Microsoft 환경
    unique_ptr<AbstractFactory> factory = make_unique<MicrosoftFactory>();
    auto doc1 = factory->createTextDocument();
    auto doc2 = factory->createSpreadsheetDocument();

    doc1->open();
    doc1->save();
    doc1->close();

    doc2->open();
    doc2->save();
    doc2->close();

    cout << "---------------------------\n";

    // 2. Mac 환경
    factory = make_unique<MacFactory>();
    auto doc3 = factory->createTextDocument();
    auto doc4 = factory->createSpreadsheetDocument();

    doc3->open();
    doc3->save();
    doc3->close();

    doc4->open();
    doc4->save();
    doc4->close();

    return 0;
}
```

#### Abstract Product

```cpp
#include <iostream>
#include <memory>
using namespace std;

#include <iostream>
#include <memory>
using namespace std;

class Document {
public:
    virtual void open() = 0;
    virtual void save() = 0;
    virtual void close() = 0;
    virtual ~Document() = default;
};
```

먼저 `Document` 객체는 저번에 `Factory Method Pattern` 으로 공통적인 기능 3개를 포함한다.

#### Concrete Product

```cpp
class MsTextDocument : public Document {
public:
    void open() override  { cout << "[MS] 텍스트 문서를 엽니다.\n"; }
    void save() override  { cout << "[MS] 텍스트 문서를 저장합니다.\n"; }
    void close() override { cout << "[MS] 텍스트 문서를 닫습니다.\n"; }
};

class MsSpreadsheetDocument : public Document {
public:
    void open() override  { cout << "[MS] 스프레드시트 문서를 엽니다.\n"; }
    void save() override  { cout << "[MS] 스프레드시트 문서를 저장합니다.\n"; }
    void close() override { cout << "[MS] 스프레드시트 문서를 닫습니다.\n"; }
};

class MacTextDocument : public Document {
public:
    void open() override  { cout << "[Mac] 텍스트 문서를 엽니다.\n"; }
    void save() override  { cout << "[Mac] 텍스트 문서를 저장합니다.\n"; }
    void close() override { cout << "[Mac] 텍스트 문서를 닫습니다.\n"; }
};

class MacSpreadsheetDocument : public Document {
public:
    void open() override  { cout << "[Mac] 스프레드시트 문서를 엽니다.\n"; }
    void save() override  { cout << "[Mac] 스프레드시트 문서를 저장합니다.\n"; }
    void close() override { cout << "[Mac] 스프레드시트 문서를 닫습니다.\n"; }
};
```

구체적인 제품으로 `TextDocument`, `SpreadSheetDocument` 두개를 각각 `ConcreteFactory`인 `Ms` 와 `Mac`으로 구현 중이다.


#### Abstract Factory

```cpp
class AbstractFactory {
public:
    virtual unique_ptr<Document> createTextDocument() = 0;
    virtual unique_ptr<Document> createSpreadsheetDocument() = 0;
    virtual ~AbstractFactory() = default;
};
```
#### Concrete Factory

```cpp
class MicrosoftFactory : public AbstractFactory {
public:
    unique_ptr<Document> createTextDocument() override {
        return make_unique<MsTextDocument>();
    }
    unique_ptr<Document> createSpreadsheetDocument() override {
        return make_unique<MsSpreadsheetDocument>();
    }
};

class MacFactory : public AbstractFactory {
public:
    unique_ptr<Document> createTextDocument() override {
        return make_unique<MacTextDocument>();
    }
    unique_ptr<Document> createSpreadsheetDocument() override {
        return make_unique<MacSpreadsheetDocument>();
    }
};
```

저번과 마찬가지로 RAII로 객체를 관리한다.

#### Client

```cpp
int main() {
    // 1. Microsoft 환경
    unique_ptr<AbstractFactory> factory = make_unique<MicrosoftFactory>();
    auto doc1 = factory->createTextDocument();
    auto doc2 = factory->createSpreadsheetDocument();

    doc1->open();
    doc1->save();
    doc1->close();

    doc2->open();
    doc2->save();
    doc2->close();

    cout << "---------------------------\n";

    // 2. Mac 환경
    factory = make_unique<MacFactory>();
    auto doc3 = factory->createTextDocument();
    auto doc4 = factory->createSpreadsheetDocument();

    doc3->open();
    doc3->save();
    doc3->close();

    doc4->open();
    doc4->save();
    doc4->close();

    return 0;
}
```

#### 차이점 (Factory Method → Abstract Factory)

* **Factory Method 패턴**: `Document` **단일 제품**만 생성
* **Abstract Factory 패턴**:
  `Document`, `Viewer` 등 **연관된 여러 제품군을 묶어서 생성**

이 예제처럼 팩토리 메서드 예제를 확장하면 **추상 팩토리 패턴**이 된다.

---

### 마무리

Abstract Factory Pattern은 서로 연관된 객체들을 그룹으로 생성할 때 강력한 패턴이다.
단일 객체만 필요하면 Factory Method Pattern으로 충분하지만, 여러 객체의 스타일이나 버전 일관성을 유지하려면 추상 팩토리 패턴이 훨씬 유리하다.

다음 글에서는 Builder Pattern을 통해 복잡한 객체를 단계별로 생성하는 방식을 살펴볼 예정이다.



