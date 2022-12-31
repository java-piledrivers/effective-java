# 53. 가변인수는 신중히 사용하라

---

가변인수(vargs) 메서드는 `명시한 타입의 인수를 0개 이상` 받을 수 있다. 가변인수 메서드를 호출하면, 가장 먼저 인수의 개수와 길이가 같은 배열을 만들고 인수들을 이 배열에 저장하여 가변인수 메서드에 건네준다.

### 간단한 가변인수 활용 예

```java
static int sum(int... args){
  int sum = 0;
  for (int arg : args){
    sum += arg;
  }
  return sum;
}
```

### 인수가 1개 이상이어야 하는 가변인수 메서드 - 잘못 구현한 예!

```java
static int min(int... args){
  if (args.length == 0)
    throw new IllegalArgumentException("인수가 1개 이상 필요");
  int min = args[0];
  for (int i = 1; i < args.length; i++){
    if (args[i] < min){
      min = args[i];
    }
  }
  return min;
}
```

위 코드의 문제점

1. 인수를 0개만 넣어 호출한 경우 컴파일 타임이 아닌 런타임에 실패
2. 코드 지저분 (유효성 검사 명시적으로 해야하며 더 명료한 for-each문도 사용 불가)

### 인수가 1개 이상이어야 할 때 가변인수를 제대로 사용하는 방법

```java
static int min(int firstArg, int... remainingArgs) {
    int min = firstArg;
    for (int arg : remainingArgs)
        if (arg < min)
            min = arg;
    return min;
}
```

---

## 성능에 민감한 상황이라면 가변인수 사용을 고려하자

가변인수 메서드는 호출될 때 마다 배열을 새로 하나 할당하고 초기화된다. 이 비용을 감당할 수는 없지만 가변인수의 유연성이 필요할 때 선택할 수 있는 멋진 패턴이 있다.

만약 해당 메서드 호출의 95%가 인수를 3개 이하로 사용한다고 가정해보자.

```java
public void foo() { }
public void foo(int a1) { }
public void foo(int a1, int a2) { }
public void foo(int a1, int a2, int a3) { } -> 여기까지 95%의 호출 담당

public void foo(int a1, int a2, int a3, int... rest) { } -> 5%의 호출을 담당
```

EnumSet의 정적 팩터리도 이 기법을 사용해 열거 타입 집합 생성 비용을 최소화한다.

EnumSet은 비트 필드(아이템 36)을 대체하면서 성능까지 유지해야 하므로 아주 적절하게 활용한 예

```java
import java.util.*;

// 코드 36-2 EnumSet - 비트 필드를 대체하는 현대적 기법 (224쪽)
public class Text {
    public enum Style {BOLD, ITALIC, UNDERLINE, STRIKETHROUGH}

    // 어떤 Set을 넘겨도 되나, EnumSet이 가장 좋다.
    public void applyStyles(Set<Style> styles) {...}
}
```

---

## 정리

- 인수 개수가 일정하지 않은 메서드를 정의해야 한다면 가변인수가 반드시 필요하다. 메서드를 정의할 때 필수 매개변수는 가변인수 앞에 두고, 가변인수를 사용할 때는 성능 문제까지 고려하자.
