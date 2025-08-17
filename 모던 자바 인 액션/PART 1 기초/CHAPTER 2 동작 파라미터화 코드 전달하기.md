### content
- [동작 파라미터화](#동작-파라미터화)
- [2.1 변화하는 요구사항에 대응하기](#21-변화하는-요구사항에-대응하기)
  - [2.1.1 첫 번째 시도: 녹색 사과 필터링](#211-첫-번째-시도-녹색-사과-필터링)
  - [2.1.2 두 번째 시도 : 색을 파라미터화](#212-두-번째-시도--색을-파라미터화)
  - [2.1.3 세 번째 시도: 가능한 모든 속성으로 필터링](#213-세-번째-시도-가능한-모든-속성으로-필터링)
- [2.2 동작 파라미터화](#22-동작-파라미터화)
  - [2.2.1 네 번째 시도: 추상적 조건으로 필터링](#221-네-번째-시도-추상적-조건으로-필터링)
- [2.3 복잡한 과정 간소화](#23-복잡한-과정-간소화)
  - [2.3.1 익명 클래스](#231-익명-클래스)
  - [2.3.2 다섯 번째 시도 : 익명 클래스 사용](#232-다섯-번째-시도--익명-클래스-사용)
  - [2.3.3 여섯 번째 시도 : 람다 표현식 사용](#233-여섯-번째-시도--람다-표현식-사용)
  - [2.3.4 일곱 번째 시도 : 리스트 형식으로 추상화](#234-일곱-번째-시도--리스트-형식으로-추상화)

---

## 동작 파라미터화

아직 어떻게 실행할 것인지 결정하지 않은 코드 블럭으로, 이 코드 블럭은 나중에 프로그램에서 호출한다.
즉, 코드 블록의 실행은 나중으로 미뤄진다.

나중에 실행될 메서드의 인수로 코드 블록을 전달할 수 있다. 결과적으로 코드 블록에 따라 메서드의 동작이 파라미터화 된다.

ex) 컬렉션을 처리할 때 다음과 같은 메서드를 구현한다고 가정하자.

- 리스트의 모든 요소에 대해서 ‘어떤 동작’을 수행할 수 있음.
- 리스트 관련 작업을 끝낸 다음에 ‘어떤 다른 동작’을 수행할 수 있음.
- 에러가 발생하면 ‘정해진 어떤 다른 동작’을 수행할 수 있음.

이러한 동작 파라미터화를 이용하면 자주 바뀌는 요구사항에 효과적으로 대응할 수 있다.

## 2.1 변화하는 요구사항에 대응하기

ex) 기존의 농장 제고목록 애플리케이션 리스트에서 녹색 사과만 필터링하는 기능을 추가한다고 가정하자.

### 2.1.1 첫 번째 시도: 녹색 사과 필터링

사과 색을 정의하는 `Color num`이 존재한다고 가정하자.

```java
enum Color { RED, GREEN }
```

```java
public static List<Apple> filterGreenApples(List<Apple> inventory) {
	List<Apple> result = new ArrayList<>();
	
	for (Apple apple : inventory) {
		if (**GREEN.equals(apple.getColor()**) {    // 녹색 사과만 선택
			resuslt.add(apple);
		}
	}
	return result;
}
```

위 코드는 녹색 사과만 필터링한다. 만약 농부가 갑자기 변심하여 녹색 사과 말고 빨간 사과도 필터링하게 해달라 요구했다면 어떻게 해야할까?

위 메서드를 복사해서 `filterRedApples` 라는 새로운 메서드를 만드는 방법이 있지만, 이 방법은 추후 좀 더 다양한 색으로 필터링하는 등의 변화에는 적절하게 대응할 수 없다.

이런 경우 다음과 같은 규칙을 적용할 수 있다.

> 거의 비슷한 코드가 반복 존재한다면 그 코드를 추상화한다.
> 

### 2.1.2 두 번째 시도 : 색을 파라미터화

색을 파라미터화할 수 있도록 메서드에 파라미터를 추가하면 변화하는 요구사항에 좀 더 유연하게 대응하는 코드를 만들 수 있다.

```java
public static List<Apple> filterApplesByColor(List<Apple> inventory, Color color) {
	List<Apple> result = new ArrayList<>();
	
	for (Apple apple: inventory) {
		if (apple.getColor().equals(color)) {
			result.add(apple);
		}
	}
	return apple;
}
```

다음처럼 구현한 메서드를 호출할 수 있다.

```java
List<Apple> greenApples = filterApplesByColor(inventory, GREEN);
List<Apple> redApples = filterApplesByColor(inventory, RED);
```

위와 같이 구현하면 농부의 요구사항을 모두 만족할 수 있다.

만약 농부가 색 이외에도 무게에 따라 사과를 분류해달라 요구했다면 어떨까?

사과의 색을 필터링할 때와 마찬가지로 무게를 필터링하는 메서드를 다음과 같이 만들면 된다.

```java
public static List<Apple> filterApplesByWeight(List<Apple> inventory, int weight) {
	List<Apple> result = new ArrayList<>();
	
	for (Apple apple : inventory) {
		if (apple.getWeight() > weight) {
			result.add(apple);
		}
	}
	return result;
}
```

하지만 구현 코드를 보면 목록을 검색하고, 각 사과에 필터링 조건을 적용하는 부분의 코드가 색 필터링 코드와 대부분 중복된다.

⇒ 이는 **소프트웨어 공학의 DRY(같은 것을 반복하지 말 것) 원칙을 어기는 것**이다.

### 2.1.3 세 번째 시도: 가능한 모든 속성으로 필터링

다음 방법은 엔지니어링적으로 비싼 대가를 치러야 하는 방법이다. 

색과 무게를 `filter`라는 메서드로 합치는 방법이다. 이때 색이나 무게 중, 어떤 것을 기준으로 필터링할지 가리키는 플래그를 추가할 수 있다.

```java
public static List<Apple> filterApples(List<Apple> inventory, Color color, int weight, boolean flag) {
	List<Apple> result = new ArrayList<>();
	
	for (Apple apple : inventory) {
		if ((flag && apple.getColor().equals(color)) || (!flag && apple.getWeight() > weight)) {
			result.add(apple);
		}
	}
	return result;
}
```

```java
List<Apple> greenApples = filterApples(inventory, GREEN, 0, true);
List<Apple> heavyApples = filterApples(inventory, null, 150, false);
```

위처럼 코드를 작성하면 메서드를 사용할 때, 각 파라미터들이 어떤 역할을 하는지 알 수 없다.

또한 앞으로 요구사항이 변경되었을 때 유연하게 대응할 수 없다.

ex) 사과의 크기, 모양, 출하지 등으로 사과를 필터링하고 싶다면, 결국 여러 중복 필터 메서드를 만들거나 모든 것을 처리하는 거대한 하나의 필터 메서드를 구현해야 한다.

## 2.2 동작 파라미터화

앞에서 파라미터를 추가하는 방법이 아닌 변화하는 요구사항에 좀 더 유연하게 대응할 수 있는 방법이 필요하다는 것을 확인했다.

프레디케이트 함수를 통해 사과의 어떤 속성에 기초해 불리언 값을 반환할 수 있다.

```java
public interface ApplePredicate {
	boolean test(Apple apple);
}
```

프레디케이트 함수를 통해 다양한 선택 조건을 대표하는 여러 버전의 `ApplePredicate`를 정의할 수 있다.

```java
public class AppleHeavyWeightPredicate implements ApplePredicate {
	public boolean test(Apple apple) {    // 무거운 사과만 선택
		return apple.getWeight() > 150;
	}
}
```

```java
public class AppleGreenColorPredicate implements ApplePredicate {
	public boolean test(Apple apple) {     // 녹색 사과만 선택
		return GREEN.equals(apple.getColor());
	}
}
```

이처럼 `AppleHeavyWeightPredicate`와 `AppleGreenColorPredicate`에서 조건에 따라 filter 메서드가 다르게 동작할 것이라고 예측할 수 있다.

이를 **전략 디자인 패턴**이라고 한다.

> 각 알고리즘(전략이라 불리는)을 캡슐화하여 알고리즘 패밀리는 정의해둔 다음에 런타임에 알고리즘을 선택하는 기법
> 
- `ApplePredicate`: 알고리즘 패밀리
- `AppleHeavyWeightPredicate`, `AppleGreenColorPredicate`: 전략

📌 **ApplePredicate가 다양한 동작을 수행할 수 있었던 이유**

`filterApples`에서 `ApplePredicate` 객체를 받아 애플의 조건을 검사하도록 메서드를 고쳐야 한다.

메서드가 다양한 동적(또는 전략)을 받아 내부적으로 다양한 동작을 수행하도록 **동작 파라미터화** 할 수 있

다.

### 2.2.1 네 번째 시도: 추상적 조건으로 필터링

다음은 `ApplePredicate`를 이용한 필터 메서드다.

```java
public static List<Apple> filterApples(List<Apple> inventory, ApplePredicate p) {
	List<Apple> result = new ArrayList<>();
	
	for (Apple apple : inventory) {
		if (p.test(apple)) {
			result.add(apple);
		}
	}
	return result;
}
```

📌 **코드/동작 전달하기**

```java
public class AppleRedAndHeavyPredicate implements ApplePredict {
	public boolean test(Apple apple) {
		return RED.equals(apple.getColor()) && apple.getWeight() > 150;
	}
}
```

```java
List<Apple> redAndHeavyApples = filterApples(inventory, new AppleRedAndHeavyPredicate());
```

`ApplePredicate` 객체에 의해 `filterApples` 메서드의 동작이 결정됐다.

✨ **동작 파라미터의 강점**

- 컬렉션 탐색 로직과 각 항목에 적용할 동작을 분리할 수 있다.
    
    때문에 한 메서드가 다른 동작을 수행하도록 재활용할 수 있다.
    

## 2.3 복잡한 과정 간소화

위처럼 여러 클래스를 구현해서 인스턴스화하는 과정이 조금 거추장스럽게 느껴질 수 있다.

```java
public class AppleHeavyWeightPredicate implements ApplePredicate {
	public boolean test(Apple apple) {
		return apple.getWeight() > 150;
	}
}

public class AppleGreenColorPredicate implements ApplePredicate {
	public boolean test(Apple apple) {
		return GREEN.equlas(apple.getColor());
	}
}

public calls FilteringApples {
	public static void main(String...args) {
		List<Apple> inventory = Arrays.asList(new Apple(80, GREEN),
																					new Apple(155, GREEN),
																					new Apple(120, RED));
																
		List<Apple> heavyApples = filterApples(inventory, new AppleHeavyWeightPredicate());
		List<Apple> greenApples = filterApples(inventory, new AppleGreenPredicate());
	}
	
	public static List<Apple> filterApples(List<Apple> inventory, ApplePredicate p) {
		List<Apple> result = new ArrayList<>();
		
		for (Apple apple : inventory) {
			if (p.test(apple)) {
				result.add(apple);
			}
		}
		return result;
	}
}
```

로직과 관련 없는 코드가 많이 추가되었다.

이를 개선하기 위해서는 클래스의 선언과 인스턴스화를 동시에 수행할 수 있는 익명 클래스를 사용하면 된다.

### 2.3.1 익명 클래스

> 이름이 없는 클래스
> 

클래스 선언과 인스턴스화를 동시에 할 수 있다. 즉, 즉석에서 필요한 구현을 만들어 사용할 수 있다.

### 2.3.2 다섯 번째 시도 : 익명 클래스 사용

다음은 익명 클래스를 사용해서 `ApplePredicate`를 구현하는 객체를 만드는 방법으로 필터링 예제를 구현한 코드다.

```java
List<Apple> redApples = filterApples(inventory, new ApplePredicate() {
	public boolean test(Apple apple) {
		return RED.equals(apple.getColor());
	}
});
```

📌 **익명 클래스를 사용할 때의 단점**

1. 너무 많은 공간을 차지한다.
2. 많은 프로그래머가 익명 클래스의 사용에 익숙하지 않다.

때문에 익명 클래스를 사용하는 것보다 동작 파라미터를 사용하는 것이 요구사항 변화에 더 유연하게 대응할 수 있다.

### 2.3.3 여섯 번째 시도 : 람다 표현식 사용

```java
List<Apple> result = filterApples(inventory, (Apple apple) -> RED.equals(apple.getColor()));
```

위와 같이 람다 표현식을 사용하면 이전 코드보다 더욱 간결하면서 문제를 더 잘 설명할 수 있게 되었다.

### 2.3.4 일곱 번째 시도 : 리스트 형식으로 추상화

```java
public interface Predicate<T> {
	boolean test(T t);
}
```

```java
public static <T> List<T> filter(List<T> list, Predicate<T> p) {
	List<T> result = new ArrayList<>();
	
	for (T e : list) {
		if (p.test(e)) {
			result.add(e);
		}
	}
	return result;
}
```

```java
List<Apple> redApples = filter(inventory, (Apple apple) -> RED.equals(apple.getColor()));

List<Integer> evenNumbers = filter(numbers, (Integer i) -> i % 2 == 0);
```