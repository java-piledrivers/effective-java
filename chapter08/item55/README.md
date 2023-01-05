# [아이템55] 옵셔널 반환은 신중히 하라

## 메서드가 값을 반환할 수 없을 때 
1. 예외 던지기
   - 예외 생성시 스택 추적 전체를 캡처해야 하는 꽤 큰 비용이 든다.
2. null 반환
   - 사용하는 쪽에서 별도의 null 처리 코드를 추가해야 한다.
   - 무시할 경우 실제 원인과는 상관없는 코드에서 NullPointException 발생
3. Java8부터 `Optional<T>` 반환
   - 옵셔널을 반환하는 메서드는 예외를 던지는 메서드보다 유연하고 사용하기 쉬우며, null을 반환하는 메서드보다 오류 가능성이 적다.

---

## `Optional<T>`이란
- 보통은 T를 반환해야하지만 특정 조건에서는 아무것도 반환하지 않아야 할 때 T 대신  `Optional<T>`를 반환하도록 선언
- T 타입 참조를 하나 담거나, 아무것도 담지 않은 `불변` 컬렉션.
- 아무것도 담지 않은 옵셔널 : `비었다` 라고 표현
- 하나의 값을 담은 옵셔널 : `비어있지 않다` 라고 표현

---

## 옵셔널 반환 선택 기준
**옵셔널은 검사 예외(checked Exception)와 취지가 비슷**하다.   
값이 없을 수도 있음을 API 사용자에게 명확히 알려준다.
검사 예외를 던지면 클라이언트에서는 반드시 이에 대처하는 코드를 작성해야 한다.

### **Optional 반환 시 클라이언트 대처 코드**

#### 1. 기본값 설정 
- orElse
- orElseGet : 초기 설정 비용 낮출 수 있다.
```java
String result = max(words).orElse("단어없음");
max(words).orElseGet(Supply<T>)
```

#### 2. 예외 던지기
- 실제 예외가 아닌 `예외 팩터리` 건넴 : 예외가 실제로 발생하지 않는한 **예외 생성 비용은 들지 않는다.**

```java
String result = max(words).orElseThrow(TemperException::new); //예외 팩터리 건넴
```

#### 3. 값이 항상 채워져있음을 확신하면 바로 꺼내쓴다.
- get() 
- 그러나 값이 없을 때는 `NoSuchElementException` 발생.
```java
String result = max(words).get();
```

#### 3. filter, map, flatMap, ifPresent 사용
- 기본 메서드로 처리할기 어려울 때 사용 검토

#### 4. isPresent 사용
- 다른 메서드로도 대부분 대체 가능하므로 신중히 사용

### 옵셔널 사용 예시
1. 스트림을 사용한다면 옵셔널들을 `Stream<Optional<T>>`로 받아 채워진 옵셔널들에서 값을 뽑아 `Stream<T>` 에 건네 담아 처리
   ```java
   streamOfOptionals
      .filter(Optional::isPresent) // 옵셔널에 값이 있다면
      .map(Optional::get)      //  그 값을 스트림에 매핑
   ```
2. Java9에서 stream() 메서드
    - **Optional을 Stream으로 변환해주는 어댑터**
    - 값이 있으면 값을 원소로 담은 스트림으로, 값이 없다면 빈 스트림으로 변환
    - 1번 코드를 java9의 stream()메서드를 사용해 아래와 같이 변환 
   ```java
    streamOfOptionals
       .flatMap(Optional::stream)
   ```


---


## 2. 옵셔널 사용시 유의 사항

### 2-1 옵셔널을 반환하는 메서드는 절대 null을 반환하지 말자
- Optional.of(value) 에 null을 넣으면 NullPointException 발생.
- null을 넣을 경우 Optional.ofNullable(value) 사용

### 2-2 결기본 타입을 담은 옵셔널은 전용 Optional 사용
- 기본 타입을 담은 옵셔널은 전용 Optional을 사용한다. 박싱된 기본 타입을 담은 옵션널 반환하지 않도록 주의
  - 예 :  `OptionalInt`, `Optional`
- 그러나, Boolean, Byte, Character, SHort, Float는 예외일 수 있다.

### 2-3 사용하면 안되는 곳
- Map의 value
- 컬렉션의 key
- 원소, 배열의 원소로 사용
- 인스턴스 필드로 사용

#### 2-3-1 옵셔널을 인스턴스 필드에 저장해두는 게 필요할 때가 있을까?
- 이런 상황은 대부분 필수 필드를 갖는 클래스와 이를 확장해 선택적 필드를 추가한 하위 클래스를 따로 만들어야 함을 암시하는 `나쁜 냄새`
- 하지만 `가끔 적절한 상황도` 있다. 
  - 예 : item2의 NutritionFacts 클래스 
  - 인스턴스의 필드 중 상당수는 필수가 아니다. 또한 그 필드들을 기본 타입이라 값이 없음을 나타낼 방법이 마땅치 않다. 
  - 이런 선택적 필드의 `getter 메서드들이 옵셔널을 반환`하게 해주면 좋을 것이다. 
  - 또는 `필드 자체를 옵셔널로 선언`

```java
public class NutritionFacts{
	private final int servingSize;  //필수
	private final int servings;     //필수
	private final OptionalInt calories;     //선택
	private final OptionalInt fat;          //선택
	private final OptionalInt sodium;       //선택
	private final int carbohydrate;         //선택

   public OptionalInt getCarbohydrate() {     
      return Optional.ofNullable(carbohydrate);
   }
```

---

## Optional 단점
- 컬렉션, 스트림, 배열, 옵셔널같은 컨테이너 타입은 옵셔널로 감싸면 안된다.
- 빈 `Optional<T>`보단 빈 `List<T>`를 반환하는게 좋다. 왜냐면 클라이언트에 옵셔널 처리 코드를 넣지 않아도 되기 때문
