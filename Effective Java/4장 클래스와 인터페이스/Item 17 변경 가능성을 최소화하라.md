# Item 17 변경 가능성을 최소화하라
### content
  - [❓불변 클래스](#불변-클래스)
  - [불변 클래스를 만들기 위한 5가지 규칙](#불변-클래스를-만들기-위한-5가지-규칙)
    - [1. 객체의 상태를 변경하는 메서드(변경자)를 제공하지 않는다.](#1-객체의-상태를-변경하는-메서드변경자를-제공하지-않는다)
    - [2. 클래스를 확장할 수 없도록 한다.](#2-클래스를-확장할-수-없도록-한다)
      - [✨클래스 확장을 막는 방법 1: 클래스를 `final`로 선언한다.](#클래스-확장을-막는-방법-1-클래스를-final로-선언한다)
      - [✨클래스 확장을 막는 방법 2: `정적 팩터리 메서드`를 사용한다.](#클래스-확장을-막는-방법-2-정적-팩터리-메서드를-사용한다)
    - [3. 모든 필드를 final로 선언한다.](#3-모든-필드를-final로-선언한다)
    - [4. 모든 필드를 private으로 선언한다.](#4-모든-필드를-private으로-선언한다)
    - [5. 자신 외에는 내부의 가변 컴포넌트에 접근할 수 없도록 한다.](#5-자신-외에는-내부의-가변-컴포넌트에-접근할-수-없도록-한다)
  - [불변 복소수 클래스](#불변-복소수-클래스)
  - [불변 객체](#불변-객체)
    - [📌 불변 객체의 장점](#-불변-객체의-장점)
      - [👍 불변 객체는 생성된 시점의 상태를 파괴될 때까지 그대로 간직한다.](#-불변-객체는-생성된-시점의-상태를-파괴될-때까지-그대로-간직한다)
      - [👍 불변 객체는 근본적으로 스레드 안전하여 따로 동기화할 필요가 없다.](#-불변-객체는-근본적으로-스레드-안전하여-따로-동기화할-필요가-없다)
      - [👍 불변 객체는 자유롭게 공유할 수 있음을 물론, 불변 객체끼리는 내부 데이터를 공유할 수 있다.](#-불변-객체는-자유롭게-공유할-수-있음을-물론-불변-객체끼리는-내부-데이터를-공유할-수-있다)
      - [👍 불변 객체는 그 자체로 실패 원자성을 제공한다.](#-불변-객체는-그-자체로-실패-원자성을-제공한다)
    - [📌 불변 객체의 단점](#-불변-객체의-단점)
      - [👎 값이 다르면 반드시 독립된 객체로 만들어야 한다.](#-값이-다르면-반드시-독립된-객체로-만들어야-한다)
  - [✨ 정리](#-정리)


---
## ❓불변 클래스
불변 클래스란 그 인스턴스 내부 값을 수정할 수 없는 클래스이며, 불변 인스턴스에 설정된 정보는 고정되어 객체가 파괴되는 순간까지 절대 달라지지 않는다.

이러한 불변 클래스는 가변 클래스보다 설계하고 구현하기 쉬우며, 오류가 생길 여지가 적고 안전하다.

자바 라이브러리 안에는 다음과 같이 다양한 불변 클래스가 있다.
- String
- 기본 타입의 박싱된 클래스들
- BigInteger, BigDecimal

## 불변 클래스를 만들기 위한 5가지 규칙
### 1. 객체의 상태를 변경하는 메서드(변경자)를 제공하지 않는다.
즉, 세터(setter) 메서드를 제공하지 않는다. 

세터 메서드를 제공하면 세터 메서드를 통해 상태가 변경될 수 있기 떄문에 불변 클래스로 만들 수 없다.

아래 `Drink` 클래스는 세터 메서드를 제공하는 예시이다.
```java
public class Drink {
    private String name;

    public Drink(final String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }

    // setter 메서드
    public void setName(String name) { 
        this.name = name;
    }
}
```
```java
Drink cola = new Drink("cola");
System.out.println(cola.getName());     // cola;
cola.setName("soda");  
System.out.println(cola.getName());     // soda;
```
`setName` 메서드를 사용하면 `name` 필드의 상태가 변경되기 때문에 위 클래스를 불변으로 만들 수 없다.

### 2. 클래스를 확장할 수 없도록 한다.
클래스를 확장할 수 없도록 하면 하위 클래스에서 부주의하게 or 나쁜 의도로 객체의 상태를 변하게 만드는 사태를 막는다.

#### ✨클래스 확장을 막는 방법 1: 클래스를 `final`로 선언한다.
상속을 막는 대표적인 방법 중 하나는 클래스를 `final`로 선언하는 방법이다.

```java
public class Drink {
    private String name;

    public Drink(final String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }
}
```
위 예시는 세터 메서드가 없기 때문에 `1. 객체의 상태를 변경하는 메서드(변경자)를 제공하지 않는다.`를 만족하지만 생성자가 `public`이기 떄문에 클래스를 확장하면서 상태가 변경된 것처럼 사용 가능하다.

```java
public class Cola extends Drink {
    private String name;

    public Cola(final String name) {
        super(name);
        this.name = "Drink" + name;
    }

    @Override
    public String getName() {
        return name;
    }
}
```
```java
Drink cocaCola = new Cola("cola");
System.out.println(cocaCola.getName()); // Drinkcola
```

위 예시는 설명한 것처럼 `Drink`를 확장하면서 상태가 변경된 것처럼 사용된다.

이렇게 클래스를 확장하면서 상태가 변경될 수 있도록 변화하는 경우가 있기 때문에 아래 `LocalDate` 클래스처럼 클래스 자체를 `final`로 선언하여 확장을 방지하고 있다.

```java
public final class LocalDate {
    // ...
}
```

#### ✨클래스 확장을 막는 방법 2: `정적 팩터리 메서드`를 사용한다.
클래스 확장을 막는 방법으로 `정적 팩터리 메서드`가 쓰일 수 있으며, 이 방법은 클래스를 `final`로 선언하는 것보다 유연한 방법이다.

```java
public class Drink {
    private String name;

    Drink(final String name) {
        this.name = name;
    }

    public static Drink of(final String name) {
        return new Drink(name);
    }

    public String getName() {
        return name;
    }
}
```

정적 팩터리 메서드를 사용하면, package 외부에서는 `Drink` 생성자가 `final`로 선언된 것과 동일하게 동작한다.

> 📌 [정적 팩터리 메서드 - Item 1 생성자 대신 정적 팩터리 메서드를 고려하라](https://github.com/sieunp06/Books/blob/main/Effective%20Java/2%EC%9E%A5%20%EA%B0%9D%EC%B2%B4%20%EC%83%9D%EC%84%B1%EA%B3%BC%20%ED%8C%8C%EA%B4%B4/Item%201%20%EC%83%9D%EC%84%B1%EC%9E%90%20%EB%8C%80%EC%8B%A0%20%EC%A0%95%EC%A0%81%20%ED%8C%A9%ED%84%B0%EB%A6%AC%20%EB%A9%94%EC%84%9C%EB%93%9C%EB%A5%BC%20%EA%B3%A0%EB%A0%A4%ED%95%98%EB%9D%BC.md)

### 3. 모든 필드를 final로 선언한다.
모든 필드를 `final`로 선언하는 방법은 시스템이 강제하는 수단을 이용해 설계자의 의도를 명확히 드러낸다.

```java
public class Drink {
    public final String name;

    public Drink(final String name) {
        this.name = name;
    }
}
```
위 예시의 필드는 `public final`로 선언되었기 때문에 설계자 입장에서 해당 필드가 변경 불가능하다는 것임을 알려준다.

### 4. 모든 필드를 private으로 선언한다.
필드가 참조하는 가변 객체를 클라이언트에서 직접 접근해 수정하는 일을 막아준다.

기본 타입 필드나 불변 객체를 참조하는 필드를 `public final`로만 선언해도 불변 객체를 만들 수 있지만, 이런 방법을 사용하면 다음 릴리즈에서 내부 표현을 바꾸지 못하기 때문에 추천되지 않는 방법이다.

`3. 모든 필드를 final로 선언한다.`에서 사용된 예시를 보면 `public`으로 선언되었기 때문에 필드에 직접 접근이 가능하다.

```java
public class Drink {
    public final String name;

    public Drink(final String name) {
        this.name = name;
    }
}
```
```java
Drink cocaCola = new Cola("cola");
System.out.println(cocaCola.name);
```
이런 경우 `public final` 필드를 수정할 경우 파급효과가 크게 때문에 추천하지 않고, 대신 필드를 `private`로 선언하는 것을 권장한다.

> 📌 [클래스의 접근 권한 - Item 15 클래스와 멤버의 접근 권한을 최소화하라]()

> 📌 [getter 메서드 - Item 16 public 클래스에서는 public 필드가 아닌 접근자 메서드를 사용하라]()

### 5. 자신 외에는 내부의 가변 컴포넌트에 접근할 수 없도록 한다.
클래스에서 가변 객체를 참조하는 필드가 하나라도 있다면 클라이언트에서 그 객체의 참조를 얻을 수 없도록 해야 한다.

이런 필드는 절대 클라이언트가 제공한 객체 참조를 가리키게 해서는 안된다.

또한, 접근자 메서드가 그 필드를 그대로 반환해서도 안된다.

생성자, 접근자, readObject 메서드에서 방어적 복사를 수행해야 한다.

> 📌 [readObject 메서드 - Item 88 readObject 메서드는 방어적으로 작성하라]()

## 불변 복소수 클래스
아래 예시 코드는 복소수(실수부와 허수부로 구성된 수)를 표현한다.
```java
public final class Complex {
    private final double re;    // 실수부
    private final double im;    // 허수부

    public Complex(double re, double im) {
        this.re = re;
        this.im = im;
    }

    // 접근자 메서드
    public double realPart() {
        return re;
    }

    public double imaginaryPart() {
        return im;
    }

    // 사칙연산 메서드 plus, minus, times, dividedBy
    public Complex plus(Complex c) {
        return new Complex(re + c.re, im + c.im);
    }

    public Complex minus(Complex c) {
        return new Complex(re - c.re, im - c.im);
    }

    public Complex times(Complex c) {
        return new Complex(re * c.re - im * c.im, re * c.im + im * c.re);
    }

    public Complex dividedBy(Complex c) {
        double tmp = c.re * c.re + c.im * c.im;
        return new Complex((re * c.re + im * c.im) / tmp, (im * c.re - re * c.im) / tmp);
    }

    @Override
    public boolean equals(Object o) {
        if (o == this) {
            return true;
        }
        if (!(o instanceof Complex)) {
            return false;
        }
        Complex c = (Complex) o;

        return Double.compare(c.re, re) == 0 && Double.compare(c.im, im) == 0;
    }

    @Override
    public int hashCode() {
        return 31 * Double.hashCode(re) + Double.hashCode(im);
    }

    @Override
    public String toString() {
        return "(" + re + " + " + im + "i)";
    }
}
```

위 예시에서 `plus`, `minus`, `times`, `dividedBy`와 같은 사칙연산 메서드에서는 정적 팩토리 메서드를 통해 확장을 막았다.

```java
public Complex plus(Complex c) {
    return new Complex(re + c.re, im + c.im);
}
```

이는 피연산자에 함수를 적용해 그 결과를 반환하지만 피연산자 자체는 그대로인 프로그래밍 패턴으로, **함수형 프로그래밍**이라 한다.

함수형 프로그래밍을 적용하면 코드에서 불변이 되는 영역의 비율이 높아진다.

## 불변 객체
### 📌 불변 객체의 장점
> 불변 객체는 단순하다.

#### 👍 불변 객체는 생성된 시점의 상태를 파괴될 때까지 그대로 간직한다.
- 모든 생성자가 클래스 불변식(class invariant)을 보장한다면 그 클래스를 사용하는 프로그래머가 다른 노력을 들이지 않더라도 영원히 불변으로 남는다.
- 변경자 메서드가 일으키는 상태 전이를 정밀하게 문서를 남겨 놓지 않은 가변 클래스는 믿고 사용하기 어려울 수 있다.
#### 👍 불변 객체는 근본적으로 스레드 안전하여 따로 동기화할 필요가 없다.
- 여러 스레드가 동시에 사용해도 절대 훼손되지 않는다.
- 때문에 불변 객체는 안심하고 공유할 수 있으며, 이 때문에 한번 만든 인스턴스를 최대한 재활용하기를 권한다.
  ```java
  public static final Complex ZERO = new Complex(0, 0);
  public static final Complex ONE = new Complex(1, 0);
  ```
  자주 사용되는 인스턴스를 캐싱하여 같은 인스턴스를 중복 생성하지 않게 해주는 정적 팩토리를 제공할 수 있다.
- 불변 객체를 자유롭게 공유할 수 있다는 점은 방어적 복사도 필요 없게 한다.
  - 그러니 불변 클래스는 `clone` 메서드나 복사 생성자를 제공하지 않는 것이 좋다.

> 📌 [방어적 복사 - Item 50 적시에 방어적 복사본을 만들어라.]()

> 📌 [복사 생성자 - Item 13 clone의 재정의는 주의해서 진행하라.]()
#### 👍 불변 객체는 자유롭게 공유할 수 있음을 물론, 불변 객체끼리는 내부 데이터를 공유할 수 있다.
- negate 메서드에서는 크기가 같고 부호가 반대인 BigInteger를 생성하는데, 비록 가변이지만 복사하지 않고 원본 인스턴스와 공유해도 된다.

#### 👍 불변 객체는 그 자체로 실패 원자성을 제공한다.
> 📌 [readObject 메서드 - Item 76 readObject 메서드는 방어적으로 작성하라]()
- 상태가 절대 변하지 않기 때문에 잠깐이라도 불일치 상태에 빠질 가능성이 없다.

### 📌 불변 객체의 단점
#### 👎 값이 다르면 반드시 독립된 객체로 만들어야 한다.
- 값의 가짓수가 많다면 이들을 모두 만드는 데 큰 비용을 치러야 한다.

## ✨ 정리
> 클래스는 꼭 필요한 경우가 아니라면 불변이어야 한다.

불변 클래스는 장점이 많으며, 단점이라곤 특정 상황에서의 잠재적 성능 저하뿐이다.

<br>

하지만, 모든 클래스를 불변으로 만들 수 없다.

⇒ 불변으로 만들 수 없는 클래스라도 변경할 수 있는 부분을 최소한으로 줄이자.

- 그 객체를 예측하기 쉬워지고 오류가 생길 가능성이 줄어든다.
- 다른 합당한 이유가 없다면 모든 필드는 `private final`이어야 한다.