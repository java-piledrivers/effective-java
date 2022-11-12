# 아이템28. 배열보다는 리스트를 사용하라

## 공변과 불공변

### 공변 (covariant)
- 배열
- 함께 변한다는 뜻.
- Sub가 Super의 하위 타입이라면 배열 Sub[] 는 배열 Super[]의 하위타입

### 불공변 (invariant)
- 제네릭
- 변하지 않는다는 뜻
- 서로 다른 타입 Type1과 Type2가 있을 때, List<Type1>은 List<Type2>의 하위 타입도 아니고 상위 타입도 아니다.

```java
class Covariant {
  public static void main(String[] args) {
    Object[] objectArray = new Long[1];
    objectArray[0] = "타입이 달라 넣을 수 없다."; // ArrayStoreException을 던진다.
  }
}

class Invariant {
  public static void main(String[] args) {
    List<Object> ol = new ArrayList<Long>();
    ol.add("타입이 달라 넣을 수 없다.");
  }
}
```

## 실체화(reify)
### 실체화 타입
자신의 타입 정보를 런타임에도 알고 있는 것.
### 비 구체화 타입
런타임 시에 소거(erasure)되기 때문에 런타임에는 컴파일타임보다 타입 정보를 적게 가지는 타입이다.    
E, List<E>, List<String> 과 같은 타입을 실체화 불가 타입 혹은 비 구체화 타입이라고 한다.
### 소거
실행시간에 오버헤드가 발생하지 않도록 컴파일 타임에만 타입 제약 조건을 정의하고, 런타임에는 타입을 제거한다는 뜻이다.

### 배열 
- 실체화된다.
- 배열은 런타임에도 자신이 담기로 한 원소의 타입을 인지하고 확인한다. 그래서 Long 배열에 String을 넣으려 하면 ArrayStoreException이 발생하는 것이다.
### 제네릭 
- 타입 정보가 런타임에는 소거(erasure)된다.
- 이는 원소 타입을 컴파일에만 검사하며 런타임에는 알 수조차 없다는 뜻이다.

### 제네릭 배열
- 배열은 제네릭 타입, 매개변수화 타입, 타입 매개변수로 사용할 수 없다.
- new List<E>[], new List<String[]>, new E[]와 같이 사용하려고 하면 컴파일할 때 제네릭 배열 생성 오류를 일으킨다.
- 제네릭 배열을 만들 수 있게 되면 타입 안전하지 않다.

```java
class GenericArray {
    public static void main(String[] args) {
        List<String>[] stringLists = new List<String>[1]; // 된다고 가정
        List<Integer> intList = List.of(42); 
        Object[] objects = stringLists; // 공변이라 OK
        objects[0] = intList; 
        String s = stringLists.get(0); 
    }
}
```

## issue
>"소거 메커니즘 때문에 매개변수화 타입 가운데 실체화될 수 있는 타입은 List<?> 와 Map<?,?> 같은 비한정적 와일드카드 타입뿐이다."   
윗 문장 잘 이해가 안가네요. "실체화"가 무엇을 의미하는지부터 짚고 넘어가아할거 같은데 수재님이 이해하신 부분에 대해서 공유해주시면 감사하겠습니다

https://docs.oracle.com/javase/tutorial/java/generics/unboundedWildcards.html
1. Object class 를 넣는 것과 같은 용도인 경우.
2. Collection 을 parameter 로 받으면서 List.size, List.clear 등과 같이 type 에 dependency 가 없는 Collection 자체의 함수들만 호출하는 경우.

