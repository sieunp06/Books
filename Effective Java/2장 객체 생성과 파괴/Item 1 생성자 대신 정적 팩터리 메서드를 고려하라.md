# Item 1. 생성자 대신 정적 팩터리 메서드를 고려하라.

### Content
- [정적 팩터리 메서드(static factory method)](#정적-팩터리-메서드static-factory-method)
  - [👍 장점 1 - 이름을 가질 수 있다.](#-장점-1---이름을-가질-수-있다)
  - [👍 장점 2 - 호출될 때마다 인스턴스를 새로 생성하지는 않아도 된다.](#-장점-2---호출될-때마다-인스턴스를-새로-생성하지는-않아도-된다)
  - [👍 장점 3 - 반환 타입의 하위 타입 객체를 반환할 수 있는 능력이 있다.](#-장점-3---반환-타입의-하위-타입-객체를-반환할-수-있는-능력이-있다)
  - [👍 장점 4 - 입력 매개변수에 따라 매번 다른 클래스의 객체를 반환할 수 없다.](#-장점-4---입력-매개변수에-따라-매번-다른-클래스의-객체를-반환할-수-없다)
  - [👎 단점 1 - 정적 팩토리 메서드만 제공하면 하위 클래스를 만들 수 없다.](#-단점-1---정적-팩토리-메서드만-제공하면-하위-클래스를-만들-수-없다)
  - [👎 단점 2 - 정적 팩토리 메서드는 프로그래머가 찾기 어렵다.](#-단점-2---정적-팩토리-메서드는-프로그래머가-찾기-어렵다)

---
## 정적 팩터리 메서드(static factory method)
| 클래스의 인스턴스를 반환하는 단순한 정적 메서드.

<br>

다음은 boolean의 기본 타입인 박싱 클래스(boxed class)인 Boolean에서 발췌한 간단한 예시이다.
```java
public static Boolean valueOf(boolean b) {
	return b ? Boolean.TRUE : Boolean.FALSE;
}
```

위 메서드 기본 타입인 boolean의 값을 받아 Boolean 객체 참조로 변환한다.

<br>

정적 팩터리 메서드를 사용할 때, 다음과 같은 장•단점이 존재한다.

### 👍 장점 1 - 이름을 가질 수 있다.
생성자에 넘기는 매개변수와 생성자 자체만으로는 반환될 객체의 특성을 제대로 설명하지 못한다.

반면, 정적 팩터리 메서드는 반환될 객체의 특성을 쉽게 묘사할 수 있다.

#### ❌ 생성자
```java
public class Fruit {
	int price;
	
	public Fruit(int price) {
		this.price = price;
	}
	
	public static void main(String[] args) {
		Fruit apple = new Fruit(500);     // 용도를 파악하기 힘듦.
	}
}
```

#### ⭕ 정적 팩터리 메서드
```java
public class Fruit {
	int price;
	
	public Fruit(int price) {
		this.price = price;
	}
	
	// 정적 팩터리 메서드
	public static Fruit withPrice(int price) {
		return new Fruit(price);
	}
	
	public static void main(String[] args) {
		Fruit apple = Fruit.withPrice(500);
	}
}
```
위처럼 정적 팩터리 메서드를 사용하면 반환될 객체의 특성을 더욱 쉽게 이해할 수 있다.

<br>

하나의 시그니처로는 생성자를 하나만 만들 수 있다. 하지만 정적 팩터리 메서드는 그러한 제약이 없다.

```java
public class Fruit {
	int price;
	int from;
	
	public Fruit() {}
	
	public static Fruit withName(String name) {
		Fruit fruit = new Fruit();
		fruit.name = name;
		return fruit;
	}
	
	public static Fruit withFrom(String from) {
		Fruit fruit = new Fruit();
		fruit.from = from;
		return fruit;
	}
}
```
위 예제에서 `Fruit.withName`과 `Fruit.withFrom`의 시그니처가 같지만, 정적 팩터리 메서드는 이러한 시그니처 제한으로부터 자유롭다.


### 👍 장점 2 - 호출될 때마다 인스턴스를 새로 생성하지는 않아도 된다.
정적 팩터리 메서드는 호출될 때마다 인스턴스를 새로 생성하지 않아도 된다.

때문에 불변 클래스(immutable class)는 인스턴스를 미리 만들어 놓거나 새로 생성한 인스턴스를 캐싱하여 재활용하는 식으로 불필요한 객체 생성을 피할 수 있다.

<br>

`Boolean.valueOf(boolean)` 메서드는 자주 사용하면서 변하지 않을 객체를 미리 생성해두고 반환하기 때문에 객체를 생성하지 않는다.

```java
public final class Boolean implements java.io.Serializable, Comparable<Boolean> {
	public static final Boolean TRUE = new Boolean(true);
	public static final Boolean FALSE = new Boolean(false);

	public static Boolean valueOf(boolean b) {
		return b ? Boolean.TRUE : Boolean.FALSE;
	}
}
```
따라서 같은 객체가 자주 요청되는 상황이라면 성능을 상당히 끌어올려 준다.

<br>

위와 같이 반복되는 요청에 같은 객체를 반환하는 식으로 정적 팩터리 방식의 클래스는 언제 어느 인스턴스를 살아 있게 할지를 철저히 통제할 수 있다.

이를 인스턴스 통제(instance-controlled)라고 한다.

### 👍 장점 3 - 반환 타입의 하위 타입 객체를 반환할 수 있는 능력이 있다.
인터페이스의 구현 클래스를 공개하지 않고도 그 객체를 반환할 수 있어 API를 작게 유지할 수 있다.

```java
public interface Fruit {
	static Apple getApple() {
		return new Apple();
	}
	static Banana getBanana() {
		return new Banana();
	}
}

class Apple implements Fruit {}
class Banana implements Fruit {}

public class Main {
	public static void main(String[] args) {
		Fruit fruit = Fruit.getApple();
		Fruit fruit = Fruit.getBanana();
	}
}
```

### 👍 장점 4 - 입력 매개변수에 따라 매번 다른 클래스의 객체를 반환할 수 없다.
클라이언트는 정적 팩토리 메서드의 이름만 알 뿐, 어떤 객체가 반환되는지 알 수 없으며 알 필요도 없다.

EnumSet 클래스는 public 생성자 없이 오직 정적 팩터리만을 제공하는데, OpenJDK에서는 원소의 수에 따라 두 가지 하위 클래스 중 하나의 인스턴스를 반환한다.

```java
public static <E extends Enum<E>> EnumSet<E> noneOf(Class<E> elementType) {
	Enum<?>[] universe = getUniverse(elementType);
	if (universe == null) 
		throw new ClassCastException(elementType + " not an enum");
		
	if (universe.length <= 64) 
		return new RegularEnumSet<>(elementType, universe);
	else 
		return new JumboEnumSet<>(elementType, universe);
}
```
위  `EnumSet.noneOf` 메서드는 매개변수인 elementType의 길이에 따라 RegularEnumSet과 JumboEnumSet을 반환한다. 때문에 클라이언트는 `EnumSet.noneOf` 메서드의 반환 결과를 알 수 없게 된다. 

### 👎 단점 1 - 정적 팩토리 메서드만 제공하면 하위 클래스를 만들 수 없다.
상속을 하려면 `public`이나 `protected` 생성자가 필요하다.

하지만 정적 팩토리 메서드를 사용할 때, 클라이언트가 코드 작성자의 의도대로 객체를 생성하도록 하기 위해 생성자를 `private`으로 막아두는 것이 보통이다.

때문에 정적 팩토리 메서드만 제공하면 상속을 통해 하위 클래스를 만들 수 없다.

하지만, 상속보다 컴포지션을 사용하도록 유도하고 불변 타입으로 만들려면 이러한 제약을 지켜야 한다.

때문에 막연히 단점만은 아니다.

### 👎 단점 2 - 정적 팩토리 메서드는 프로그래머가 찾기 어렵다.

정적 팩토리 메서드를 사용하면 생성자와 달리 API 설명이 명확히 드러나지 않는다. 

때문에 사용자가 인스턴스를 생성해주는 메서드를 직접 찾아내야 한다.

이러한 문제점을 해결하기 위해 API 문서를 잘 작성하고 메서드 이름도 널리 알려진 규약을 통해 지어야 한다.

다음은 정적 팩토리 메서드에 흔히 사용하는 명명 방식이다.

- **`from`**: 매개변수를 하나 받아서 해당 타입의 인스턴스를 반환하는 형변환 메서드
- **`of`**: 여러 매개변수를 받아 적합한 타입의 인스턴스를 반환하는 집계 메서드
- **`valueOf`**: from과 of의 더 자세한 버전
- **`instance 혹은 getInstance`**: (매개변수를 받는다면) 매개변수로 명시한 인스턴스를 반환하지만, 같은 인스턴스임을 보장하지는 않는다.
- **`create 혹은 newInstance`**: instance 혹은 getInstance와 같지만, 매번 새로운 인스턴스를 생성해 반환한다.
- **`getType`**: getInstance와 같으나, 생성할 클래스가 아닌 다른 클래스에 팩터리 메서드를 정의할 때 쓴다. “Type”은 팩터리 메서드가 반환할 객체의 타입이다.
- **`newType`**: newInstance와 같으나, 생성할 클래스가 아닌 다른 클래스에 팩터리 메서드를 정의할 때 쓴다. “Type”은 팩터리 메서드가 반환할 객체의 타입이다.
- **`type`**: getType과 newType의 간결한 버전