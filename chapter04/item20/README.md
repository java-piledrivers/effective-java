# [아이템 20] 추상 클래스보다는 인터페이스를 우선하라


### 기존 클래스에도 손쉽게 새로운 인터페이스를 구현해넣을 수 있다.
기존 클래스에 새로운 인터페이스를 구현하려면, 간단하게 해당 인터페이스를 클래스 선언문에 implements 구문으로 추가하고 인터페이스가 제공하는 메소드들을 구현하기만 하면 끝이다. 반면에 기존 클래스에 새로운 추상 클래스를 상속하기에는 어려움이 따른다.   
두 클래스가 같은 추상 클래스를 상속하길 원한다면 계층적으로 두 클래스는 공통인 조상을 가지게 된다. 만약 두 클래스가 어떠한 연관도 없는 클래스라면 클래스 계층에 혼란을 줄 수 있다.


### 인터페이스는 믹스인 정의에 안성맞춤이다.
- 믹스인이란 어떤 클래스의 주 기능에 추가적인 기능을 혼합한다 하여서 믹스인이라고 한다. 그러므로 믹스인 인터페이스는 어떤 클래스의 주 기능이외에 믹스인 인터페이스의 기능을 추가적으로 제공하게 해주는 효과를 준다.   
- 추상 클래스가 믹스인 정의에 맞지않은 이유는 기존 클래스에 덧씌울 수 없기 때문이다. 자바는 단일 상속을 지원하기 때문에 한 클래스가 두 부모를 가질 수 없고 부모와 자식이라는 클래스 계층에서 믹스인이 들어갈 합리적인 위치가 없다.
- 믹스인 인터페이스엔 대표적으로 Comparable, Cloneable, Serializable 이 존재한다.

```java
public class Mixin implements Comparable {
	@Override
	public int compareTo(Object o) {
    	return 0;
    }
}
```

### 인터페이스로는 계층구조가 없는 타입 프레임워크를 만들 수 있다.
- 추상 클래스로 만들었기 때문에 Singer 클래스와 SongWriter 클래스를 둘다 상속할 수 없어 SingerSongWriter라는 또 다른 추상 클래스를 만들어서 클래스 계층을 표현할 수 밖에 없다.    
- 만약 이런 Singer 와 SongWriter와 같은 속성들이 많이 있다면, 그러한 클래스 계층구조를 만들기 위해 많은 조합이 필요하고 결국엔 고도비만 계층구조가 만들어질 것이다. 이러한 현상을 조합 폭발이라고 한다.   


#### 인터페이스를 사용한 경우
```java
public class People implements Singer, SongWriter {

  @Override
  public void sing(String s) {

  }

  @Override
  public void compose(int chartPosition) {

  }
}
```

#### 추상 클래스를 사용한 경우
```java
public abstract class Singer {
    abstract void sing(String s);
}

public abstract class SongWriter {
    abstract void compose(int chartPosition);
}

public abstract class SingerSongWriter {
    abstract void compose(int chartPosition);
    abstract void sing(String s);
}

//

public class People extends SingerSongWriter {

  @Override
  public void sing(String s) {

  }

  @Override
  public void compose(int chartPosition) {

  }
}
```

### 래퍼 클래스 관용구와 함께 사용하면 인터페이스는 기능을 향상키는 안전하고 강력한 수단이 된다.
- 타입을 추상 클래스로 정의해두면 해당 타입에 기능을 추가하는 방법은 상속 뿐이다. 
- 상속해서 만든 클래스는 래퍼 클래스보다 활용도가 떨어지고 쉽게 깨진다.

### 구현이 명백한 것은 인터페이스의 디폴트 메서드를 사용할 수 있다.
- 인터페이스의 메서드 중 구현 방법이 명백한 것이 있다면, 그 구현을 디폴트 메서드로 제공해줄 수 있다. 이는 프로그래머의 일을 상당히 덜어준다.
- 디폴트 메서드를 제공할 때는 상속하려는 사람을 위한 설명을 @implSpec 자바독 태그를 붙여 문서화해야 한다.

### 추상 골격 구현(skeletal implementation) 클래스
https://dzone.com/articles/favour-skeletal-interface-in-java

### * 상속(is-a)와 인터페이스(has-a)
https://minusi.tistory.com/entry/%EA%B0%9D%EC%B2%B4-%EC%A7%80%ED%96%A5%EC%A0%81-%EA%B4%80%EC%A0%90%EC%97%90%EC%84%9C%EC%9D%98-has-a%EC%99%80-is-a-%EC%B0%A8%EC%9D%B4%EC%A0%90
