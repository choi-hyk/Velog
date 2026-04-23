[Velog로 가기](https://velog.io/@choi-hyk/Design-Pattern-Builder-Pattern)

released at 2025-07-31 20:28:00 KST

updated at 2026-04-24 02:59:51 KST

|[Creational Pattern](https://velog.io/tags/Creational-Pattern)|[Design Pattern](https://velog.io/tags/Design-Pattern)|
|----|----|

## 🛠️ Builder Pattern

이번에는 `Builder Pattern`에 대해서 알아보겠다. `Builder`라는 단어를 살펴보면 건축, 건축자라는 의미를 가진다. 해당 패턴은 이름 그대로 객체에 대한 **청사진(구성 설계도)** 을 제공하는 패턴이다.

> 복잡한 객체를 생성하는 방법과 표현하는 방법을 별도의 클래스로 분리하여, 서로 다른 표현이라도 동일한 절차로 생성할 수 있도록 한다.

여기서 중요한 점은 **"서로 다른 표현이라도 동일한 절차를 제공할 수 있도록 한다"**는 것이다.

이러한 생성 절차를 제공하는 것이 `Builder` 클래스의 의도이다. 따라서 `Builder` 패턴은 이전에 살펴본 `Factory Method Pattern`이나 `Abstract Factory Pattern`의 `Factory` 클래스와 달리 **생성 방법(과정)을 캡슐화**하여 청사진을 제공하는 것을 목표로 한다.

---

### 언제 사용하나?

GOF 책에서는 RTF(Rich Text Format) 문서 판독기를 예로 들어 설명했다.
문서 판독기는 ASCII 문서나 다른 텍스트 형식으로 변환해 읽을 수 있어야 한다.

이러한 변환 과정을 담당하는 것이 **`Builder` 클래스(TextConverter)** 이고, 변환된 텍스트를 처리하고 조립하는 역할을 하는 것이 **`Director` 클래스(RTFReader)** 이다.

`RTFReader`는 주어진 `ConcreteBuilder`의 변환 메서드를 호출해 동일한 방식으로 파싱을 진행하고, 어떤 포맷으로 변환할지는 Builder(Converter)에 따라 달라진다.

<img width="1206" height="513" alt="Image" src="https://github.com/user-attachments/assets/786bf667-d760-4d13-866c-aae72c7a05f1" />

위의 그림을 보면 설명한 것처럼 `TextConverter`는 변환 메서드들을 인터페이스로 제공하고,
`ASCIIConverter`, `TeXConverter`, `TextWidgetConverter` 등 **구체 클래스(ConcreteBuilder)** 들은 자신만의 로직으로 이 인터페이스를 구현한다.

---

### 구조

Builder 패턴을 구조적으로 표현하면 다음과 같다.

<img width="530" height="280" alt="Image" src="https://github.com/user-attachments/assets/194f0369-9922-4f44-b583-2c29175ff98a" />

핵심 포인트는 **Director가 Builder를 통해 동일한 절차로 객체를 생성하지만, Builder 종류에 따라 최종 결과물이 달라진다**는 것이다.

---

### 구현 방법

#### 전체 코드

```cpp
#include <iostream>
#include <string>
#include <memory>
using namespace std;

// ======= Product =======
class ASCIIText {
public:
    string content;
};
class TeXText {
public:
    string content;
};
class TextWidget {
public:
    string widgetData;
};

// ======= Builder 인터페이스 =======
class TextConverter {
public:
    virtual ~TextConverter() {}
    virtual void ConvertCharacter(char c) = 0;
    virtual void ConvertFontChange(const string& font) = 0;
    virtual void ConvertParagraph() = 0;
};

// ======= ConcreteBuilder: ASCII =======
class ASCIIConverter : public TextConverter {
    shared_ptr<ASCIIText> ascii = make_shared<ASCIIText>();
public:
    void ConvertCharacter(char c) override {
        ascii->content += c;
    }
    void ConvertFontChange(const string& font) override {
        // ASCII에서는 폰트 개념이 없으므로 무시
    }
    void ConvertParagraph() override {
        ascii->content += "\n";
    }
    shared_ptr<ASCIIText> GetASCIIText() { return ascii; }
};

// ======= ConcreteBuilder: TeX =======
class TeXConverter : public TextConverter {
    shared_ptr<TeXText> tex = make_shared<TeXText>();
public:
    void ConvertCharacter(char c) override {
        tex->content += c;
    }
    void ConvertFontChange(const string& font) override {
        tex->content += "\\font{" + font + "}";
    }
    void ConvertParagraph() override {
        tex->content += "\n\n";
    }
    shared_ptr<TeXText> GetTeXText() { return tex; }
};

// ======= Director =======
class RTFReader {
    TextConverter* builder; 
public:
    RTFReader(TextConverter* b) : builder(b) {}

    void ParseRTF() {
        string tokens = "ABC\nFONT Times\nDEF\n";
        for (size_t i = 0; i < tokens.size(); ++i) {
            char t = tokens[i];
            if (t == '\n') {
                builder->ConvertParagraph();
            } else if (t == 'F') { 
                builder->ConvertFontChange("Times");
                i += 10; 
            } else {
                builder->ConvertCharacter(t);
            }
        }
    }
};

// ======= 클라이언트 =======
int main() {
    // ASCII로 변환
    ASCIIConverter asciiBuilder;
    RTFReader reader1(&asciiBuilder);
    reader1.ParseRTF();
    cout << "[ASCII 결과]" << endl;
    cout << asciiBuilder.GetASCIIText()->content << endl;

    // TeX으로 변환
    TeXConverter texBuilder;
    RTFReader reader2(&texBuilder);
    reader2.ParseRTF();
    cout << "\n[TeX 결과]" << endl;
    cout << texBuilder.GetTeXText()->content << endl;

    return 0;
}
```
#### Converter

```cpp
class TextConverter {
public:
    virtual ~TextConverter() {}
    virtual void ConvertCharacter(char c) = 0;
    virtual void ConvertFontChange(const string& font) = 0;
    virtual void ConvertParagraph() = 0;
};

class ASCIIConverter : public TextConverter {
    shared_ptr<ASCIIText> ascii = make_shared<ASCIIText>();
public:
    void ConvertCharacter(char c) override {
        ascii->content += c;
    }
    void ConvertFontChange(const string& font) override {
        // ASCII에서는 폰트 개념이 없으므로 무시
    }
    void ConvertParagraph() override {
        ascii->content += "\n";
    }
    shared_ptr<ASCIIText> GetASCIIText() { return ascii; }
};

class TeXConverter : public TextConverter {
    shared_ptr<TeXText> tex = make_shared<TeXText>();
public:
    void ConvertCharacter(char c) override {
        tex->content += c;
    }
    void ConvertFontChange(const string& font) override {
        tex->content += "\\font{" + font + "}";
    }
    void ConvertParagraph() override {
        tex->content += "\n\n";
    }
    shared_ptr<TeXText> GetTeXText() { return tex; }
};
```

`ASCIIConverter`는 `ConvertFontChange()`를 무시하는데, ASCII 문서에는 폰트 개념이 없기 때문이다.
이처럼 **각 Builder는 자신이 생성하는 결과물(Product)에 따라 로직을 달리 구현**한다.

#### Director

```cpp
class RTFReader {
    TextConverter* builder; 
public:
    RTFReader(TextConverter* b) : builder(b) {}

    void ParseRTF() {
        string tokens = "ABC\nFONT Times\nDEF\n";
        for (size_t i = 0; i < tokens.size(); ++i) {
            char t = tokens[i];
            if (t == '\n') {
                builder->ConvertParagraph();
            } else if (t == 'F') { 
                builder->ConvertFontChange("Times");
                i += 10; 
            } else {
                builder->ConvertCharacter(t);
            }
        }
    }
};
```

`Director` 클래스는 텍스트를 읽고 **토큰의 종류에 따라 Builder의 메서드를 호출**하여 Product를 조립한다.

### Client

```cpp
int main() {
    ASCIIConverter asciiBuilder;
    RTFReader reader1(&asciiBuilder);
    reader1.ParseRTF();
    cout << "[ASCII 결과]" << endl;
    cout << asciiBuilder.GetASCIIText()->content << endl;

    TeXConverter texBuilder;
    RTFReader reader2(&texBuilder);
    reader2.ParseRTF();
    cout << "\n[TeX 결과]" << endl;
    cout << texBuilder.GetTeXText()->content << endl;

    return 0;
}
```

Client는 Builder를 생성해 Director에 주입하고, 파싱 작업을 요청할 뿐이다. **생성과정과 사용을 완전히 분리**할 수 있다는 것이 큰 장점이다.

---

### 마무리

이렇게 `Builder Pattern`은 복잡한 객체 생성을 단계별로 분리해 코드 가독성과 유지보수성을 높인다. 그리고 동일한 절차로 서로 다른 결과물을 만들 수 있다. 그러나 남용할 경우 클래스 수(Builder, Director)가 많아져 구조가 복잡해질 수 있다. 또한 `Director`와 `Builder` 간의 의존 관계 설정이 필요해 초기 설계 비용이 발생한다.

다음 글에서는 `Creational Pattern` 에서 마지막으로 다룰 패턴인 `Prototype Pattern`을 다뤄보겠다.`Prototype Pattern`은 **복잡한 객체를 새로 만드는 대신 기존 객체를 복제(clone)하여 생성하는 방식**을 제공하는 패턴이다.
