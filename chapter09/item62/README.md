# 아이템 65. 다른 타입이 적절하다면 문자열 사용을 피하라.
문자열은 다른 값 타입을 대신하기에 적합하지 않다.

- 문자열은 열거 타입을 대신하기에 적합하지 않다.
- 문자열은 혼합 타입을 대신하기에 적합하지 않다.
- 문자열은 권한을 표현하기에 적합하지 않다.

## 문자열은 열거 타입을 대신하기에 적합하지 않다.
 - 열거 타입은 기본적으로 상수보다 가독성과 안전성 부분에서 뛰어나다.
 - 상수 하나당 자신의 인스턴스를 하나씩 만들어 public static final 필드로 공개한다(인스턴스 하나임을 보장)
 - 정의한 상수를 참조하고 있음을 확신할 수 있기에, 컴파일 타임 안전성을 제공한다

## 문자열은 혼합 타입을 대신하기에 적합하지 않다.

```java
String compoundKey = className + "#" + i.next();
```
 - 각 요소를 개별로 접근하라면 문자열을 파싱해야하기에 성능상 느리고 오류 가능성이 커진다.
 - 적절한 equals, toString, compareTo 메서드를 제공할 수 없고, String이 제공하는 기능에만 의존해야 한다.

 - 문제점을 해결하는 방법: 아이템24

## 문자열은 권한을 표현하기에 적합하지 않다.

[권한을 위한 키를 문자열로 구현하는 방법]
* 각 스레드가 자신만의 변수를 가지도록 하는 기능을 구현하는 상황이다.

```java
public class ThreadLocal {
    private ThreadLocal() { }

   // 현 스레드의 값을 키로 구분해 저장한다.
   // 각 스레드가 고유한 키를 가지고 있도록 설계된다면, value는 각 스레드의 고유한 변수가 된다.
   public static void set(String key, Object value);
 
  // (키가 가리키는) 현 스레드의 값을 반환한다.
  public static Object get(String key);
}
```

 - 이 방식의 문제는 스레드 구분용 문자열 키가 전역 이름공간에서 공유된다.
 
! 클라이언트들에게 실수로 같은 키가 제공된다면 해당 기능이 제공되지 않는다.

! 악의적인 사용자에 의해 키가 위조될 위험이 있다.

[개선한 방법]
```java
public class ThreadLocal {
    private ThreadLocal() { } // 객체 생성 불가
   
    public static class Key { // (권한)
        Key() { } //키 생성자
    }

    // 위조 불가능한 고유 키를 생성한다.
    public static Key getKey() {
        return new Key();
    }
   
    public static void set(Key key, Object value);
    public static Object get(Key key);
}
```
 - Key 클래스의 인스턴스를 고유한 키로 사용하여 키 변조 위험과 중복 가능성을 제거한 방식이다.
 
