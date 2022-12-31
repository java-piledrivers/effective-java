# 60. 정확한 답이 필요하다면 float와 double은 피하라

---

float와 double 타입은 넓은 범위의 수를 빠르게 정밀한 '근사치'로 계산하도록 설계되었기 때문에, 정확한 결과가 필요할 때는 사용하면 안 된다.

특히 금융 관련 계산과는 맞지 않는다. 0.1 혹은 10의 음의 거듭제곱 수를 표현할 수 없기 때문이다.

### 어설프게 작성한 코드

```java
// 1.03달러가 있었는데 그중 42센트를 사용할 경우 얼마가 남았을까?
System.out.println(1.03 - 0.42);
// result : 0.610000000000000001

// 1달러로 10센트짜리 사탕 9개를 살 경우에 얼마가 남았을까?
System.out.println(1.00 - 9 * 0.10);
// result : 0.0999999999999998
```

### 오류 발생! 금융 계산에 부동소수 타입을 사용했다.

1달러를 가지고 10센트,20센트,30센트…1달러 짜리 사탕을 몇개 살 수 있을까

```java
public static void main(String[] args) {
    double funds = 1.00;
    int itemsBought = 0;
    for (double price = 0.10; funds >= price; price += 0.10) {
        funds -= price;
        itemsBought++;
    }
    System.out.println(itemsBought + "개 구입");
    System.out.println("잔돈(달러): " + funds);
}

// 3개 구입
// 잔돈(달러): 0.3999999999999999
```

문제를 올바로 해결하기 위해선?

`금융계산에는 BigDecimal, int 혹은 long을 사용해야 한다.`

### BigDecimal을 사용한 해법. 속도가 느리고 쓰기 불편하다.

```java
public static void main(String[] args) {
    final BigDecimal TEN_CENTS = new BigDecimal(".10");

    int itemsBought = 0;
    BigDecimal funds = new BigDecimal("1.00");
    for (BigDecimal price = TEN_CENTS; funds.compareTo(price) >= 0; 
						price = price.add(TEN_CENTS)) {
        funds = funds.subtract(price);
        itemsBought++;
    }
    System.out.println(itemsBought + "개 구입");
    System.out.println("잔돈(달러): " + funds);
}

// 4개 구입
// 잔돈(달러): 0.00
```

단점

1. 기본 타입보다 쓰기가 훨씬 불편하다.
2. 기본타입보다 훨씬 느리다.

### **BigDecimal의 대안으로 int 혹은 long 타입을 쓸 수도 있다.**

int와 long을 사용하여 성능을 향상시킬 순 있지만 다룰 수 있는 값의 크기가 제한되고, 소수점을 직접 관리해야 하는 단점이 있다. 다음은 모든 계산을 달러 대신 센트로 수행한 코드이다.

```java
public static void main(String[] args) {
    int itemsBought = 0;
    int funds = 100;
    for (int price = 10; funds >= price; price += 10) {
        funds -= price;
        itemsBought++;
    }
    System.out.println(itemsBought + "개 구입");
    System.out.println("잔돈(센트): " + funds);
}

// 4개 구입
// 잔돈(센트): 0
```

---

## 정리

정확한 답이 필요한 계산에는 float나 double을 피하라. 소수점 추정은 시스템에 맡기고, 코딩 시의 불편함이나 성능 저하를 신경 쓰지 않겠다면 BigDecimal을 사용하라. BigDecimal이 제공하는 여덞 가지 반올림 모드를 이용하여 반올림을 완벽히 제어할 수 있다. 법으로 정해진 반올림을 수행해야 하는 비즈니스 계산에서 아주 편리한 기능이다. 반면, 성능이 중요하고 소수점을 직접 추적할 수 있고 숫자가 너무 크지 않다면 int나 long을 사용하라. 숫자를 아홉자리 십진수로 표현할 수 있다면 int를 사용하고, 열여덞 자리 십진수로 표현할 수 있다면 long을 사용하라. 열여덞 자리를 넘어가면 BigDeciamal을 사용해야 한다.

참고 )

[https://velog.io/@syleemk/CS-부동-소수점-오차](https://velog.io/@syleemk/CS-%EB%B6%80%EB%8F%99-%EC%86%8C%EC%88%98%EC%A0%90-%EC%98%A4%EC%B0%A8)
