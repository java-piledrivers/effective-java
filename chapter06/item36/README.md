# 아이템36. 비트 필드 대신 EnumSet을 사용하라

## 비트 필드

열거한 값들이 집합으로 사용될 경우, 예전에는 각 상수에 서로 다른 2의 거듭제곱 값을 할당한 정수 열거 패턴을 사용했다.
비트 필드(bit field)란, 이러한 상수들에 비트별 OR 연산을 하여 만들어진 집합을 말한다.

다음은 비트 필드 열거 상수를 사용한 예제이다.

```java
public class Text {

  public static final int STYLE_BOLD = 1 << 0; // 1 (0000 0001)
  public static final int STYLE_ITALIC = 1 << 1; // 2 (0000 0010)
  public static final int STYLE_UNDERLINE = 1 << 2; // 4 (0000 0100)
  public static final int STYLE_STRIKETHROUGH = 1 << 3; // 8 (0000 1000)

  // 매개변수 styles는 0개 이상의 STYLE_ 상수를 비트별 OR한 값이다.
  // ex) text.applyStyles(STYLE_BOLD | STYLE_ITALIC);
  public void applyStyles(int styles) {...}
}
```

### 문제점

비트 필드는 정수 열거 상수의 단점을 그대로 가지며, 추가로 다음과 같은 문제를 가지고 있다.

1. 비트 필드 값이 그대로 출력되면, 해석하기가 어렵다. (단순한 정수 열거 상수를 출력할 때보다 더 어렵다)
2. 비트 필드 하나에 녹아 있는 모든 원소를 순회하기 까다롭다.
3. 최대 몇 비트가 필요한지 API 작성 시 미리 예측하여 적절한 타입(보통은 int나 long)을 선택해야 한다. API를 수정하지 않고는 비트 수(32 bit or 64
   bit)를 더 늘릴 수 없기 때문이다.

## EnumSet 클래스

비트 필드의 대안은 java.util 패키지의 EnumSet 클래스를 사용하는 것이다.   
EnumSet 클래스는 열거 타입 상수의 값으로 구성된 집합을 효과적으로 표현해준다.
Set 인터페이스를 완벽히 구현하며, 타입 안전하고, 다른 모든 Set 구현체와 함께 사용할 수 있다.

```java
public class Text {

  public enum Style {BOLD, ITALIC, UNDERLINE, STRIKETHROUGH}

  // 어떤 Set을 넘겨도 되나, EnumSet이 가장 좋다.
  public void applyStyles(Set<Style> styles) { ...}
}
```

### applyStyles의 메서드가 EnumSet\<Style>이 아닌 Set\<Style>을 받은 이유

모든 클라이언트가 EnumSet을 건네리라 짐작되는 상황이라도 이왕이면 인터페이스로 받는 게 일반적으로 좋은 습관이다(아이템 64).   
이렇게 하면 좀 특이한 클라이언트가 다른 Set 구현체를 넘기더라도 처리할 수 있으니 말이다.

### 장점

1. 열거 타입의 장점을 가진다. (아이템 34)
2. 내부적으로는 비트 벡터로 구현되어 있어, 비트 필드에 비견되는 성능을 보여준다. (enum 갯수가 64 이하인 경우 EnumSet은 long 값 하나만 사용)
3. removeAll과 retainAll 같은 대량 작업은 비트를 효율적으로 처리할 수 있는(비트 필드를 사용할 때 쓰는 것과 같은) 산술 연산을 써서 구현했다.
4. 비트를 직접 다룰 때 흔히 겪는 오류들에서 해방된다. 

--- 
#### removeAll, retainAll

```java
public class RemoveAllTest {

  public static void main(String[] args) {
    Set<Integer> set1 = new HashSet<>();

    set1.add(1);
    set1.add(2);
    set1.add(3);
    set1.add(4);
    set1.add(5);

    Set<Integer> set2 = new HashSet<>();
    set2.add(1);
    set2.add(2);
    set2.add(3);

    set1.removeAll(set2);

    System.out.println("removeAll() operation : "
        + set1); // [4, 5]
  }
}


public class RetainAllTest {

   public static void main(String[] args) {
      Set<Integer> set1 = new HashSet<>();

      set1.add(1);
      set1.add(2);
      set1.add(3);
      set1.add(4);
      set1.add(5);

      Set<Integer> set2 = new HashSet<>();
      set2.add(1);
      set2.add(2);
      set2.add(3);

      set1.removeAll(set2);

      System.out.println("removeAll() operation : "
              + set1); // [1, 2, 3]
   }
}

```

#### EnumSet의 주의할 점
https://yeti.tistory.com/287
