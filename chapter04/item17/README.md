불변 객체는 근본적으로 스레드 안전하여 따로 동기화할필요 없이, 안심하고 공유할 수 있다.

클래스는 꼭 필요한 경우가 아니라면 불변이여야한다.
1.모든 클래스를 불변으로 만들 수는 없어도 변경할 수 있는 부분을 최소한으로 줄여야한다.
2.그렇기에 합당한 이유가 없다면 필드는 기본적으로 private final 이어야한다.
3.메서드의 경우 상속을 통해 메서드의 값을 변경할 수 있는 제어자(public)을 함부로 제공하면 안된다.

클래스가 불변임을 보장하려면 자신을 상속하지 못하게 해야한다. 
BigInteger의 경우 메서드들의 접근제한자가 public이면서 정적이지 않아 패키지 외부에서 재정의가 가능하다. 

BigInteger 클래스의 divide 메서드 (오라클 JDK8 버전 기준)
```java
    public BigInteger divide(BigInteger val) {
        if (val.mag.length < BURNIKEL_ZIEGLER_THRESHOLD ||
                mag.length - val.mag.length < BURNIKEL_ZIEGLER_OFFSET) {
            return divideKnuth(val);
        } else {
            return divideBurnikelZiegler(val);
        }
    }
```

! 
불변 객체를 만들기 위해 무조건 fianl를 사용해야하는 것이아니라 유연한 방법으로 패키지 바깥에서 클라이언트에서 바라볼때 사실상 fianl이면 불변객체여도 된다.

유연한 방법 예시

패키지 바깥에서 호출가능한 생성자가 없기에 상속을 통한 클래스 확장이 불가능하다.
``` java 
public class Complex {
    private final double re;
    private final double im;

    private Complex(double re, double im) {
        this.re =re;
        this.in = in;
    }

    //아이템 1에 나온 생성자 대신 정적 팩터리를 통한 객체 생성 방식
    public static Complex valueOf(double re, double im) {
        return new Complex(re, im);
    }
}
```




불변 클래스의 단점 : 값이 다르면 반드시 독립된 새로운 객체를 생성해야하기에 특정상황에서 잠재적으로 성능 저하를 일으킬 수 있다.
ex) 비트 하나를 바꾸기 위해 백만비트짜리 객체를 새로만들어야하는 경우

불변 클래스의 단점은 다단계 연산(multistep operation)을 통해 해결가능



