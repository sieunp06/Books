# Item 3 private 생성자나 열거 타입으로 싱글턴임을 보증하라.
### content
  - [싱글턴(singleton)](#싱글턴singleton)
      - [💡 싱글턴의 예시](#-싱글턴의-예시)
  - [싱글턴을 만드는 방법](#싱글턴을-만드는-방법)
    - [✏ public static final 필드 방식](#-public-static-final-필드-방식)
    - [✏ 정적 팩토리 메서드 방식](#-정적-팩토리-메서드-방식)
    - [✏ 원소가 하나인 열거 타입 선언 방식](#-원소가-하나인-열거-타입-선언-방식)
  - [싱글턴을 깨뜨리는 방법](#싱글턴을-깨뜨리는-방법)
    - [reflection](#reflection)
    - [직렬화 역직렬화](#직렬화-역직렬화)

---
## 싱글턴(singleton)
> 인스턴스를 오직 하나만 생성할 수 있는 클래스

#### 💡 싱글턴의 예시
- 함수와 같은 무상태(stateless) 객체
- 설계상 유일해야 하는 시스템 컴포넌트

## 싱글턴을 만드는 방법
### ✏ public static final 필드 방식

```java
public class Elvis {
    public static final Elvis INSTANCE = new Elvis();
    private Elvis() { ... }

    public void leaveTheBuilding() { ... }
}
```
`private` 생성자는 `public static final` 필드인 `Elvis.INSTANCE`를 초기화할 때 딱 한 번만 호출된다.

`public`이나 `protected` 생성자가 없기 때문에 `Elvis`가 전체 시스템에서 하나 뿐임을 보장한다.

#### 👍 장점
- 해당 클래스가 싱글턴임을 API에 명백히 드러남.<br>
  `public static final`이기 때문에 절대 다른 객체를 참조할 수 없다.
- 간결함.

### ✏ 정적 팩토리 메서드 방식
```java
public class Elvis {
    private static final Elvis INSTANCE = new Elvis();
    private Elvis() { ... }
    public static Elvis getInstance() {
        return INSTANCE;
    }

    public void leaveTheBuilding() { ... }
}
```
`Elvis.getInstance`는 항상 같은 객체를 참조를 반환하기 때문에 새로운 인스턴스가 만들어질 일이 없다.

#### 👍 장점
- API를 바꾸지 않고도 싱글턴이 아니게 변경할 수 있다.
- 원한다면 제네릭 싱글턴 팩토리로 만들 수 있다.
- 정적 팩토리의 메서드 참조를 공급자(supplier)로 사용할 수 있다.
  - `Elvis::getInstance`를 `Supplier<Elvis>`로 사용하는 식이다.

> 📌 [제네릭 싱글턴 팩토리 - Item 30 이왕이면 제네릭 메서드로 만들라.]()

### ✏ 원소가 하나인 열거 타입 선언 방식
```java
public enum Elvis {
    INSTANCE;

    public void leaveTheBuilding() { ... }
}
```

원소가 하나인 열거 타입은 `public` 필드 방식과 비슷하지만, 더 간결하고 추가적인 노력 없이 직렬화 또한 가능하다.

해당 방법은 아래에서 설명할 싱글턴을 깨는 방법인 직렬화 상황과 리플렉션 공격에서 제 2의 인스턴스가 생기는 것을 막아준다.

때문에 대부분의 상황에서 원소가 하나뿐인 열거 타입이 싱글턴을 만드는 가장 좋은 방법이라고 할 수 있다.

## 싱글턴을 깨뜨리는 방법
`private`을 활용한 두 가지의 방법은 아래 상황에서 새로운 인스턴스가 생성될 수 있다.

### reflection
> 클래스 정보에 접근할 수 있도록 하는 API

`reflection`을 사용하면 private으로 선언된 생성자에 접근하여 강제로 호출할 수 있다.

때문에 reflection을 사용하면 싱글턴이 깨질 수 있다.

### 직렬화 역직렬화
싱글턴 패턴에서 직렬화 가능한 클래스가 되기 위해 `Serializable` 인터페이스를 구현하는 순간, 싱글턴 클래스가 아니게 된다.