[Velog로 가기](https://velog.io/@choi-hyk/Design-Pattern-Factory-Method-Pattern)

released at 2025-07-23 18:41:23 KST

updated at 2025-08-30 21:33:17 KST

|[Creational Pattern](https://velog.io/tags/Creational-Pattern)|[Design Pattern](https://velog.io/tags/Design-Pattern)|
|----|----|

## 🛠️ Factory Method Pattern

`Factory Method Pattern`은 **생성 패턴(Creational Pattern)** 중에서 가장 핵심적인 패턴 중 하나다. 핵심은 객체 생성을 단순히 직접 하는 게 아니라, **서브클래스에서 책임지고 생성하도록 위임**한다는 점이다. 이름 그대로 **Factory(공장)**를 통해 객체를 생성하는데, 이때 그 공장(Facility)은 직접 생성하지 않고 **"어떤 제품을 만들지 결정하는 책임만 가진다"**.

#### ✅ 핵심 개념

* 클라이언트는 객체를 직접 생성하지 않는다.
* 객체 생성의 책임을 서브클래스에 위임함으로써 **확장성**과 **유연성**이 커짐.
* 상위 클래스에는 **팩토리 메서드**만 정의하고, **구체적인 생성은 하위 클래스에서 구현**한다.

---

### 언제 사용하나?

보통 프레임워크를 만들 때 자주 쓰인다. 예를 들어 GUI 프레임워크에서 "문서를 여는" 기능이 있다고 하자. 이때 프레임워크는 단순히 `open()` 메서드만 호출하고, 실제로 **Word 문서를 열 것인지 PDF 문서를 열 것인지는 서브클래스에서 결정**한다.

---

### 구조

<img src="https://scvgoe.github.io/img/factory_method.png">

위의 그림을 살펴보면 `Product` 클래스와 `ConcreteProduct`가 존재하는데, `ConcreteProduct`는 여러 개가 존재 가능하다. 일단 이러한 Product를 구현하는 방법은 `Abstract Factory Pattern` 추상 팩토리를 사용한다. 해당 패턴은 다음 글에서 살펴볼 생각이다. 이 두 패턴은 함께 쓰는 것이 정형화되어 있어서 같이 알아두면 좋다.

다시 돌아와서 `Factory Method Pattern`은 `Creator(Factory)`가 `FactoryMethod()`라는 `Product`를 생성하는 함수를 가지고 있다. 그리고 `AnOperation()`을 통해서 `Product`의 동작을 수행 가능한데, 여기서 중요한 점은 `FactoryMethod()`는 그림에서 보면 서브클래스인 `ConcreteCreator`에 위임되어 있다. `ConcreteCreator`는 `ConcreteProduct`의 수에 일치하도록 구현이 되어 있을 것이다.

여기서 `Factory Method Pattern`의 핵심을 알 수 있는데, 특정 `Product`를 선택하여 만드는 것은 `ConcreteCreator`가 `FactoryMethod()`를 통해서 이루어지고, `Product`의 공통적인 행동은 Creator에서 정의된 `AnOperation()`을 통해서 모든 `ConcreteCreator`에서 사용이 가능하다.

생성하는 것은 서브클래스인 `ConcreteCreator`가 만들고 싶은 `Product`를 선택하고, 구체적인 행동은 `Creator`에만 정의하면 된다.

> `Factory Method Pattern`은 생성 책임을 서브클래스에게 넘기고, 상위 클래스는 생성된 객체의 기능만 사용하는 구조이다. 생성과 사용의 분리를 통해 유연한 확장과 변경이 가능하다.

---

### 기본 구현

```cpp
#include <iostream>

class Document {
public:
    virtual void open() = 0;  
    virtual void save() = 0;
    virtual void close() = 0;
    virtual ~Document() = default;
};

class TextDocument : public Document {
public:
    void open() override {
        std::cout << "텍스트 문서를 엽니다." << std::endl;
    }

    void save() override {
        std::cout << "텍스트 문서를 저장합니다." << std::endl;
    }

    void close() override {
        std::cout << "텍스트 문서를 닫습니다." << std::endl;
    }
};

class SpreadsheetDocument : public Document {
public:
    void open() override {
        std::cout << "스프레드시트 문서를 엽니다." << std::endl;
    }

    void save() override {
        std::cout << "스프레드시트 문서를 저장합니다." << std::endl;
    }

    void close() override {
        std::cout << "스프레드시트 문서를 닫습니다." << std::endl;
    }
};

class Creator {
public:
    void runDocument() {
        Document* doc = createDocument();  
        doc->open();
        doc->save();
        doc->close();
        delete doc;  
        std::cout << "문서 작업이 완료되었습니다." << std::endl;
    }

    virtual Document* createDocument() = 0;  
    virtual ~Creator() = default;
};

class TextCreator : public Creator {
public:
    Document* createDocument() override {
        return new TextDocument();
    }
};

class SpreadsheetCreator : public Creator {
public:
    Document* createDocument() override {
        return new SpreadsheetDocument();
    }
};

int main() {
    Creator* creator1 = new TextCreator();
    creator1->runDocument(); 
    delete creator1;

    Creator* creator2 = new SpreadsheetCreator();
    creator2->runDocument();
    delete creator2;

    return 0;
}
```

#### Product

```cpp
class Document {
public:
    virtual void open() = 0;  
    virtual void save() = 0;
    virtual void close() = 0;
    virtual ~Document() = default;
};

class TextDocument : public Document {
public:
    void open() override {
        std::cout << "텍스트 문서를 엽니다." << std::endl;
    }

    void save() override {
        std::cout << "텍스트 문서를 저장합니다." << std::endl;
    }

    void close() override {
        std::cout << "텍스트 문서를 닫습니다." << std::endl;
    }
};

class SpreadsheetDocument : public Document {
public:
    void open() override {
        std::cout << "스프레드시트 문서를 엽니다." << std::endl;
    }

    void save() override {
        std::cout << "스프레드시트 문서를 저장합니다." << std::endl;
    }

    void close() override {
        std::cout << "스프레드시트 문서를 닫습니다." << std::endl;
    }
};
```

`Product`는 텍스트, 스프레드시트 두 가지 종류가 구현되어 있고, `open()`, `save()`, `close()` 세 가지 기능이 존재한다.

#### Creator

```cpp
class Creator {
public:
    void runDocument() {
        Document* doc = createDocument();  
        doc->open();
        doc->save();
        doc->close();
        delete doc;  
        std::cout << "문서 작업이 완료되었습니다." << std::endl;
    }

    virtual Document* createDocument() = 0;  
    virtual ~Creator() = default;
};

class TextCreator : public Creator {
public:
    Document* createDocument() override {
        return new TextDocument();
    }
};

class SpreadsheetCreator : public Creator {
public:
    Document* createDocument() override {
        return new SpreadsheetDocument();
    }
};
```

`Creator`는 `Product`의 세 가지 기능 `open()`, `save()`, `close()`를 모두 사용하는 `runDocument()` 함수를 가진다.

`ConcreteCreator`는 `createDocument()`를 위임받아 종류에 맞는 문서를 생성할 수 있다.

#### Client

```cpp
int main() {
    Creator* creator1 = new TextCreator();
    creator1->runDocument();
    delete creator1;

    Creator* creator2 = new SpreadsheetCreator();
    creator2->runDocument();
    delete creator2;

    return 0;
}
```

`Client`는 두 개의 문서를 정의하여 생성하고 기능까지 사용할 수 있다. 그런데 구현하면서 드는 생각이, 만약에 `ConcreteProduct`가 10개 더 생긴다면, 우리는 확장을 위해서 `ConcreteCreator`를 10개 더 만들어야 할 것이다. 하지만 이를 해결하는 방식이 존재한다.

---

### Template

```cpp
template <typename T>
class GenericCreator {
public:
    void openDocument() {
        Document* doc = createDocument(); 
        doc->open();
        doc->save();
        doc->close();
        delete doc;
    }

    Document* createDocument() {
        return new T();
    }
};
```

이렇게 템플릿 문법을 통해 문서 타입을 지정 가능하게 하면, 서브클래싱을 피할 수 있다.

```java
public class GenericCreator<T extends Document> {

    private final Class<T> clazz;

    public GenericCreator(Class<T> clazz) {
        this.clazz = clazz;
    }

    public void openDocument() {
        try {
            T doc = clazz.getDeclaredConstructor().newInstance();
            doc.open();
            doc.save();
            doc.close();
        } catch (Exception e) {
            throw new RuntimeException("문서를 생성할 수 없습니다.", e);
        }
    }
}
```

자바에서도 마찬가지로 이렇게 Generic 문법을 사용해서 구현이 가능하다.

다시 C++로 돌아와서 살펴보면 고려해야 할 점이 또 있다. 바로 `Product` 객체의 해제를 고려해야 한다.

---

### RAII (Resource Acquisition Is Initialization)

RAII는 **"자원을 얻는 것은 초기화이다"**라는 뜻으로, C++에서 정의된 메모리 관리 기법이다. C++은 개발자에게 메모리 관리를 전적으로 맡긴다. 이에 따라 개발자는 메모리 관리를 자기 마음대로 할 수 있어서 편하지만, 매우 위험할 수도 있다. 따라서 Java의 GC처럼 메모리 관리 기법을 도입하는 문법이 추가되었다.

물론 GC와 RAII는 내부적으로 디테일하게 보면 많이 다르긴 하다.

```cpp
template <typename T>
class GenericCreator {
    static_assert(std::is_base_of<Document, T>::value, "T must be derived from Document");
public:
    void run() {
        std::unique_ptr<Document> doc = std::make_unique<T>(); 
        doc->open();
        doc->save();
        doc->close();
    }
};
```

위 코드를 보면 `unique_ptr()`을 통해서 `doc`을 생성한다. 이때 템플릿 타입으로 `Document`를 받아 템플릿을 적용할 수 있다. `run()` 스코프가 끝나면 `doc`은 자동으로 해제된다. 여기서 중요한 점은 `doc`은 `Document` 객체를 **생성과 동시에 점유하게 된다**. 이것이 RAII의 핵심 개념이다.

자바는 하나의 객체에 여러 개의 객체가 소유하게 될 때, 소유주인 객체가 해제되어도 나머지 객체들은 해제되지 않을 수 있다. 이는 `try-with-resources` 구문으로 예방이 가능하지만, 이는 나중에 알아볼 것이다.

---

### 마무리

`Factory Method Pattern`은 **객체 생성 로직이 복잡하거나, 제품군이 자주 변경/확장되는 상황에서 매우 유용**하다. 하지만 **모든 생성에 적용하면 오히려 유지보수가 어려워지고 테스트도 복잡해질 수 있다.**

다음번에는 위의 Document 예제를 `Abstract Factory Pattern`을 통해 확장시키고 알아보는 시간을 가질 것이다.
