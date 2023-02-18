# 아이템 89. 인스턴스 수를 통제해야 한다면 readResolve보다는 열거 타입을 사용하라

아이템 3에서 보았던 싱글톤 패턴 클래스의 생성자는 오로지 하나의 인스턴스만 생성하도록 보장한다.
```java
public class Elvis {
  public static final Elvis INSTANCE = new Elvis();
  private Elvis() { ... }
  public void leavingTheBuilding() {...}
}
```


```java
class Elvis implements Serializable {
  public static final Elvis INSTANCE = new Elvis();
  private Elvis() {}
   public static Elvis getInstance() {
     return INSTANCE;
   }
  public void leavingTheBuilding() {
    System.out.println("I'm leaving the building");
  }

//  @Serial // https://stackoverflow.com/a/63783607
//  private Object readResolve() {
//    return INSTANCE;
//  }
}

public class Test6 {

  public static void main(String[] args) throws IOException, ClassNotFoundException {
        Elvis elvis = Elvis.getInstance();

        ByteArrayOutputStream bos = new ByteArrayOutputStream();
        ObjectOutputStream out = new ObjectOutputStream(bos);
        out.writeObject(elvis);

        ObjectInputStream ois = new ObjectInputStream(new ByteArrayInputStream(bos.toByteArray()));
        Elvis deserializedElvis = (Elvis) ois.readObject();

        System.out.println("elvis = " + elvis);
        System.out.println("deserializedElvis = " + deserializedElvis);
        System.out.println("elvis.equals(deserializedElvis) = " + elvis.equals(deserializedElvis));
      }
}



```

- elvis와 deserializedElvis는 다른 인스턴스로 인식된다.
- readObject 메서드는 생성자와 같은 역할을 한다. 따라서 생성자와 마찬가지로 방어적으로 작성해야 한다.
- readResolve 메서드는 객체 타입 인스턴스 필드 모두 transient로 선언해야 한다. 그렇지 않으면 역직렬화 과정에서 해당 필드가 null로 초기화된다.

이를 쉽게 해결하는 방법이 있다. Elvis를 Enum 타입으로 만들면 된다.
## Enum 타입 
Enum 타입과 같이 인스턴스가 통제된 직렬화 가능한 클래스를 작성하면 선언된 객체 이외에는 어떠한 인스턴스도 있지 않음을 보장한다. (https://blog.naver.com/PostView.nhn?isHttpsRedirect=true&blogId=kbh3983&logNo=220907314096)   
(하지만 Accessible.setAccessible와 같은 리플렉션을 사용한다면 보장하지 못한다. https://thebook.io/006985/ch04/05/04-01/)

```java
public enum Elvis {
  INSTANCE;
  private String[] favoriteSongs = {"Hound Dog", "Heartbreak Hotel"};

  public void printFavorites() {
    System.out.println(Arrays.toString(favoriteSongs));
  }
}
```
인스턴스 통제를 위해 readResolve를 사용하는 방식이 완전히 쓸모없는 것은 아니다.   
직렬화 가능 인스턴스 통제 클래스를 작성해야 하는데, 컴파일 타임에는 어떤 인스턴스들이 있는지 알 수 없는 상황이라면 열거 타입으로 표현하는 것이 불가능하기 때문이다.

## readResolve 메서드의 접근성
final 클래스인경우 readResolve 메서드는 private 접근 제한자 이어야 한다.

### final 클래스가 아닐 경우 주의사항
- private으로 선언하면 하위 클래스에서 사용할 수 없다. 
- package-private으로 선언하면 같은 패키지에 속한 하위 클래스에서만 사용할 수 있다. 
- protected나 public으로 선언하면 이를 재정의하지 않은 모든 하위 클래스에서 사용할 수 있다. 
- protected나 public이면서 하위 클래스에서 재정의 하지않았다면, 하위 클래스의 인스턴스를 역직렬화하면 상위 클래스의 인스턴스를 생성하여 ClassCastException을 일으킬 수 있다.
