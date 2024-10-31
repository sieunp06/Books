# Item 6 불필요한 객체 생성을 피하라.
### content
  - [String 객체 생성](#string-객체-생성)
  - [재사용하라.](#재사용하라)
    - [정적 팩토리 메서드를 통해 재사용하라.](#정적-팩토리-메서드를-통해-재사용하라)
    - [생성 비용이 비싼 객체 또한 재사용하라.](#생성-비용이-비싼-객체-또한-재사용하라)
  - [어댑터(Adapter) 패턴을 사용한 객체 생성은 안전하지 않을 수 있다.](#어댑터adapter-패턴을-사용한-객체-생성은-안전하지-않을-수-있다)
  - [오토박싱은 불필요한 객체를 만들어 낼 수 있다.](#오토박싱은-불필요한-객체를-만들어-낼-수-있다)
  - [✨ 정리](#-정리)

---
## String 객체 생성
```java
String s = new String("bikini");
```

위와 같은 예시는 실행될 때마다 String 인스턴스를 새로 만든다.

이는 생성자에 넘겨진 `bikini` 자체가 이 생성자로 만들어내려는 String과 기능적으로 완전히 똑같다.

<br>

위 코드는 아래와 같이 반복문이 사용된 경우, 같은 인스턴스가 99개 만들어지게 된다.
```java
for (int i = 0; i < 99; i++) {
    String s = new String("bikini");
    System.out.println(s);
}
```

## 재사용하라.
### 정적 팩토리 메서드를 통해 재사용하라.
생성자 대신 정적 팩토리 메서드를 제공하는 불변 클래스에서는 정적 팩토리 메서드를 사용해 불필요한 객체 생성을 막을 수 있다.

`Boolean(String)` 생성자보다는 `Boolean.valueOf(String)` 팩토리 메서드를 사용하는 것이 좋다.

```java
public static Boolean valueOf(boolean b) {
	return b ? Boolean.TRUE : Boolean.FALSE;
}
```

생성자는 호출할 때마다 새로운 객체를 만들지만, 팩토리 메서드는 그렇지 않다.

### 생성 비용이 비싼 객체 또한 재사용하라.
아래 예시와 같이 생성 비용이 비싼 객체가 있다.
```java
static boolean isRomanNumeral(String s) {
    return s.matches("^(?=.)M*(C[MD]|D?C{0,3})"
                + "(X[CL]|L?X{0,3})(I[XV|V?IP{0,3}])$");
}
```
`String.matches`는 정규표현식으로 문자열 형태를 확인하는 가장 쉬운 방법이지만, 성능이 중요한 상황에서 반복해 사용하기에는 적합하지 않다.

내부에서 만드는 정규표현식용 Pattern 인스턴스는 한 번 쓰고 버려져서 곧바로 가비지 컬렉션 대상이 된다.

<br>

성능을 개선하려면 Pattern 인스턴스를 클래스 초기화 과정에서 직접 캐싱해서 재사용하는 방법을 사용하는 것이 좋다.

```java
public class RomanNumerals {
    private static final Pattern ROMAN = Pattern.compile(
        "^(?=.)M*(C[MD]|D?C{0,3})"
                + "(X[CL]|L?X{0,3})(I[XV|V?IP{0,3}])$");

    static boolean isRomanNumeral(String s) {
        return ROMAN.matcher(s).matches();
    }
}
```

위와 같이 개선하면 성능을 상당히 끌어올릴 수 있다.

<br>

만약 위와 같이 정성껏 초기화하여 개선했는데 한번도 호출하지 않는다면, ROMAN 필드는 쓸 데 없이 초기화된 꼴이다.

이떄는 `isRomanNumeral` 메서드가 처음 호출될 때 필드를 초기화하는 `지연 초기화`로 불필요한 초기화를 없앨 수 는 있지만, 권하지 않는다.

> 📌 [지연 초기화 - Item 67 최적화는 신중히 하라.]()

## 어댑터(Adapter) 패턴을 사용한 객체 생성은 안전하지 않을 수 있다.
어댑터는 실제 작업은 뒷단 객체에 위임하고 자신은 제 2의 인터페이스 역할을 해주는 객체다.

즉, 뒷단 객체만 관리하면 된다.

<br>

아래와 같이 `keySet()` 메서드는 Map 객체 안의 key들을 모두 담은 객체를 반환한다.

이때 Map 객체 내부의 원소를 바꾸고 keySet() 메서드를 다시 호출하면 완전히 다른 Set 인스턴스가 생성될 것 같지만, 실제로는 같은 Set 인스턴스를 반환한다.
```java
Map<Integer, String> map = new HashMap<>();
map.put(1, "hello");
map.put(2, "my name is");
map.put(3, "9");

Set<Integer> keySet1 = map.keySet();

System.out.println("map 출력 :" + map);
System.out.println("keySet1 출력 :" + keySet1);


map.remove(3);
Set<Integer> keySet2 = map.keySet();
System.out.println("map 출력 :" + map);
System.out.println("keySet1 출력 :" + keySet1);
System.out.println("keySet2 출력 :" + keySet2);

keySet1.remove(2);
Set<Integer> keySet3 = map.keySet();
System.out.println("map 출력 :" + map);
System.out.println("keySet1 출력 :" + keySet1);
System.out.println("keySet2 출력 :" + keySet2);
System.out.println("keySet3 출력 :" + keySet3);

/*
map 출력 :{1=hello, 2=my name is, 3=9}
keySet1 출력 :[1, 2, 3]
map 출력 :{1=hello, 2=my name is}
keySet1 출력 :[1, 2]
keySet2 출력 :[1, 2]
map 출력 :{1=hello}
keySet1 출력 :[1]
keySet2 출력 :[1]
keySet3 출력 :[1]
*/
```

따라서 keySet이 여러 개 만들어져도 상관은 없지만, 이는 낭비에 불과하다.

## 오토박싱은 불필요한 객체를 만들어 낼 수 있다.
오토박싱은 기본 타입과 그에 대응하는 박싱된 기본 타입의 구분을 흐려주지만, 완전히 없애주는 것은 아니다.

의미상으로는 별다를 것 없지만 성능에서는 그렇지 않다.

<br>

다음 예시는 모든 양의 정수의 총합을 구하는 메서드로, `int`는 충분히 크지 않으니 `long`을 사용해 계산한다.
```java
private static long sum() {
    Long sum = 0L;
    for (long i = 0; i <= Integer.MAX_VALUE; i++) {
        sum += i;
    }
    return sum;
}
```

위 예시에서는 `sum` 변수를 `Long`으로 선언해 불필요한 `Long` 인스턴스가 Integer.MAX_VALUE 만큼 생성되었다.

이때, `sum` 변수를 `long`으로 변경하기만 해도 성능이 향상된다.

<br>

박싱된 기본 타입보다는 기본 타입을 사용하고, 의도치 않은 오토박싱이 숨어들지 않도록 주의하자.

---
## ✨ 정리
`객체 생성은 비싸니 피해야 한다.`라는 의미가 아니다.

요즘 JVM에서는 별 다른 일을 하지 않는 작ㅇ느 객체를 생성하고 회수하는 일이 크게 부담되지 않는다.

프로그램의 명확성, 간결성, 기능을 위해서 객체를 추가로 생성하는 것이라면 일반적으로 좋은 일이다.

<br>

거꾸로, 아주 무거운 객체가 아닌 다음에야 단순히 객체 생성을 피하고자 객체 풀을 만들지는 말자는 뜻이다.
