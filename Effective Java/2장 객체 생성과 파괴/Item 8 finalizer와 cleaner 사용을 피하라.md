# Item 8. finalizer와 cleaner 사용을 피하라.

자바는 두 가지 객체 소멸자를 제공한다.

- `finalizer`
    - 예측할 수 없고, 상황에 따라 위험할 수 있어 일반적으로 불필요하다.
- `cleaner`
    - 자바 9에서 `finalizer`를 deprecated API로 지정하고, `cleaner`를 그 대안으로 소개했다.
    - cleaner는 finalizer보다는 덜 위험하지만, 여전히 예측할 수 없고, 일반적으로 불필요하다.

## finalizer와 cleaner의 문제점 1 - 수행 시점 및 여부 보장하지 않음.

`finalizer`와 `cleaner`는 즉시 수행된다는 보장이 없다.

객체에 접근할 수 없게 된 후 finalizer와 cleaner가 실행되기까지 얼마나 걸릴지 알 수 없다.

즉, finalizer와 cleaner로는 제 때 실행되어야 하는 작업은 절대 할 수 없다.

`finalizer`와 `cleaner`를 얼마나 신속히 수행할지는 전적으로 가비지 컬렉터 알고리즘에 달렸으며, 이는 가비지 컬렉터 구현마다 천차만별이다.

또한 `finalizer`나 `cleaner` 수행 시점에 의존하는 프로그램의 동작 또한 마찬가지다.

자바 언어 명세는 `finalizer`나 `cleaner`의 수행 시점 뿐 아니라 수행 여부조차 보장하지 않는다.

접근할 수 없는 일부 객체에 딸린 종료 작업을 전혀 수행하지 못한 채 프로그램이 중단될 수도 있다는 것이다.

따라서 프로그램 생애주기와 상관없는, 상태를 영구적으로 수정하는 작업에서는 절대 finalizer나 cleaner에 의존해서는 안된다.

## finalizer와 cleaner의 문제점 2 - 예외 무시 후 즉시 종료

`finalizer` 동작 중 발생한 예외는 무시되며, 처리할 작업이 남았더라도 그 순간 종료된다.

잡지 못한 예외 때문에 해당 객체는 마무리가 덜 된 상태로 남을 수 있따.

그리고 다른 스레드가 이처럼 훼손된 객체를 사용하려 한다면 어떻게 동작할지 예측할 수 없다.

또한 `finalizer`에서 이런 일이 발생한다면 경고조차 출력하지 않는다.

`cleaner`의 경우는 이런 문제가 발생하지 않는다.

## finalizer와 cleaner의 문제점 3 - 심각한 성능 문제

간단한 `AutoCloseable` 객체를 생성하고 수거하기까지의 시간을 `try-with-resources`를 사용한 경우와 `finalizer`를 사용한 경우를 비교하면, 약 50배정도의 차이가 발생하며 finalizer가 느렸다.

`finalizer`가 가비지 컬렉터의 효율을 떨어뜨리기 때문이다.

## finalizer와 cleaner의 문제점 4 - 심각한 보안 문제

생성자나 직렬화 과정에서 예외가 발생하면, 이 생성되다 만 객체에서 악의적인 하위 클래스의 finalizer가 수행될 수 있게 된다.

이 finalizer는 정적 필드에 자신의 참조를 할당하여 가비지 컬렉터가 수집하지 못하게 막을 수 있다.

객체 생성을 막으려면 생성자에서 예외를 던지는 것만으로 충분하지만, finalizer가 있다면 그렇지도 않다.

final이 아닌 클래스를 finalizer 공격으로부터 방어하려면 아무 일도 하지 않는 finalizer 메서드를 만들고 final로 선언하면 된다.