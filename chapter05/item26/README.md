
### 목차
- raw 타입은 사용하지 말자
- 용어정리
- `List와 List<Object>`의 차이
- `Set과 Set<?>` 차이
- raw type 사용하는 경우

---

#  [아이템26] raw 타입은 사용하지 말자

`제네릭 클래스/제네릭 인터페이스` : 타입 매개변수(Type Paramter)를 클래스와 인터페이스 선언에 사용한 것. 통들어 `제네릭 타입`이라고 한다.

<br>

raw type은 컬렉션에 넣을 때는 unchecked call 경고가 발생하고, 명시적 형변환을 사용해 컬렉션에서 꺼낼 때 에러가 발생한다. 

`오류는 가능한 한 발생 즉시, 이상적으로는 컴파일할 때 발견하는 것이 좋다.` 
raw type은 런타임에 문제를 겪는 코드와 `원인을 제공한 코드가 물리적으로 떨어져` 있을 가능성이 커진다. 

```java
private final Collection stamps = new ArrayList();
stamps.add(new Coin());; //unchecked call 경고

for(Iterator i = stamps.iterator(); i.hasNext();) {
    Stamp stamp = (Stamp) i.next(); //ClassCastException 발생
}
```

<br>

---

## 용어 정리
- 제네릭 클래스[인터페이스] : 클래스[인터페이스] 선언에 타입 매개변수(type parameter)가 쓰인다. Example.class
- 제네릭 타입(Generic Type) : 제네릭 클래스와 제네릭 인터페이스를 통틀어 이르는 말. Example<T>

|용어|영문|예|
|----|----|--|
|매개변수화 타입|parameterized Type|List<String>|
|제네릭 타입|generic type|List<E>|
|정규 타입 매개 변수|formal type parameter|E|
|실제 타입 매개 변수|actual type parameter|String|
|로 타입|raw type|List|
|비한정적 와일드카드 타입|unbounded wildcard type|List<?>|
|한정적 와일드카드 타입|bounded wildcard type|List<? extends Number>|
|재귀적 타입 한정|recursive type bound|<T extends Comparable<T>|
|제네릭 메서드|generic method|static <E> List<E> asList(E[] a)|
|타입 토큰|type token|String.class|


---

## `List와 List<Object>`의 차이

raw type과 Object를 타입 매개변수로 사용하는 것의 차이

- `List` : 제네릭 타입을 완전히 적용하지 않은 것
- `List<Object>` : 모든 타입을 허용한다는 의사를 컴파일러에게 전달한 것 

`List<Object>` 같은 매개변수화 타입을 사용할 때와 달리 List 같은 로 타입을 사용하면 타입 안정성을 잃게 된다. 

- List raw type 사용 - 런타임시 ClassCastException 발생
  - class java.lang.Integer cannot be cast to class java.lang.String 

```java
public static void main(String[] args) {
		List<String> strings = new ArrayList<>();
		unsafeAdd(strings, Integer.valueOf(42));
		String s = strings.get(0);
	}

	private static void unsafeAdd(List list, Object o) {
		list.add(o);
	}
```

- `List<Object>` 사용
  - 컴파일되지 않으므로 타입 안정성 보장

```java
	private static void unsafeAdd(List<Object> list, Object o) {
		list.add(o);
	}
```

---

## ? 사용 - 비한정적 와일드카드 타입 

어떤 타입이라도 담을 수 있는 가장 범용적인 매개변수화 타입으로 제네릭 타입을 사용하고 싶지만 `타입 매개변수가 무엇인지 신경 쓰고 싶지 않을 때` 비한정적 와일드카드 타입(unbounded wildcard type)을 사용한다.

예를 들면, 2개의 Set을 받아 공통 원소를 반환하는 메서드가 필요할 때 로 타입보다는 비한정적 와일드카드 타입을 사용한다. 

- 잘못된 예 : raw type 사용
    ```java
    static int numElementsInCommon(Set s1, Set s2) {
            int result = 0;
            for (Object o : s1)
                if (s2.contains(o)) 
                    result++;
            return result;
        }
    ```

- 좋은 예 : 비한정적 와일드카드 타입 사용
    ```java
    static int numElementsInCommon(Set<?> s1, Set<?> s2) {
            int result = 0;
            for (Object o : s1)
                if (s2.contains(o)) 
                    result++;
            return result;
        }
    ```

### `Set과 Set<?>` 차이

와일드카드 타입은 안전하고 raw type 은 타입 불변식을 훼손하기 쉽다. 
- [ ] `Collection<?>`는 null외에는 어떤 원소도 넣을 수 없고, 컬렉션에서 꺼낼 수 있는 객체의 타입도 알 수 없다. 

비한정적 와일드카드의 제약을 받아들일 수 없으면 `제네릭 메서드 또는 한정적 와일드카드 타입`을 사용하면 된다. 

--- 

## raw type 사용하는 경우

### 1. class 리터럴에는 raw type을 사용해야 한다.
자바 명세에 class 리터럴에 매개변수화 타입을 사용하지 못하게 했다. 
- 허용 : List.class, String[].class, int.class
- 비허용 : `List<String>.class`, `List<?>.class`

    ```java
    Class<List> listClass = List.class;
    Class<String[]> stringArrClass = String[].class;
    Class<Integer> integerClass = int.class;
    ```

### 2. instanceof 연산자 사용시에는 raw type을 사용해야 한다.
제네릭은 런타임에 타입 정보가 지워지므로 `instanceof` 연산자는 매개변수화 타입에는 적용할 수 없다. 

- 제네릭 타입에 instanceof를 사용하는 올바른 예
  - o의 타입이 Set임을 확인한 다음, 와일드카드 타입인 Set<?>로 형변환해야 한다.
    checked cast 이므로 컴파일 경고가 뜨지 않는다.
  ```java
    if(o instanceof set) {
        Set<?> s = (Set<?>) o; //와일드카드 타입
    }
  ```

