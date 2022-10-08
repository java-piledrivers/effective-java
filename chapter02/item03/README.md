# [아이템3] private 생성자나 열거 타입으로 싱글턴임을 보증하라


### 싱글턴 인스턴스의 테스트
클래스를 싱글턴으로 만들면 이를 사용하는 클라이언트를 테스트하기가 어려워질 수 있다. 
타입을 인터페이스로 정의한 다음 인터페이스를 구현해서 만든 싱글턴이 아니면 싱글턴 인스턴스를 mock구현으로 대체할 수 없기 때문이다.

### 싱글턴 만드는 방식
생성자는 private으로 감춰두고 인스턴스에 접근하는 수단 2가지를 제공할 수 있다.
1. public static 멤버 제공

    ```java
    public class Elvis {
        pulbic static final Elvis INSTANCE = new Elvis();
        private Elivs() {...}
    }
    ```
    #### 예외
    private 생성자는  static 필드를 초기화할 때 딱 한 번만 호출된다. 
    중요한 점은 **예외**가 있는데 `권한이 있는 클라이언트(?)`는 리플렉션 API인 **AccessibleObject.setAccessible**을 사용해 private 생성자를 호출할 수 있다. 
    리플렉션 API를 통한 private생성자 호출을 막으려면, 두번째 객체가 생성될 때 예외를 던지도록 생성자에 코드를 작성해야 한다.

    #### 장점
    해당 클래스가 싱글턴인것이 명백하게 드러난다. public static 필드가 final이므로 다른 객체를 참조할 수 없기 때문이다.


2. 정적 팩터리 메서드를 public static 멤버로 제공
    ```java
    public class Elvis {
        private static final Elvis INSTANCE = new Elvis();
        private Elivs() {...}

        public static Elivs getInstance() { return INSTANCE;}
    }
    ```

    #### 장점
    - 설계를 바꾸고자하면 API를 변경하지 않고 싱글턴이 아니게 메서드를 수정할 수 있다.
    - [ ] 정적 팩터리를 `제네릭 싱글턴 팩터리`로 만들 수 있다.
    - [ ] 정적 팩터리 메서드 참조를 `supplier`로 사용할 수 있다.
     - 예) Elivs::getInstance()를  Supplier<Elvis> 로 사용

    이런 장점이 필요하지 않으면 public 필드 방식이 좋다.

---

### 싱글턴 클래스의 직렬화
위 두가지 방식으로 만든 싱글턴 클래스를 직렬화하려면 Serializable을 구현하는 것만으로는 부족하고 아래 두가지 조건을 제공해야 한다.

- 모든 필드를 **transient** 로 선언
- readResolve() 메서드 제공 
  
    ```java
    //싱글턴임을 보장해주는 메서드
    private Object readResolve() {
        return INSTANCE;
    }

    ```

그래야 직렬화된 인스턴스를 역직렬화할 때마다 새로운 인스턴스가 생성되는 것을 막을 수 있다.

---

### 싱글턴을 만드는 또 다른 방법
원소가 하나인 열거 타임을 선언하는 것이다. 

```java
public enum Elvis {
    INSTANCE;
    
}
```

#### 장점
- public 필드 방식과 비슷하지만 더 간결하다
- 추가 조건없이 직렬화할 수 있다.
- 복잡한 직렬화 상황이나 리플렉션 공격에도 다른 인스턴스가 생기는 일을 완벽히 막아준다.

대부분의 상황에서 원소가 하나인 enum타입이 싱글턴을 만드는 가장 좋은 방법이다. 
그러나 열거 타입은  인터페이스를 구현할 수는 있으나, 만드려는 싱글턴이 다른 클래스를 상속해야 한다면 이 방법은 사용할 수 없다.

---

## 추가 키워드 
- 리플렉션 API인 **AccessibleObject.setAccessible**
- 리플렉션 `권한이 있는 클라이언트(?)`
- 직렬화, readResolve() 메서드
