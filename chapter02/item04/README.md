# [아이템4] 인스턴스화를 막으려거든 private 생성자를 사용하라


### 정적 메서드만 담은 유틸리티 클래스는 인스턴스로 만들어 쓰려고 설계한 클래스가 아니다.
보통 유틸리티 클래스를 사용할 경우, 인스턴스화를 하지 않는다.   
static 메서드를 사용할 경우, 인스턴스화를 사용하게 되면 오히려 이 메서드가 static 메서드인지, 인스턴스 메서드인지 판별하기 어렵기 때문에 좋지 않은 코드이다.   
그래서 이 경우 private 생성자를 이용하여 인스턴스화를 막는다.
### 추상 클래스로 만드는 것으로는 인스턴스화를 막을 수 없다.
```java
public abstract class UtilityClass {
  public UtilityClass() {
    
  }
}
```
위와 같이 abstract를 사용할 경우 UtilityClass의 생성자를 사용할 순 없지만, 저 클래스를 상속받은 자식 클래스에서 인스턴스화 시킬 수 있다.   
오히려 abstract 키워드 때문에 이 클래스가 상속해서 사용하는 클래스라고 착각하여 코드 해석에 어려움을 준다. 
### private 생성자를 추가하면 클래스의 인스턴스화를 막을 수 있다.
### 생성자에 주석으로 인스터스화 불가한 이유를 설명하는 것이 좋다.
```java
public class UtilityClass {

  // 해당 생성자가 왜 private 인지(막아뒀는지) 이유를 설명
  /**
   * 이 클래스는 인스턴스를 만들 수 없습니다.
   */
  private UtilityClass() {
    // AssertionError()는 try-catch로 처리하라는 뜻이 아니라 해당 상황이 발생되면 안된다는 것을 나타낸다.
    throw new AssertionError();
  }
}
```
### 상속을 방지할 때도 같은 방법을 사용할 수 있다.


### 사실 스프링에도 private 처리대신 abstract로 인스턴스화 를 막는 Util 클래스들이 많다.
ActiveProfilesUtils,AnnotationConfigUtils 등 private으로 생성자를 정의하지 않고, abstract로 만들어서 인스터스화를 막는다.    
하지만 두 번째(추상 클래스로 만드는 것으로는 인스턴스화를 막을 수 없다.)와 같은 이유로 해당 방법은 좋지 못하다.
