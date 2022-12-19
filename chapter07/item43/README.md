# 아이템 43. 람다보다는 메서드 참조를 사용하라

## 메서드 참조
람다는 익명 클래스보다 간결하다.   
람다보다 메서드 참조(method reference)가 더 간결하다.

```
map.merge(key, 1, (count, incr) -> count + incr); // 람다
map.merge(key, 1, Integer::sum); // 메서드 참조
```

```java
// Example
public class Test {

  public static void main(String[] args) {
    Map<String, Integer> map1 = new HashMap<>();
    map1.put("Apple", 1000);
    map1.put("Banana", 2000);
    map1.put("Orange", 3000);

    Map<String, Integer> map2 = new HashMap<>();
    map2.put("Apple", 4000);
    map2.put("Tomato", 5000);
    map2.put("WaterMelon", 6000);

//    map2.forEach((key, value) -> map1.merge(key, value, (v1, v2) -> v1 + v2));
    map2.forEach((key, value) -> map1.merge(key, value, Integer::sum));
    
    System.out.println("map1 = " + map1);
  }
}
```

람다로 할 수 없는 일이라면 메서드 참조로도 할 수 없다(애매한 예외를 제외한다면). 
그렇더라도 람다로 구현했을 때 코드가 너무 길다면 메서드 참조가 보통은 더 짧고 간결하기 때문에 좋은 대안이 되어준다.

## 람다가 더 나은 경우
```
service.excute(GoshThisClassNameIsHumongous::action); // 메서드 참조
service.excute(() -> action()); // 람다
```

람다와 메서드 참조 중 더 간결한 것을 찾고, 만약 메서드 참조 코드가 코드 자체로 의미를 띄지 못할거면 코드가 좀 길어지더라도 람다의 매개변수 이름 등으로 의미를 명확하게 하는 것을 고려해보자.

## 메서드 참조의 유형
메서드 참조의 유형은 다섯 가지이다.
1. 정적 메서드를 가리키는 메서드 참조하는 유형
2. 수신 객체(receiving object; 참조 대상 인스턴스)를 특정하는 한정적(bound) 인스턴스 메서드 참조. 
   - 한정적 참조는 근본적으로 정적 참조와 비슷하다. 함수 객체가 받는 인수와 참조되는 메서드가 받는 인수가 똑같다. 
3. 수신 객체를 특정하지 않는 비한정적(unbound) 인스턴스 메서드 참조.
   - 비한정적 참조에서는 함수 객체를 적용하는 시점에 수신 객체를 알려준다.
   - 이를 위해 수신 객체 전달용 매개변수가 매개변수 목록의 첫 번째로 추가되며, 그 뒤로는 참조되는 메서드 선언에 정의된 매개변수들이 뒤따른다.
   - 비한정적 참조는 주로 스트림 파이프라인에서의 매핑과 필터 함수에 쓰인다.
4. 클래스 생성자를 가리키는 메서드 참조. 생성자 참조는 팩터리 객체로 사용된다.
5. 배열 생성자를 가리키는 메서드 참조.


메서드 참조 유형|예|	같은 기능을 하는 람다
|------|---|---|
|정적|Integer::parseInt|str -> Integer.parseInt(str)
|한정적(인스턴스)|Instant.now::isAfter|Instant then = Instant.now(); t -> then.isAfter(t)
|비한정적(인스턴스)|String::toLowerCase|str -> str.toLowerCase()
|클래스 생성자|TreeMap<K, V>::new|() -> new TreeMap<K, V>()
|배열 생성자|int[]::new|len -> new int[len]

https://stackoverflow.com/questions/35914775/java-8-difference-between-method-reference-bound-receiver-and-unbound-receiver
