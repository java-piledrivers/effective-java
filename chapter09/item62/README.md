# 아이템 65. 다른 타입이 적절하다면 문자열 사용을 피하라.
문자열은 다른 값 타입을 대신하기에 적합하지 않다.

- 문자열은 열거 타입을 대신하기에 적합하지 않다.
- 문자열은 혼합 타입을 대신하기에 적합하지 않다.
- 문자열은 권한을 표현하기에 적합하지 않다.

## 문자열은 열거 타입을 대신하기에 적합하지 않다.
 - 아이템34 참조
 - 수정중

## 문자열은 혼합 타입을 대신하기에 적합하지 않다.

```java
String compoundKey = className + "#" + i.next();
```
 - 각 요소를 개별로 접근하라면 문자열을 파싱해야하기에 성능상 느리고 오류 가능성이 커진다.
 - 적절한 equals, toString, compareTo 메서드를 제공할 수 없고, String이 제공하는 기능에만 의존해야 한다.

 - 문제점을 해결하는 방법: 아이템24

## 문자열은 권한을 표현하기에 적합하지 않다.

```java
public class ThreadLocal {
    private ThreadLocal() { }

   // 현 스레드의 값을 키로 구분해 저장한다.
   public static void set(String key, Object value);
 
  // (키가 가리키는) 현 스레드의 값을 반환한다.
  public static Object get(String key);
}
```

 - 이 방식의 문제는 스레드 구분용 문자열 키가 전역 이름공간에서 공유된다.
! 각 클라이언트가 실수로 같은 키를 쓰게된다면 모두 제대로 동작하지 않는다.
! 악의적인 사용자에 의해 키가 위조될 위험이 있다.

[개선한 방법]
```java
public class ThreadLocal {
    private ThreadLocal() { } // 객체 생성 불가
   
    public static class Key { // (권한)
        Key() {  } //키 생성 코드
    }

    // 위조 불가능한 고유 키를 생성한다.
    public static Key getKey() {
        return new Key();
    }
   
    public static void set(Key key, Object value);
    public static Object get(Key key);
}
```

 - 키 변조 위험이 없어진다.
