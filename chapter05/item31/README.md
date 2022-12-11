# 31. 한정적 와일드카드를 사용해 API 유연성을 높이라

### 아이템 28 잠깐 복습

Sub가 Super의 하위타입이라면

배열 Sub[]는 배열 Super[]의 하위타입이다 ⇒ 공변

그러나 List<Sub>는 List<Super>의 하위타입도, 상위타입도 아니다 ⇒ 불공변 

---

### 생산자(Producer) 예시

생산자 : 입력 매개변수로부터 이 컬렉션으로 원소를 옮겨담는 역할

```java
public void pushAll(Iterable<E> src) {
    for (E e : src) {
        push(e);
    }
}
```

위 코드의 문제점은 무엇일까?

```java
Stack<Number> numberStack = new Stack<>();
Iterable<Integer> integers = ...;

// incompatible types...
// Iterable<Integer> cannot be converted t Iterable<Number>
numberStack.pushAll(integers);
```

Iterable 타입 역시 불공변이므로 문제가 생긴다.

이런 상황에 대처할 수 있는 것이 `한정적 와일드카드 타입`이다.

pushAll의 입력 매개변수 타입은 ‘E의 Iterable’이 아니라 ‘E의 하위 타입의 Iterable’이어야 하는데

그것을 표현한 것이 `Iterable<? extends E>` 이다

```java
public void pushAll(Iterable<? extends E> src) {
    for (E e : src) {
        push(e);
    }
}
```

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/dd2b6774-77bd-4ac8-b5c1-4f9f38383cf9/Untitled.png)

---

### 소비자(Consumer) 예시

소비자 : 이 컬렉션 인스턴스의 원소를 입력 매개변수로 옮겨담는 역할

```java
public void popAll(Collection<E> dst) {
    while(!isEmpty()) {
        dst.add(pop());
    }
}
```

위 코드 역시 pushAll과 비슷한 문제를 발생시킨다.

```java
Stack<Number> numberStack = new Stack<>();
Collection<Object> objects = ...;

// Collection<Object>는 Collection<Number>의 하위 타입이 아니다.
// 와 같은 생산자 때와 비슷한 오류를 낸다.
numberStack.popAll(objects);
```

이 경우 역시 `한정적 와일드카드`를 통해 해결 가능하다.

popAll의 입력 매개변수의 타입이 ‘E의 Collection’이 아니라 ‘E의 상위 타입의 Collection’이어야 한다.

이것을 표현한것이 `Iterable<? super E>` 이다

---

위에서 설명한 두가지 예시에 대한 규칙을 다음과 같은 공식으로 기억하자

> 펙스(PECS) : producer-extends, consumer-super
> 

PECS 공식은 와일드카드를 사용하는 기본 원칙이다.
