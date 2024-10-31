# Item 5 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라.
### content
  - [유연하지 않고 테스트하기 어려운 예시](#유연하지-않고-테스트하기-어려운-예시)
    - [정적 유틸리티를 잘못 사용한 예](#정적-유틸리티를-잘못-사용한-예)
    - [싱글턴을 잘못 사용한 예](#싱글턴을-잘못-사용한-예)
  - [의존 객체 주입](#의존-객체-주입)

---
## 유연하지 않고 테스트하기 어려운 예시
### 정적 유틸리티를 잘못 사용한 예
```java
public class SpellChecker {
    private static final Lexicon dictionaty = ...;

    private SpellChecker() {}  // 객체 생성 방지

    public static boolean isValid(String word) { ... }
    public static List<String> suggestions(String typo) { ... }
}
```

### 싱글턴을 잘못 사용한 예
```java
public class SpellChecker {
    private final Lexicon dictionary = ...;

    private SpellChecker(...) {}
    public static SpellChecker INSTANCE = new SpellChecker(...);

    public boolean isValid(String word) { ... }
    public List<String> suggestions(String type) { ... }
}
```

위의 두 예시는 dictionary가 `final`로 선언되어 있다. 때문에 dictionary는 단 하나만 사용할 수 있다.

그렇다면, `SpellChecker`가 여러 사전을 이용할 수 있도록 하려면 어떻게 해야 할까?

## 의존 객체 주입
`의존 객체 주입`이란, 인스턴스를 생성할 때 생성자에 필요한 자원을 넘겨주는 방식이다.

```java
public class SpellChecker {
    private final Lexicon dictionary;

    public SpellChecker(Lexicon dictionary) {
        this.dictionary = Objects.requireNonNull(dictionary);
    }

    public boolean isValid(String word) { ... }
    public List<String> suggestions(String type) { ... }
}
```

위처럼 `dictionary`를 외부에서 주입하면 원하는 사전으로 객체를 생성할 수 있다.

