[item 42] 익명 클래스보다는 람다를 사용하라
====
전략 패턴 처럼, 함수 객체를 사용하는 과거 객체 지향 디자인 패턴에는 익명 클래스면 충분했으나, 
익명 클래스의 코드가 너무 길기 때문에 자바는 함수형 프로그래밍에 적합하지 않습니다.

자바 8부터 추상 메서드 하나짜리 인터페이스는 특별한 의미를 인정받아 대우를 받게 되었습니다.
함수형 인터페이스를 통해 람다식을 만들 수 있게 되었고, 자질구레한 코드 없이 어떤 동작을 하는지가 명확하게 드러나게 할 수 있습니다.

예시로 Comparator 인터페이스는 정렬을 담당하는 추상 전략을 뜻하며,문자열을 정렬하는 구체적인 전략을 익명 클래스로 구현할 수 있습니다.
```java
// 익명 클래스 사용
Collections.sort(words, new Comparator<String>() {
    @Override
    public int compare(String s1, String s2) {
        return Integer.compare(s1.length(), s2.length());
    }
});
```

람다식을 사용한 경우입니다. 간결하게 의도를 표현합니다.
```java
// 람다식 사용
Collections.sort(words,
        (s1, s2) -> Integer.compare(s1.length(), s2.length());
```

### 타입을 명시해야 코드가 더 명확할 때만 제외하고는, 람다의 모든 매개변수 타입은 생략하자.
- 컴파일러가 타입 추론을 해줍니다.
  - 위 코드를 보면, 매개변수인 `Comparator<String>`, `String`과 반환값인 `int`는 람다식에서는 언급이 없습니다.
- 일단 생략하고, 컴파일러가 "타입을 알 수 없다"는 오류를 낸다면, 해당 타입을 명시하기

컴파일러가 타입 추론을 하는데 필요한 타입 정보 대부분을 제네릭에서 얻기 때문에, 제네릭을 통해 정보를 잘 제공해야 합니다. (타입 추론 규칙은 자바 언어 명세의 장 하나를 통째로 차지할 만큼 복잡하다고 합니다.)


**추가로,** 람다 자리에 비교자 생성 메서드를 사용하며 더 간결하게 만들 수 있습니다.

```java
// 비교자 생성 메서드 사용
Collections.sort(words, comapringInt(String::length));

// List엥 추가된 sort
words.sort(comparingInt(String::length));
```

📖참고 : `coparingInt`의 실제 코드
```java
public static <T> Comparator<T> comparingInt(ToIntFunction<? super T> keyExtractor) {
        Objects.requireNonNull(keyExtractor);
        return (Comparator<T> & Serializable)
            (c1, c2) -> Integer.compare(keyExtractor.applyAsInt(c1), keyExtractor.applyAsInt(c2));
    }
```

item 34 Operation의 변형
```java
public enum Operation {
    PLUS  ("+", (x, y) -> x + y),
    MINUS ("-", (x, y) -> x - y),
    TIMES ("*", (x, y) -> x * y),
    DIVIDE("/", (x, y) -> x / y);

    private final String symbol;
    private final DoubleBinaryOperator op;

    Operation(String symbol, DoubleBinaryOperator op) {
        this.symbol = symbol;
        this.op = op;
    }

    @Override public String toString() { return symbol; }

    public abstract double apply(double x, double y) {
        return op.applyAsDouble(x, y);
    }
}
```
이전에 item 34에서 상수별 클래스 몸체를 구현하는 방식보다는 열거타입에 인스턴스 필드를 두는 편이 낫다 했습니다.
람다를 이용하면 열거 타입의 인스턴스 필드를 이용하는 방식으로 상수별로 다르게 동작하는 코드를 쉽게 구현할 수 있습니다.

### 람다는 이름이 없고 문서화도 못 한다. 따라서 코드 자체로 동작이 명확히 설명되지 않거나 코드 줄 수가 많아지면 람다를 쓰지 말아야 한다.
- 람다는 한줄 일때 가장 좋다. 세 줄 안에 끝내기.
- 길거나 읽기 어렵다면 안쓰는 쪽으로 리팩터링 해보기.
- 열거 타입 생성자에 넘겨지는 인수들의 타입도 컴파일 타임에 추론된니다. 그런데 열거 타입의 인스턴스는 런타임에 만들어지지 때문에 주의가 필요합니다.
  - 상수별 동작을 단 몇 줄로 구현하기 어렵거나, 인스턴스 필드나 메서드를 사용해야만 하는 상황이라면 상수별 클래스 몸체를 사용해야 한다.

### 익명 클래스의 입지가 좁아짐. 하지만 람다로 대체할 수 없는 곳
- 추상 클래스의 인스턴스를 만들 때, 추상 메서드가 여러 개인 인터페이스의 인스턴스를 만들때, 익명클래스만 가능하다.
  - 람다는 함수형 인터페이스에서만 쓰인다.
- 함수 객체가 자신을 참조해야 한다면 반드시 익명클래스를 사용해야 한다.
  - 람다는 자기 자신을 참조할 수 없다.
  - 익명 클래스에서의 this는 익명 클래스의 인스턴스 자신을 가리킨다.

❓추상클래스에서는 람다식을 못쓰는건가?
- [스택 오버 플로 - Lambda Expressions for Abstract Classes](https://stackoverflow.com/questions/34424410/lambda-expressions-for-abstract-classes)
- 위 질문의 2번째 답변에 달린 링크 : [Allow lambdas to implement abstract classes](https://mail.openjdk.org/pipermail/lambda-dev/2013-March/008441.html)

### 람다를 직렬화하는 일은 극히 삼가야 한다.
- 람다도 익명 클래스처럼 직렬화 형태가 구현별로(가령 가상머신별로) 다를 수 있다. 따라서 직렬화를 삼가야 한다.(익명클래스의 인스턴스도 마찬가지다.)
- 직렬화해야만 하는 함수 객체가 있다면(ex.Comparator) private 정적 중첩 클래스의 인스턴스를 사용하자

## 정리
- 자바 8부터 람다가 도입
- 익명 클래스는 (함수형 인터페이스가 아닌) 타입의 인스턴스를 만들때만 사용하라.
- 람다는 작은 함수 객체를 아주 쉽게 표현할 수 있어 함수형 프로그래밍의 지평을 열었다.