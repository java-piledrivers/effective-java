```java
class Point {
    public double x;
    public double y;
}
```
- 데이터 필드에 직접 접근할 수 있으니 캡슐화의 이점을 제공하지 못한다.
- API 를 수정하지 않고는 내부 표현을 바꿀 수 없다.
- 불변식을 보장할 수 없다.
- 외부에서 필드에 접근할 때 부수 작업을 수행할 수도 없다.
 
```java
class Point {
    private double x;
    private double y;

    public Point(double x, double y) {
        this.x = x;
        this.y = y;
    }

    // getter, setter
}
```
- 접근자와 변경자 메소드를 활용해 데이터를 캡슐화한다.
 

```java
public final class Time {
    private static final int HOURS_PER_DAY = 24;
    private static final int MINUTES_PER_HOUR = 60;

    public final int hour;
    public final int minute;

    public Time(int hour, int minute) {
        if (hour < 0 || hour >= HOURS_PER_DAY) {
            throw new IllegalArgumentException("시간: " + hour);
        }
        if (minute < 0 || minute >= HOURS_PER_HOUR) {
            throw new IllegalArgumentException("분: " + minute);
        }        
        this.hour = hour;
        this.minite = minute;    
    }

    // ...
}
```
- public 클래스의 필드가 불변이라면 직접 노출할때의 단점이 조금은 줄어들지만 전혀 좋은 생각이 아니다.
- API를 변경하지 않고는 표현 방식을 바꿀수 없다.
- 필드를 읽을 때 부수 작업을 수행할 수 없다.
- 단, 불변은 보장할 수 있게 된다.
- 하지만 package-private 클래스 혹은 private 중첩 클래스라면 데이터 필드를 노출한다 해도 하등의 문제가 없다.
 

결론
- public 클래스는 절대 가변 필드를 직접 노출해서는 안된다.
- 불변 필드라면 덜 위험하지만 완전히 안심할 수는 없다.
- 하지만, package-private, private 중첩클래스에서는 종종 필드를 노출하는 편이 나을 때도 있다.
