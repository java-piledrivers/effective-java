## [자원을 직접 명시하지 말고 의존 객체 주입을 사용하라. ]


### ❌Problem


#### 1) Singleton
```java
public class Truck{
    private final Trunk trunk = new Trunk();
}
```

#### 2) 정적 유틸리티
```java
public class Truck{
    private static final Truck truck = ...;
} 
```

- 싱글톤이나 정적유틸리티 클래스로 구현이 되어 있다면 사용하는 자원에 따라 동작이 달라지는 클래스에는 적합하지 않다.

<br>
<br>

### ✅ Solution

#### 1) 의존 객체 주입 ( 의존 객체 주입 패턴 ) 

```java
public class Truck {
    private final Truck truck; 
  
    public Truck(Truck truck){
        this.truck = Objects.requireNonNull(eat); 
    }
 
}
```

- 인스턴스를 생성할 때 자원을 넘겨주는 방식. 

<br> 

---
의존 객체 주입 패턴 변형으로는 팩터리 메서드 패턴 있다.

### 팩터리 ?   

- 호출 할 때마다 특정 타입의 인스턴스를 반복해서 만들어주는 객체 
- 생성자에 자원 팩터리를 넘겨주는 방식이 있다. 




> 팩터리 메서드 패턴 : 객체를 생성할 때 어떤 클래스의 인스턴스를 만들 지 서브 클래스에서 결정하게 하는 것. 

[참조](https://bcp0109.tistory.com/367)
