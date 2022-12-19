# 아이템 50. 적시에 방어적 복사본을 만들라
자바는 안전한 언어이다. 네이티브 메서드를 사용하지 않으니 C, C++ 같이 안전하지 않은 언어에서 흔히 보는 버퍼 오버런, 배열 오버런, 와일드 포인터 같은 메모리 충돌 오류에서 안전하다.    


## 방어적 프로그래밍
자바라 해도 다른 클래스로부터의 침범을 아무런 노력 없이 다 막을 수 있는 것은 아니다.   
그러므로 클라이언트가 악의적으로 불변식을 깨드리려 한다고 가정하고 방어적으로 프로그래밍 하라.

### Period 클래스 예시
```java
public final class Period {
    private final Date start;
    private final Date end;

    public Period(Date start, Date end) {
        if (start.compareTo(end) > 0) {
            throw new IllegalArgumentException(start + "가 " + end + "보다 늦다.");
        }
        this.start = start;
        this.end = end;
    }

  public Date start(){
    return start;
  }

  public Date end(){
    return start;
  }

  public static void main(String[] args) {
    Date start = new Date();
    Date end = new Date();
    Period period = new Period(start, end);
    end.setYear(78); // period의 내부를 수정했다!
  }
}
```

- 자바 8 이후로는 쉽게 해결할 수 있다. Date 대신 불변인 Instant를 사용하면 된다(혹은 LocalDateTime이나 ZonedDateTime을 사용해도 된다)
- Date는 낡은 API이니 새로운 코드를 작성할 때는 **더 이상 사용하면 안 된다.**
- 외부 공격으로부터 Period 인스턴스의 내부를 보호하려면 생성자에서 받은 가변 매개변수 각각을 방어적으로 복사(defensive copy)해야 한다. 그 다음 Period 인스턴스 안에서는 원본이 아닌 복사본을 사용한다.

## 수정한 생성자
```java
public final class Period {
    private final Date start;
    private final Date end;

    public Period(Date start, Date end) {
        this.start = new Date(start.getTime());
        this.end = new Date(end.getTime());
        if (start.compareTo(end) > 0) {
            throw new IllegalArgumentException(start + "가 " + end + "보다 늦다.");
        }
    }

  public Date start(){
    return start;
  }

  public Date end(){
    return start;
  }

  public static void main(String[] args) {
    Date start = new Date();
    Date end = new Date();
    Period period = new Period(start, end);
    period.end().setYear(78); // period의 내부를 변경했다!
  }
}

```
- 매개변수의 유효성을 검사하기 전에 방어적 복사본을 만들고, 이 복사본으로 유효성을 검사했다. 
  - 순서가 부자연스러워 보일 수 있으나, 반드시 이렇게 작성해야 한다. 
  - 멀티스레딩 환경이라면 원본 객체의 유효성을 검사한 후 복사본을 만드는 그 찰나의 취약한 순간에 다른 스레드가 원본 객체를 수정할 위험이 있기 때문이다. 
  - 방어적 복사를 매개변수 유효성 검사 전에 수행하면 이런 위험에서 해방될 수 있다. 컴퓨터 보안 커뮤니티에서는 이를 검사시점/사용시점(time-of-check/time-of-use) 공격 혹은 TOCTOU 공격이라 한다.

| 쓰레드 1                           |악의를 가진 쓰레드 2|
|---------------------------------|---|
| start.after(end) = false(유효성 검사) ||
|                                 |end = 0 (위의 유효성 검사가 더 이상 유효하지 않다.)|
| 복사                              ||

방어적 복사에 clone 메소드를 사용하지 않는다.
- Date가 final 하지 않으므로 악의를 가진 하위 클래스를 반환할수도 있다.
- start, end 참조를 private static 리스트에 담아뒀다가 이 리스트에 접근하는 것을 공격자에게 허용할 수 있다.

```java
MaliciousDate someDate = new MaliciousDate();
Date copyOfMaliciousDate = someDate;
Date anotherDate = copyOfMaliciousDate.clone();
```


### 방어적 복사본 접근자
```java
public final class Period {

    public Date start() {
        return new Date(start.getTime());
    }

    public Date end() {
        return new Date(end.getTime());
    }
}
```
- 길이가 1 이상인 배열은 무조건 가변이다. 따라서 내부에서 사용하는 배열을 클라이언트에 반환할 때는 항상 방어적 복사를 수행해야 한다.


## 방어적 복사의 생략
호출자가 컴포넌트 내부를 수정하지 않으리라 확신하면 방어적 복사를 생략해도 된다. (예를 들어 같은 패키지의 경우)이런 경우에도 문서화하는 것이 좋다.

## Reference
https://wjdtn7823.tistory.com/90
