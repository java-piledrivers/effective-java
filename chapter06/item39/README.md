# 39. 명명 패턴보다 애너테이션을 사용하라

전통적으로 도구나 프레임워크가 특별히 다뤄야 할 프로그램 요소에는 딱 구분되는 명명 패턴을 적용해왔다.

---

### 명명패턴의 단점

- 오타가 나면 안된다.
- 올바른 프로그램 요소에서만 사용 되리라 보증 할 방법이 없다.
- 프로그램 요소를 매개변수로 전달할 마땅한 방법이 없다.

명명 패턴을 사용 하던 JUnit 3은 위 언급한 여러 문제들을 갖고 있었고, 이런 문제들을 해결해주는 멋진 개념으로, JUnit 4부터는 애너테이션이 전면 도입되었다.

### 1. 마커(maker) 애너테이션 타입 선언

```java
/**
 * 테스트 메서드임을 선언하는 애너테이션이다.
 * 매개변수 없는 정적 메서드 전용이다. -> 이 부분 조금 헷갈림. 왜 정적 메서드?
 */
@Retention(RetentionPolicy.RUNTIME) // @Test가 런타임에도 유지되어야 한다는 표시
@Target(ElementType.METHOD) // @Test가 반드시 메서드 선언에서만 사용된다는 표시
public @interface Test {
}
```

- @Retention과 @Target과 같이 애너테이션 선언에 다는 애너테이션을 메타애너테이션(meta-annotation)이라 한다.

- 이와 같은 애너테이션을 "아무 매개변수 없이 단순히 대상에 마킹(marking)한다"는 뜻에서 마커(marker) 애너테이션이라 한다.

@Target

```xml
ElementType.PACKAGE : 패키지 선언
ElementType.TYPE : 타입 선언(클래스, 인터페이스, enum 등)
ElementType.ANNOTATION_TYPE : 어노테이션 타입 선언
ElementType.CONSTRUCTOR : 생성자 선언
ElementType.FIELD : 멤버 변수 선언
ElementType.LOCAL_VARIABLE : 지역 변수 선언
ElementType.METHOD : 메서드 선언
ElementType.PARAMETER : 전달인자 선언
ElementType.TYPE_PARAMETER : 전달인자 타입 선언
ElementType.TYPE_USE : 타입 선언
```

@Retention

```xml
RetentionPolicy.RUNTIME : 컴파일 이후에도 JVM 에 의해서 계속 참조가 가능
RetentionPolicy.CLASS : 컴파일러가 클래스를 참조할 때까지 유효
RetentionPolicy.SOURCE : 컴파일 전까지만 유효합니다. 즉, 컴파일 이후에는 사라지게 됩니다.
```

### 2. 마커 애너테이션을 사용한 프로그램

```java
public class Sample {

    @Test
    public static void m1() { // 성공해야 한다.
    }

    public static void m2() {
    }

    @Test
    public static void m3() { // 실패해야 한다.
        throw new RuntimeException("실패");
    }

    public static void m4() {
    }

    @Test
    public void m5() { // 잘못 사용한 예: 정적 메서드가 아니다.
    }

    public static void m6() {
    }

    @Test
    public static void m7() { // 실패해야 한다.
        throw new RuntimeException("실패");
    }

    public static void m8() {
    }
}
```

### 3. 마커 애너테이션을 처리하는 프로그램

```java
public class RunTests {
    public static void main(String[] args) throws ClassNotFoundException, InvocationTargetException, IllegalAccessException {
        int tests = 0;
        int passed = 0;
        Class<?> testClass = Class.forName(args[0]);
        for (Method m : testClass.getDeclaredMethods()) {
            if (m.isAnnotationPresent(Test.class)) {
                tests++;
                try {
                    m.invoke(null);
                    passed++;
                } catch (InvocationTargetException wrappedExc) {
                    Throwable exc = wrappedExc.getCause();
                    System.out.println(m + "실패: " + exc);
                } catch (Exception exception) {
                    System.out.println("잘못 사용한 @Test: " + m);
                }
            }
        }
        System.out.printf("성공: %d, 실패: %d%n", passed, tests - passed);
    }
}
```

---

### 4. 매개변수 하나를 받는 애너테이션 타입

```java
/**
 * 명시한 예외를 던져야만 성공하는 테스트 메서드용 애너테이션
 */
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTest {
    Class<? extends Throwable> value();
}
```

### 5. 매개변수 하나짜리 마커 애너테이션을 사용한 프로그램

```java
public class Sample2 {
    @ExceptionTest(ArithmeticException.class)
    public static void m1() {
        int i = 0;
        i = i / i; //divide by zero. ArithmeticException 예외를 발생시킴 -> 성공
    }
    @ExceptionTest(ArithmeticException.class)
    public static void m2() {
        int[] a = new int[0];
        int i = a[1]; //IndexOutOfBoundsException 발생 -> ArithmeticException가 아니므로 실패
    }
    @ExceptionTest(ArithmeticException.class)
    public static void m3() {} // 아무 Exception도 발생하지 않음 -> 실패
}
```

---

### 6. 배열 매개변수를 받는 애너테이션 타입

```java
/**
* 명시한 예외를 던져야만, 성공하는 테스트케이스 애너테이션
*/
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTest{
    //한정적 와일드카드를 통해 Throwable을 상속한 모든 타입을 지정
    Class<? extends Throwable>[] values(); 
}
```

### 7. **배열 매개변수를 받는 애너테이션을 사용하는 코드**

```java
public class RunTests {
    @ExceptionTest({IndexOutOfBoundsException.class, NullPointerException.class})
    public static void doublyBad() { // 성공해야 한다.
        List<String> list = new ArrayList<>();

        // 자바 API 명세에 따르면 다음 메서드는 IndexOutOfBoundsException이나 NullPointerException을 던질 수 있다.
        list.addAll(5, null);
    }
}
```

---

### 8. 반복 가능한 애너테이션 타입

- 자바 8에서는 여러 개의 값을 받는 애너테이션을 다른 방식으로도 만들 수 있다.
- 배열 매개변수를 사용하는 대신 애너테이션에 @Repeatable 메타애너테이션을 다는 방식이다.
- @Repeatable을 단 애너테이션은 하나의 프로그램 요소에 여러 번 달 수 있다.

```java
// 반복 가능한 애너테이션
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
@Repeatable(ExceptionTestContainer.class)
public @interface ExceptionTest {
    Class<? extends Throwable> value();
}

// 컨테이너 애너테이션
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTestContainer {
    ExceptionTest[] value();
}
```

**주의사항**

- 첫 번째, @Repeatable을 단 애너테이션을 반환하는 '컨테이너 애너테이션'을 하나 더 정의하고, @Repeatable에 이 컨테이너 애너테이션의 class 객체를 매개변수로 전달해야 한다.
- 두 번째, 컨테이너 애너테이션은 내부 애너테이션 타입의 배열을 반환하는 value 메서드를 정의해야 한다.
- 마지막으로 컨테이너 애너테이션 타입에는 적절한 보존 정책(@Retention)과 적용 대상(@Target)을 명시해야 한다. 그렇지 않으면 컴파일되지 않을 것이다.

### 9. 반복 가능한 애너테이션을 사용하는 코드

```java
@ExceptionTest(IndexOutOfBoundsException.class)
@ExceptionTest(NullPointerException.class)
public static void doublyBad() {...}
```

---

## 정리

- **애너테이션으로 할 수 있는 일을 명명 패턴으로 처리할 이유는 없다.**
- **자바 프로그래머라면 예외 없이 자바가 제공하는 애너테이션 타입들은 사용해야 한다.**
