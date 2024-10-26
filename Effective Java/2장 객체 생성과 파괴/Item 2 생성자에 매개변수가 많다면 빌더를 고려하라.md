# Item 2 생성자에 매개변수가 많다면 빌더를 고려하라.
### content
- [❓점층적 생성자 패턴](#점층적-생성자-패턴)
    - [👎점증적 생성자 패턴의 단점](#점증적-생성자-패턴의-단점)
    - [설정하기 원치 않는 매개변수에도 값을 지정해줘야 한다.](#설정하기-원치-않는-매개변수에도-값을-지정해줘야-한다)
    - [매개변수 개수가 많아지면 코드를 작성하거나 읽기 어렵다.](#매개변수-개수가-많아지면-코드를-작성하거나-읽기-어렵다)
    - [타입이 같은 매개변수가 연달아 늘어서 있으면 찾기 어려운 버그로 이어질 수 있다.](#타입이-같은-매개변수가-연달아-늘어서-있으면-찾기-어려운-버그로-이어질-수-있다)
- [❓자바빈즈 패턴](#자바빈즈-패턴)
    - [👎자바빈즈 패턴의 단점](#자바빈즈-패턴의-단점)
    - [객체 하나를 만드려면 여러 메서드를 호출해야 한다.](#객체-하나를-만드려면-여러-메서드를-호출해야-한다)
    - [객체가 완전히 생성되기 전에는 일관성이 무너진 상태에 놓이게 된다.](#객체가-완전히-생성되기-전에는-일관성이-무너진-상태에-놓이게-된다)
    - [클래스를 불변으로 만들 수 없고, 스레드 안전성을 얻으려면 추가 작업이 필요하다.](#클래스를-불변으로-만들-수-없고-스레드-안전성을-얻으려면-추가-작업이-필요하다)
- [❓빌더 패턴](#빌더-패턴)
    - [불변식 검사](#불변식-검사)
    - [계층적 설계에서의 빌더 패턴의 사용](#계층적-설계에서의-빌더-패턴의-사용)
    - [뉴욕 피자](#뉴욕-피자)
    - [칼초네 피자](#칼초네-피자)
    - [👎빌더 패턴의 단점](#빌더-패턴의-단점)
    - [객체를 만드려면 빌더부터 만들어야 한다.](#객체를-만드려면-빌더부터-만들어야-한다)
- [✨정리](#정리)

---
## ❓점층적 생성자 패턴
점층적 생성자 패턴(telescoping constructor pattern)은 필수 매개변수만 받는 생성자에 선택 매개변수를 갖는 생성자를 필요한 만큼 두는 패턴이다.

예를 들어, 식품 포장의 영양정보를 표현한다면 다음과 같은 필드를 가져야 한다.
- servingSize: 필수
- servings: 필수
- calories: 선택
- fat: 선택
- sodium: 선택
- carbohydrate: 선택

이런 필드를 가진 식품 포장의 영양정보를 점층적 생성자 패턴으로 나타내면 다음과 같다.

```java
public class NutritionFacts {
    private final int servingSize;  // (ml, 1회 제공량), 필수
    private final int servings;     // (회, 총 n회 제공량), 필수
    private final int calories;     // (1회 제공량당), 선택
    private final int fat;          // (g/1회 제공량), 선택
    private final int sodium;       // (mg/1회 제공량), 선택
    private final int carbohydrate; // (g/1회 제공량), 선택

    public NutritionFacts(int servingSize, int servings) {
        this(servingSize, serving, 0);
    }

    public NutritionFacts(int servingSize, int servings, int calories) {
        this(servingSize, servings, calories, 0);
    }

    public NutritionFacts(int servingSize, int servings, int calories, int fat) {
        this(servingSize, servings, calories, fat, 0);
    }

    public NutritionFacts(int servingSize, int servings, int calories, int fat, int sodium) {
        this(servingSize, servings, calories, fat, sodium, 0);
    }

    public NutritionFacts(int servingSize, int servings, int calories, int fat, int sodium, int carbohydrate) {
        this.servingSize = servingSize;
        this.servings = servings;
        this.calories = calories;
        this.fat = fat;
        this.sodium = sodium;
        this.carbohydrate = carbohydrate;
    }
}
```
필요한 매개변수를 모두 포함한 생성자 중 가장 짧은 것을 골라 사용하면 된다.
```java
NutritionFacts cocaCola = new NutritionFacts(240, 8, 100, 0, 35, 27);
```

### 👎점증적 생성자 패턴의 단점
#### 설정하기 원치 않는 매개변수에도 값을 지정해줘야 한다.
점층적 생성자 패턴을 사용할 때, 필요한 매개변수를 모두 포함한 생성자 중 가장 짧은 생성자를 골라 사용해야 한다.

만약 매개변수로 `servingSize`, `servings`, `calories`, `sodium`, `carbohydrate`가 필요하다면, 아래 생성자가 해당 매개변수를 모두 포함하고 있는 가장 짧은 생성자이기 때문에 아래 생성자를 사용할 것이다.
```java
public NutritionFacts(int servingSize, int servings, int calories, int fat, int sodium, int carbohydrate) {
        this.servingSize = servingSize;
        this.servings = servings;
        this.calories = calories;
        this.fat = fat;
        this.sodium = sodium;
        this.carbohydrate = carbohydrate;
    }
```
그렇다면 필요하지 않은 `fat`은 어떻게 해야 할까?
```java
NutritionFacts cocaCola = new NutritionFacts(240, 8, 100, 0, 35, 27);
```
설정하고 싶지 않은 매개변수인 `fat`은 `0`으로 설정해야 한다.

#### 매개변수 개수가 많아지면 코드를 작성하거나 읽기 어렵다.
아래 생성자 호출 예시는 매개변수가 6개만 존재하기 때문에 크게 나쁜 코드 같지 않다.
```java
NutritionFacts cocaCola = new NutritionFacts(240, 8, 100, 0, 35, 27);
```
하지만 만약 매개변수가 이보다 더 늘어나면 어떤 매개변수가 무슨 뜻인지, 매개변수가 총 몇 개인지 매우 헷갈릴 것이다.

#### 타입이 같은 매개변수가 연달아 늘어서 있으면 찾기 어려운 버그로 이어질 수 있다.
아래 생성자 호출 예시는 `servings`와 `calories`가 바뀌었다.
```java
NutritionFacts cocaCola = new NutritionFacts(240, 100, 8, 0, 35, 27);
```
하지만무엇이 잘못되었는지 한눈에 파악하기 어렵고, 컴파일 오류가 발생하지도 않는다.

## ❓자바빈즈 패턴
자바빈즈 패턴(JavaBeans pattern)은 매개변수가 없는 생성자로 객체를 만든 후, 세터(setter) 메서드를 호출해 원하는 매개변수의 값을 설정하는 방식이다.


자바빈즈 패턴을 사용하면 점층적 생성자 패턴의 단점들을 해결할 수 있다.
```java
public class NutritionsFacts {
    // 매개변수들은 (기본값이 있다면) 기본값으로 초기화한다.
    private int servingSize = -1;   // 필수; 기본값 없음.
    private int servings = -1;      // 필수; 기본값 없음.
    private int calories = 0;
    private int fat = 0;
    private int sodium = 0;
    private int carbohydrate = 0;

    public NutritionsFacts () {
        // 세터 메서드들
        public void setServingSize(int val) {servingSize = val;}
        public void setServings(int val) {servings = val;}
        public void setCalories(int val) {calories = val;}
        public void setFat(int val) {fat = val;}
        public void setSodium(int val) {sodium = val;}
        public void setCarbohydrate(int val) {carbohydrate = val;}
    }
}
```
세터 메서드들 때문에 코드가 길어졌지만, 인스턴스를 만들기 쉬워졌다.

### 👎자바빈즈 패턴의 단점
#### 객체 하나를 만드려면 여러 메서드를 호출해야 한다.
`cola` 객체를 만들려면 많은 세터 메서드를 호출해야만 한다.
```java
NutritionFacts cola = new NutritionFacts();
cola.setServingSize(240);
cola.setServings(8);
cola.setCalories(100);
cola.setFat(0);
cola.setSodium(35);
cola.setCarbohydrate(27);
```

#### 객체가 완전히 생성되기 전에는 일관성이 무너진 상태에 놓이게 된다.
만약 객체 `cola`를 만들기 위해 무수히 많은 세터 메서드를 통해 매개변수의 값을 설정하는 과정에서 이런 세터 메서드들이 여러 곳에 흩어져있다면 어떨까?

```java
NutritionFacts cola = new NutritionFacts();
cola.setServingSize(240);

// ...

cola.setCalories(100);
cola.setFat(0);

// ...

cola.setSodium(35);
cola.setCarbohydrate(27);
```
잘 살펴보면 `Servings`를 설정하는 세터 메서드가 빠진 것을 알 수 있다.

이렇게 세터 메서드가 하나 빠지게 되어도 발견하기 어렵다.

#### 클래스를 불변으로 만들 수 없고, 스레드 안전성을 얻으려면 추가 작업이 필요하다.
클래스를 불변으로 만들 수 없기 때문에, 스레드 안전성을 위해 `freeze` 메서드를 통해 추가 작업을 하여 이러한 단점을 완화한다.

`freeze` 메서드는 이러한 객체를 얼리기 전에는 사용할 수 없게 한다.

하지만, 이러한 방법은 다루기 어렵기 때문에 실전에서는 거의 사용하지 않는다.

또한, 프로그래머가 `freeze` 메서드를 확실히 호출했는지 컴파일러가 보증하지 못하기 때문에 런타임 오류에도 취약하다.

> 📌 [불변 값 클래스 - Item 17 변경 가능성을 최소화하라]()

## ❓빌더 패턴
빌더 패턴(builder pattern)은 위에서 소개한 점층적 생성자 패턴의 안전성과 자바빈즈 패턴의 가독성을 모두 갖춘 패턴이다.

이러한 빌더 패턴은 필수 매개변수만으로 생성자를 호출해 빌더 객체를 얻고, 빌더 객체의 세터 메서드들로 선택 매개변수를 설정하고, build 메서드를 호출해 객체를 얻는 방법이다.

```java
public class NutritionFacts {
    private final int servingSize;
    private final int servings;
    private final int calories;
    private final int fat;
    private final int sodium;
    private final int carbohydrate;

    public static class Builder {
        // 필수 매개변수
        private final int servingSize;
        private final int servings;

        // 선택 매개변수 - 기본값으로 초기화한다.
        private int calories = 0;
        private int fat = 0;
        private int sodium = 0;
        private int carbohydrate = 0;

        public Builder(int servingSize, int servings) {
            this.servingSize = servingSize;
            this.servings = servings;
        }

        public Builder calories(int val) {
            calories = val;
            return this;
        }
        public Builder fat(int val) {
            fat = val;
            return this;
        }
        public Builder sodium(int val) {
            sodium = val;
            return this;
        }
        public Builder carbohydrate(int val) {
            carbohydrate = val;
            return this;
        }

        public NutritionFacts build() {
            return new NutritionFacts(this);
        }
    }

    private NutritionFacts(Builder builder) {
        servingSize = builder.servingSize;
        servings = builder.servings;
        calories = builder.calories;
        fat = builder.fat;
        sodium = builder.sodium;
        carbohydrate = builder.carbohydrate;
    }
}
```

이러한 빌더 패턴은 불변이며, 모든 매개변수의 기본 값을 한 곳에 모아뒀다.

빌더의 세터 메서드들은 빌더 자신을 반환하기 때문에 연쇄적으로 호출할 수 있고, 이러한 방식을 플루언트 API(fluent API) 혹은 메서드 연쇄(method chaining)이라고 한다.

```java
NutritionFacts cola = new NutritionFacts.Builder(240, 8)
            .calories(100).sodium(35).carbohydrate(27).build();
```

위 클라이언트 코드는 쓰고 읽기 쉽다.

### 불변식 검사
잘못된 매개변수를 최대한 일찍 발견하기 위해 빌더의 생성자와 메서드에서 입력 매개변수를 검사하고, build 메서드가 호출하는 생성자에서 여러 매개변수에 걸친 불변식을 검사해야 한다.

또한, 공격에 대비해 이런 불변식을 보장하려면 빌더로부터 매개변수를 복사한 후 해당 객체 필드들도 검사해야 한다.

만약 검사 후 잘못된 점이 발견되면 예외 메시지에 해당 내용을 담아 `IllegalArgumentException`을 던지면 된다.

> 📌 [불변식 검사 - Item 50 적시에 방어적 복사본을 만들라]()

> 📌 [예외 메시지 - Item 75 예외의 상세 메시지에 실패 관련 정보를 담으라]()

### 계층적 설계에서의 빌더 패턴의 사용
이러한 빌더 패턴은 계층적으로 설계된 클래스와 함께 쓰기 좋다.

```java
public abstract class Pizza {
    public enum Topping { HAM, MUSHROOM, ONION, PEPPER, SAUSAGE };
    final Set<Topping> toppings;

    abstract static class Builder<T extends Builder<T>> {
        EnumSet<Topping> toppings = EnumSet.noneOf(Topping.class);
        public T addTopping(Topping topping) {
            toppings.add(Objects.requireNonNull(topping));
            return self();
        }

        abstract Pizza build();

        // 하위 클래스는 이 메서드를 재정의(overriding)하여
        // "this"를 반환하도록 해야 한다.
        protected abstract T self();
    }

    Pizza(Builder<?> builder) {
        toppings = builder.toppings.clone();    // Item 50
    }
}
```

`Pizza.Builder` 클래스는 `재귀적 타입 한정`을 이용하는 제네릭 타입이다.

여기에 추상 메서드 `self`를 더해 하위 클래스에서 형번환 없이 메서드 연쇄를 지원할 수 있다.

> 📌 [불변식 검사 - Item 50 적시에 방어적 복사본을 만들라]()

> 📌 [재귀적 타입 한정 - Item 30 이왕이면 제네릭 메서드로 만들라]()

아래 두 예시는 `Pizza`의 하위 클래스인 `일반적 뉴욕 피자`와 `칼초네 피자`이다.

`일반적 뉴욕 피자`는 크기 매개변수를 받고, `칼초네 피자`는 소스 여부를 매개변수로 받는다.

#### 뉴욕 피자

```java
public class NyPizza extends Pizza {
    public enum Size { SMALL, MEDIUM, LARGE };
    private final Size size;

    public static class Builder extends Pizza.Builder<Builder> {
        private final Size size;

        public Builder(Size size) {
            this.size = Objects.requireNonNull(size);
        }

        @Override
        public NyPizza build() {
            return new NyPizza(this);
        }

        @Override
        protected Builder self() {
            return this;
        }
    }

    private NyPizza(Builder builder) {
        super(builder);
        size = builder.size;
    }
}
```

#### 칼초네 피자
```java
public class Calzone extends Pizza {
    private final boolean sauceInside;

    public static class Builder extends Pizza.Builder<Builder> {
        private boolean sauceInside = false;

        public Builder sauceInside() {
            sauceInside = true;
            return this;
        }

        @Override
        public Calzone build() {
            return new Calzone(this);
        }

        @Override
        protected Builder self() {
            return this;
        }
    }

    public Calzone(Builder builder) {
        super(builder);
        this.sauceInside = builder.sauceInside;
    }
}
```

각 하위 클래스의 빌더가 정의한 build 메서드는 해당하는 구체 하위 클래스를 반환하도록 선언한다.

즉, `NyPizza.Builder`는 NyPizza를 반환하고 `Calzone.Builder`는 Calzone을 반환한다.

이렇게 하위 클래스의 메서드가 상위 클래스의 메서드가 정의한 반환 타입이 아닌, 그 하위 타입을 반환하는 기능을 `공변 반환 타이핑(covariant return typing)`이라 한다.

공변 반환 타이핑을 통해 클라이언트가 형 변환에 신경쓰지 않고 빌더를 사용할 수 있다.

```java
NyPizza pizza = new NyPizza.Builder(SMALL)
            .addTopping(SAUSAGE).addTopping(ONION).build();
Calzone calzone = new Calzone.Builder()
            .addTopping(HAM).sauceInside().build();
```

### 👎빌더 패턴의 단점
#### 객체를 만드려면 빌더부터 만들어야 한다.
빌더 생성 비용이 크지는 않지만, 성능에 민감한 상황에서는 문제가 될 수 있다.

## ✨정리
> 생성자나 정적 팩터리가 처리해야 할 매개변수가 많다면 빌더 패턴을 선택하는 게 더 낫다.

빌더는 점층적 생성자보다 클라이언트 코드를 읽고 쓰기가 훨씬 간결하고, 자바빈즈보다 훨씬 안전하다.