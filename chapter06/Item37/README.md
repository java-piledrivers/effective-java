[**EnumMap vs HashMap**](https://sabarada.tistory.com/131) 

**EnumMap?**

- **열거 타입을 키로 사용하도록 설계한 아주 빠른 Map 구현체이다.**

---

### 정리

- **enum에 null이 사용하는 경우 NullPointerException을 발생시켜 문제가 발생한다.**
- **배열의 인덱스를 얻기 위해 ordinal을 쓰는 것을 일반적으로 좋지 않으니 EnumMap을 사용해야 한다.**
- **다차원 관계는 EnumMap<K, EnumMap<K, V>> 으로 표현하는 것이 좋다.**
- **Enum.ordinal()을 사용해서는 안된다. 아이템 35 사용하는 것은 일반적인 원칙의 특수한 사례이다.**

---

### ordinal 인덱싱의 문제점.

1. **배열은 제네릭과 호환되지 않으므로 비검사 형변환을 수행해야 하고 깔끔하게 컴파일되지 않는다. - ITEM28**
2. **배열을 인덱스에 대한 의미를 모르니 출력에 레이블을 직접 달아주어야 한다.**
3. **정수는 열거 타입과 다르게 타입 안전하지 않기 때문에 정확한 정수 값을 이용함을 직접 보증해야 한다.**

> **열거 타입을 키로 사용하도록 설계된 EnumMap을 사용해 위 문제점을 해결할 수 있다.**
> 

> **또한 스트림(Item45)를 사용해 맵을 관리하면 코드를 더 줄일 수 있다.**
> 

### 스트림을 사용한 코드 1 - EnumMap을 사용하지 않는다!

```java
public class Plant {
    public static void main(String[] args) {
        List<Plant> garden = new ArrayList<>(); 
        System.out.println(garden.stream().collect(groupingBy(plant -> plant.lifeCycle)));
    }
}
```

- 위 코드는 EnumMap이 아닌 고유한 맵 구현체를 사용했기 때문에 EnumMap을 써서 얻은 공간과 성능 이점이 사라진다.

### 스트림을 사용한 코드 2 - EnumMap을 이용해 데이터와 열거 타입을 매핑한다.

```java
public class Plant {
    public static void main(String[] args) {
        List<Plant> garden = new ArrayList<>(); 
        System.out.println(garden.stream().collect(
                groupingBy(plant -> plant.lifeCycle,
                        () -> new EnumMap<>(LifeCycle.class), toSet())));
    }
}
```
