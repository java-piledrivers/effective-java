# 46. **스트림에서는 부작용 없는 함수를 사용하라**

---

### **스트림 패러다임**

스트림 패러다임의 핵심은 계산을 일련의 변환(transformation)으로 재구성하는 부분

이때 각 변환 단계는 가능한 한 이전 단계의 결과를 받아 처리하는 `순수 함수`여야 한다.

`순수 함수`란 오직 입력만이 결과에 영향을 주는 함수이다.

- 다른 가변 상태를 참조하지 않고, 함수 스스로도 다른 상태를 변경하지 않는다.
- 이렇게 하려면 스트림 연산에 건네는 함수 객체는 모두 부작용(side effect)이 없어야 한다.

### **스트림 패러다임을 이해하지 못한 채 API만 사용했다 - 따라 하지 말 것!**

```java
Map<String, Long> freq = new HashMap<>();
try (Stream<String> words = new Scanner("file").tokens()) {
    words.forEach(word -> freq.merge(word.toLowerCase(), 1L, Long::sum));
}
```

- 스트림 코드를 가장한 반복적 코드다.

- 위 코드의 모든 작업은 종단 연산인 forEach에서 일어나는데, 이때 외부 상태(빈도표)를 수정하는 람다를 실행하면서 문제가 생긴다.

### **스트림을 제대로 활용해 빈도표를 초기화한다**

```java
Map<String, Long> freq = new HashMap<>();
try (Stream<String> words = new Scanner("file").tokens()) {
    words.collect(groupingBy(String::toLowerCase, counting()));
}

itemRepository.findById(itemId).orElseThrow(() -> new 원하는 exception());
Optional.of();

num += 3;
```

- for-each 반복문은 forEach 종단 연산과 비슷하게 생겼다.

하지만 forEach 연산은 종단 연산 중 기능이 가장 적고 대놓고 반복적이라서 병렬화할 수도 없다.

**forEach 연산은 스트림 계산 결과를 보고할 때만 사용하고, 계산하는 데는 쓰지 말도록 한다.**

---

## Collectors, 수집기

| 동작 | 메소드 |
| --- | --- |
| 루핑 | forEach() |
| 매칭 | allMatch(), anyMatch(), noneMatch() |
| 집계 | count(), max(), min(), average(), sum(), reduce() |
| 조사 | findFirst(), findAny() |
| 수집 | collect() |
- java.util.stream.Collectors 클래스는 `스트림의 원소들을 객체 하나에 취합`하는 여러 메서드를 제공해준다.
- 이를 이용하면 스트림의 원소를 손쉽게 컬렉션으로 모을 수 있으며, toList(), toSet(), toCollection(collectionFactory) 등이 있다.
- Collectors의 멤버를 정적 임포트하여 쓰면 스트림 파이프라인의 가독성을 향상시킬 수 있다.

수집기에는 아래의 것들을 비롯해 여러 가지 종류가 있다.

- toMap, groupingBy, partitioningBy 등의 메서드가 있다.
- counting, filtering, mapping, flatMapping, collectiongAndThen 등의 메서드가 있다.
- minBy, maxBy는 인수로 받은 비교자를 이용해 스트림에서 값이 가장 작은 혹은 가장 큰 원소를 찾아 반환한다.
    - Stream 인터페이스의 min과 max 메서드를 살짝 일반화한 것이자, java.util.function.BinaryOperator의 minBy와 maxBy 메서드가 반환하는 이진 연산자의 수집기버전이다.
- joining 메서드는 문자열 등의 CharSequence 인스턴스의 스트림에만 적용할 수 있다.
    - 매개변수가 없는 joining은 단순히 원소들을 연결(concatenate)하는 수집기를 반환한다.
    - 인수 하나짜리 joining은 CharSequence 타입의 구분문자(delimiter)를 매개변수로 받으며, 연결 부위에 이 구분문자를 삽입하여 연결한 결과를 만들어준다.
    - 인수 3개짜리 joining은 구분문자에 더해 접두문자(prefix)와 접미문자(suffix)도 받는다.

---

## 정리

- 스트림 파이프라인 프로그래밍의 핵심은 부작용 없는 함수 객체에 있다.
    - 스트림뿐 아니라 스트림 관련 객체에 건네지는 모든 함수 객체가 부작용이 없어야 한다.
- 종단 연산 중 forEach는 스트림이 수행한 계산 결과를 보고할 때만 이용해야 한다. 계산 자체에는 이용하지 않도록 한다.
- 스트림을 올바로 사용하려면 수집기를 잘 알아야 하며, 가장 중요한 수집기 팩터리는 toList, toSet, toMap, groupingBy, joining이다.
