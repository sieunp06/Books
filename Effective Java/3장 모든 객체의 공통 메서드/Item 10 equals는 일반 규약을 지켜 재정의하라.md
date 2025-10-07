# Item 10. equals는 일반 규약을 지켜 재정의하라.

`equals` 메서드는 재정의하기 쉬워보이지만 자칫하면 끔찍한 결과를 초래한다.

문제를 회피하는 가장 쉬운 방법은 재정의 하지 않는 것이지만, 그냥 두면 그 클래스의 인스턴스는 오직 자기 자신과만 같게 된다.

## equals를 재정의하지 말아야 하는 경우들

1. 각 인스턴스가 본질적으로 고유하다.
    - 값을 표현하는 것이 아니라 동작하는 개체를 표현하는 클래스가 해당한다.
    - ex) Thread
2. 인스턴스의 ‘논리적 동치성(logical equality)’를 검사할 일이 없다.
    - ex) java.util.regex.Pattern
        
        ```java
        Pattern p1 = Pattern.compile("a*b");
        Pattern p2 = Pattern.compile("a*b");
        
        System.out.println(p1 == p2);        // false (서로 다른 객체)
        System.out.println(p1.equals(p2));   // false (동일성만 비교)
        ```
        
        - 두 Pattern 인스턴스는 같은 정규식을 나타내지만, equals를 재정의하지 않았기 때문에 Object의 기본 구현만 따른다.
        - java.util.regex.Pattern의 경우, Pattern 인스턴스끼리 비교할 일이 없고 주로 정규식을 매칭하는 데에만 사용한다. 때문에 논리적 동치성을 검사할 필요가 없다.
3. 상위 클래스에서 재정의한 equals가 하위 클래스에도 딱 들어맞는다.
    - ex) Set
        - 대부분의 Set 구현체는 AbstractSet이 구현한 equals를 상속받아 사용한다.
4. 클래스가 private이거나 package-private이고 equals 메서드를 호출할 일이 없다.

## equals를 재정의해야 하는 경우

> 객체 식별성(obejct identity; 두 객체가 물리적으로 같은가)이 아니라 논리적 동치성을 확인해야 하는데, 상위 클래스의 equals가 논리적 동치성을 비교하도록 재정의되지 않았을 때
> 

ex) `Integer`, `String`처럼 값을 표현하는 클래스

### equals를 재정의할 때 따라야 하는 일반 규약

equals 메서드는 동치관계를 구현하며, 다음을 만족한다.

- 반사성(reflexivity): null이 아닌 모든 참조 값 x에 대해, `x.equals(x)`는 true다.
- 대칭성(symmetry): null이 아닌 모든 참조 값 x, y에 대해, `x.equals(y)`가 true면 `y.equals(x)`도 true다.
    - 대칭성을 위배하는 경우
        
        ```java
        public final class CaseInsensitiveString {
        	private final String s;
        	)
        	public CaseInsensitiveString(String s) {
        		this.s = Objects.requireNonNull(s);
        	}
        	
        	// 대칭성 위배
        	@Override
        	public boolean equals(Object o) {
        		if (o instanceof CaseInsensitiveString)
        			return s.equalsIngnoreCase(
        							((CaseInsensitiveString) o).s);
        		if (o instanceof String)    // 한 방향으로만 작동한다.
        			return s.equalsIgnoreCase((String) o);
        		return false;
        	}
        }
        ```
        
        `CaseInsensitiveString`의 equals는 일반 문자열과도 비교를 시도한다.
        
        ```java
        CaseInsensitive cis = new CaseInsensitiveString("polish");
        String s = "polish";
        ```
        
        `cis.equals(s)`는 true를 반환한다.
        
        하지만 `CaseInsensitiveString`의 equals는 일반 String의 존재를 알고 있지만, String의 경우 CaseInsensitiveString의 존재를 모르기 때문에 `s.equals(cis)`는 false를 반환하게 돼 대칭성을 위반하게 된다.
        
- 추이성(transitivity): null이 아닌 모든 참조 값 x, y, z에 대해, `x.equals(y)`가 true이고 `y.equals(z)`도 true이면 `x.equals(z)`도 true다.
    - 상위 클래스에는 없는 새로운 필드를 하위 클래스에 추가하는 상황
        
        ```java
        public class Point {
        	private final int x;
        	private final int y;
        	
        	public Point(int x, int y) {
        		this.x = x;
        		this.y = y;
        	}
        	
        	@Override
        	public boolean equals(Object o) {
        		if (!(o instanceof Point))
        			return false;
        		Point p = (Point) o;
        	}
        }
        ```
        
        위 클래스를 확장해서 점에 색상을 더하도록 하는 코드는 다음과 같다.
        
        ```java
        public class ColorPoint extends Point {
        	private final Color color;
        	
        	public ColorPoint(int x, int y, Color color) {
        		super(x, y);
        		this.color = color;
        	}
        }
        ```
        
        `equals` 메서드를 구현하지 않으면 `Point`의 equals를 그대로 상속받아 사용하게 된다.
        
        하지만 이는 색상 정보를 무시한 채 비교를 수행한다.
        
    - 추이성을 위반한 경우
        
        ```java
        @Override
        public boolean equals(Object o) {
        	if (!(o instanceof Point))
        		return false;
        	
        	// o가 일반 Point면 색상을 무시하고 비교한다.
        	if (!(o instanceof ColorPoint))
        		return o.equals(this);
        		
        	return super.equals(o) && ((ColorPoint) o).color == color;
        }
        ```
        
        위 방식은 대칭성은 만족하지만, 추이성은 만족하지 못한다.
        
        ```java
        ColorPoint p1 = new ColorPoint(1, 2, Color.RED);
        Point 2 = new Point(1, 2);
        ColorPoint p3 = new ColorPoint(1, 2, Color.BLUE);
        ```
        
        `p1.equals(p2)`와 `p2.equals(p3)`는 true를 반환하는 데에 반해 `p1.equals(p3)`는 false를 반환한다.
        
    - 리스코프 치환 원칙에 따르면, 어떤 타입에 있어 중요한 속성이라면 그 하위 타입에서도 마찬가지로 중요하다. 즉, `Point`의 하위 클래스는 정의상 여전히 `Point`이므로 어디서든 `Point`로써 활용될 수 있어야 한다.
    - 상속 대신 컴포지션을 사용하고, 뷰 메서드를 제공하면 된다.
        
        ```java
        // 좌표를 나타내는 기존 클래스
        public class Point {
            private final int x;
            private final int y;
        
            public Point(int x, int y) {
                this.x = x;
                this.y = y;
            }
        
            @Override
            public boolean equals(Object o) {
                if (!(o instanceof Point)) return false;
                Point p = (Point) o;
                return x == p.x && y == p.y;
            }
        
            @Override
            public String toString() {
                return "(" + x + ", " + y + ")";
            }
        }
        
        // 색상까지 포함하는 새 클래스
        public class ColorPoint {
            private final Point point;   // Point를 상속하지 않고 포함(Composition)
            private final String color;
        
            public ColorPoint(int x, int y, String color) {
                this.point = new Point(x, y);
                this.color = color;
            }
        
            // equals는 오직 ColorPoint끼리 비교
            @Override
            public boolean equals(Object o) {
                if (!(o instanceof ColorPoint)) return false;
                ColorPoint cp = (ColorPoint) o;
                return point.equals(cp.point) && color.equals(cp.color);
            }
        
            // 뷰 메서드
            public Point asPoint() {
                return point;
            }
        }
        ```
        
        ```java
        Point p = new Point(1, 2);
        ColorPoint cp = new ColorPoint(1, 2, "RED");
        
        // 좌표만 비교하고 싶다면 asPoint()를 사용
        System.out.println(cp.asPoint().equals(p));   // true
        ```
        
- 일관성(consistency): null이 아닌 모든 참조 값 x, y에 대해, `x.equals(y)`를 반복해서 호출하면 항상 true를 반환하거나 항상 false를 반환한다.
- null 아님: null이 아닌 모든 참조 값 x에 대해, `x.equals(null)`은 false이다.

## equals 메서드 구현 방법

1. `==` 연산자를 사용해 입력이 자기 자신의 참조인지 확인한다.
    - 자기 자신이면 true를 반환한다.
2. `instanceof` 연산자로 입력이 올바른 타입인지 확인한다. 그렇지 않는다면 false를 반환한다.
    - 이때의 올바른 타입은 equals가 정의된 클래스인 것이 보통이지만, 가끔 그 클래스가 구현된 특정 인터페이스가 될 수도 있다.
3. 입력을 올바른 타입으로 형변환한다.
4. 입력 객체와 자기 자신의 대응되는 ‘핵심’ 필드들이 모두 일치하는지 하나씩 검사한다.
    - 모든 필드가 일치하면 true를, 하나라도 다르면 false를 반환한다.
5. equals를 재정의할 땐 hashCode도 반드시 재정의하자.