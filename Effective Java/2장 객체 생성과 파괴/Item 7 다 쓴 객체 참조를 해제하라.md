# Item 7. 다 쓴 객체 참조를 해제하라.
## 메모리 누수를 일으키는 원인 1 - 객체 참조

자바에는 가비지 컬렉터가 존재하며, 다 쓴 객체를 알아서 회수하는 역할을 한다.

때문에 메모리 관리에 더 이상 신경 쓰지 않아도 된다고 오해할 수 있는데, 이는 사실이 아니다.

<br>

다음은 스택을 간단히 구현한 코드이다.

```java
public class Stack {
	private Object[] elements;
	private int size = 0;
	private static final int DEFAULT_INITIAL_CAPACITY = 16;
	
	public Stack() {
		elements = new Object[DEFAULT_INITIAL_CAPACITY];
	}
	
	public void push(Object e) {
		ensureCapacity();
		elements[size++] = e;
	}
	
	public Object pop() {
		if (size == 0) {
			throw new EmptyStackException();
		}
		return elements[--size];
	}
	
	/**
	 * 원소를 위한 공간을 적어도 하나 이상 확보한다.
	 * 배열 크기를 늘려야 할 때마다 대략 두 배씩 늘린다.
	 */
	 private void ensureCapacity() {
		 if (elements.length == size) {
			 elements = Arrays.copyOf(elements, 2 * size + 1);
		 }
	 }
}
```

위 코드에는 메모리 누수가 발생하게 된다. 

스택이 커졌다가 줄어들 때, 프로그램에서 그 객체를 더 이상 사용하지 않더라도 스택에서 꺼내진 객체들을 가비지 컬렉터가 회수하지 않는다.

때문에 이 스택이 그 객체들의 다 쓴 참조(obsolete reference)를 여전히 가지고 있게 된다.

<br>

위 스택을 사용하는 프로그램을 오래 실행하다 보면 점차 가비지 컬렉션 활동과 메모리 사용량이 늘어나 결국 성능이 저하될 것이다.

드물지만 심할 때는 디스크 페이징이나 `OutOfMemoryError`를 일으켜 프로그램이 예기치 않게 종료되기도 한다.

### 가비지 컬렉터 언어에서 메모리 누수를 해결하는 방법: null

이런 가비지 컬렉터 언어에서는 메모리 누수를 찾기가 아주 까다롭다.

객체 참조 하나를 살려두면 가비지 컬렉터는 그 객체뿐만 아니라 그 객체가 참조하는 모든 객체를 회수해가지 못한다.

그래서 단 몇 개의 객체가 매우 많은 객체를 회수되지 못하게 할 수도 있고 잠재적으로 성능에 악영향을 줄 수 있다.

<br>

이런 메모리 누수를 해결하기 위해서는 해당 참조를 다 썼을 때 `null` 처리(참조 해제) 하면 된다.

```java
public Object pop() {
	if (size == 0)
		throw new EmptyStackException();
	Object result = elements[--size];
	elements[size] = null;    // 다 쓴 참조 객체
	return result;
}
```

위와 같이 `null` 처리한 다 쓴 참조를 실수로 사용하면 `NullPointerException`을 던지며 종료된다.

### 다 쓴 참조를 null 처리해야 하는 경우

다 쓴 참조를 `null` 처리하는 건 프로그램을 필요 이상으로 지저분하게 만든다.

객체 참조를 null 처리하는 일은 예외적인 경우여야 한다.

때문에 객체 참조를 해제하는 가장 좋은 방법은 그 참조를 담은 변수를 유효 범위(scope) 밖으로 밀어내는 것이다.

<br>

위 Stack 클래스가 메모리 누수에 취약한 이유는 스택이 자기 메모리를 직접 관리하기 때문이다.

이 스택은 elements 배열로 저장소 풀을 만들어 원소들을 관리한다.

배열의 활성 영역에 속한 원소들이 사용되고 비활성 영역은 쓰이지 않는다. 

하지만 가비지 컬렉터는 이를 알지 못한다.

가비지 컬렉터는 비활성 영역에서 참조하는 객체도 똑같이 유효한 객체다.

때문에 비활성 영역이 되는 순간 `null` 처리해서 해당 객체를 더 이상 쓰지 않을 것임을 가비지 컬렉터에 알려야 한다.

## 메모리 누수를 일으키는 원인 2 - 캐시

객체 참조를 캐시에 넣고, 해당 객체를 다 쓰고 난 뒤에도 방치하면 메모리 누수가 발생하게 된다.

이런 문제는 다음과 같은 방법을 통해 해결할 수 있다.

1. `WeakHashMap`
    - 캐시 외부에서 키를 참조하는 동안만 엔트리가 살아 있는 캐시가 필요한 상황에 사용할 수 있다.다 쓴 엔트리는 그 즉시 자동으로 제거될 것이다.
    - 단, WeakHashMap은 이러한 상황에서만 유용하다.
2. `LinkedHashMap`
    - 캐시를 만들 때 보통은 캐시 엔트리의 유효 기간을 정확히 정의하기 어렵기 때문에 시간이 지날수록 엔트리의 가치를 떨어뜨리는 방식을 흔히 사용한다.
    - `ScheduledThreadPoolExecutor` 같은 백그라운드 스레드를 활용하거나 캐시에 새 엔트리를 추가할 때 부수 작업으로 수행하는 방법이 있다.
    - LinkedHashMap은 `removeEldestEntry` 메서드를 써서 이를 처리한다.

## 메모리 누수를 일으키는 원인 3 - 리스너 or 콜백

클라이언트가 콜백을 등록만 하고 명확히 해지하지 않는다면, 콜백은 계속 쌓이게 된다.

이럴 경우 콜백을 약한 참조(weak reference)로 저장하면 가비지 컬렉터가 즉시 수거해간다.