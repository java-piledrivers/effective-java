#### 자바8에서 default 메서드 등장. 

- default 메소드의 단점.
- default 메소드는 왜 등장했는가?


> 자바 7까지의 세상에서는 모든 클래스가 “현재의 인터페이스에 새로운 메소드가 추가될 일은 영원이 없다”고 가정하고 작성되었다. 

---

####  **Java8 에서는 핵심 컬렉션 인터페이스들에 다수의 디폴트 메서드가 추가**
> 

[https://docs.oracle.com/javase/8/docs/api/java/util/Collection.html](https://docs.oracle.com/javase/8/docs/api/java/util/Collection.html)

#### 자바 8에 Collection 인터페이스에 추가된 removeIf 메소드를 예.

![image](https://user-images.githubusercontent.com/46278436/199999341-8b4f766c-a632-431b-9fba-a7e5e92bf8b0.png)


```java
default boolean removeIf(Predicat<? super E> fiter){
		Objects.requireNonNull(filter);
		boolean result = false;
		for(Iterator<E> it = iterator(); it.hasNext(); ){
				if(it.remove();
				result = true;
		}
		return true; 
}
```

**이 코드보다 더 범용적으로 구현하기도 어렵겠지만 그렇다고 해서 현존하는 모든 Collection 구현체와 잘 어우러지는 것은 아니다.** 

대표적인 예가 org.apache.commons.collection4.collections.SynchronizedCollection이다.  

 sychronized 키워드가 정의되어 있지만 thread safety하지 못하다.

---

#### 아파치 커먼즈 라이브러리의 클래스? 

java.util의 Collections.synchronized Collection 정적 팩터리 메서드가 반환하는 클래스와 비슷하다. 

아파치 버전은 (컬렉션 대신) 클라이언트가 제공한 객체로 락을 거는 능력을 추가로 제공한다.

![image](https://user-images.githubusercontent.com/46278436/200002305-3cda3365-d699-402f-b5a6-658c64f23cd8.png)


 즉, 모든 메서드에서 주어진 락 객체로 동기화한 후 내부 컬렉션 객체에 기능을 위임하는 래퍼 클래스이다. 

<details>
<summary> others default method</summary>
<div markdown="1">

- 자바 8에서 default 메서드 기능을 이용해서 인터페이스에 메서드를 추가한 대표적인 예는 List 인터페이스의 sort 와 collection 인터페이스의 stream 메서드다.
    
    
- 책에는 removeIf만 예로 들었지만 Oracle docs 를 보면 spliterator / stream/ parallelStream 또한 default로 정의되어 있음을 확인할 수 있다. 
    
    ![image](https://user-images.githubusercontent.com/46278436/199999749-9caf8e91-b0c1-4694-83dc-669f3294248a.png)


[https://www.codejava.net/java-core/collections/understanding-collections-and-thread-safety-in-java](https://www.codejava.net/java-core/collections/understanding-collections-and-thread-safety-in-java)

</div>
</details>


<br><br>

---

### 아파치에 SynchornizedCoillection class 에 대해 좀 더 알아보자. 

[https://commons.apache.org/proper/commons-collections/apidocs/org/apache/commons/collections4/collection/SynchronizedCollection.html](https://commons.apache.org/proper/commons-collections/apidocs/org/apache/commons/collections4/collection/SynchronizedCollection.html)

- 이 책을 쓰는 시점엔 removeIf 메서드를 **재정의 하고 있지 않다.**
- 이 클래스를 자바 8과 함께 사용한다면 (그래서 removeIf의 디폴트 구현을 물려받게 된다면), 자신이 한 약속을 더 이상 지키지 못하게 된다.
    
    다시 말해 모든 메소드 호출을 알아서 동기화해주지 못한다. 
    
- removeIf의 구현은 동기화에 관해 아무것도 모르므로 락객체를 사용할 수 없다.
- 따라서 SynchronizedCollection 인스턴스를 여러 스레드가 공유하는 환경에서 한 스레드가 removeIf를 호출하면 ConcurrentModificationException이 발생하거나 다른 예기치 못한 결과로 이어질 수 있다.

[What is the reason why “synchronized” is not allowed in Java 8 interface method](https://stackoverflow.com/questions/23453568/what-is-the-reason-why-synchronized-is-not-allowed-in-java-8-interface-methods)

[java - the reason why "synchronized" is not allowed in java 8 interface methods](https://itecnote.com/tecnote/java-the-reason-why-synchronized-is-not-allowed-in-java-8-interface-methods/)

[what default methods are and why do we need them ?](https://www.logicbig.com/tutorials/core-java-tutorial/java-8-enhancements/default-methods-in-interfaces.html)

---

### 자바 플랫폼 라이브러리에서도 이런 문제를 예방하기 위해 일련의 조치를 취했다.

예를 들어 구현한 인터페이스의 디폴트 메서드를 재정의하고, 다른 메서드에서는 디폴트 메서드를 호출하기 전에 필요한 작업을 수행하도록 했다. 

예컨대 Collections.synchronizedCollection이 반환하는 package-private  클래스들은 removeIf를 재정의하고, 

이를 호출하는 다른 메서드들은 디폴트 구현을 호출하기 전에 동기화를 하도록 했다. 

하지만 자바 플랫폼에 속하는 않은 제 3의 기존 

> default method는 (컴파일에 성공하더라도) 기존 구현체에 런타임 오류를 일킬 수 있다. 
흔한 일은 아니지만, 나에게는 일어나지 않으리라는 보장도 없다. 
자바 8은 컬렉션 인터페이스에 꽤 많은 디폴트 메서드를 추가했고, 그 결과 기존에 짜여진 많은 자바 코드가 영향을 받은 것으로 알려졌다.
> 

기존 인터페이스에 디폴트 메서드로 새 메서드를 추가하는 일은 꼭 필요한 경우가 아니면 피해야 한다. 

추가하려는 디폴트 메서드가 기존 구현체들과 충돌하지는 않을지 심사숙고해야 함도 당연하다. 

반면 새로운 인터페이스를 만드는 경우라면 표준적인 메서드 구현을 제공하는데 아주 유용한 수단이며, 그 인터페이스를 더 쉽게 구현해 활용할 수 있게끔 해준다.  (Item20)


---

### 정리

- 디폴트 메서드는 인터페이스로부터 메서드를 제거하거나 기존 메서드의 시그니처를 수정하는 용도로 아님을 명시해야 한다.
- 이런 형태로 인터페이스를 변경하면 반드시 기존 클라이언트를 망가뜨리게 된다.

> 핵심은 명백하다. 디폴트 메소드라는 도구가 생겼더라도 인터페이스를 설계할 때는 여전히 세심한 주의를 기울여야 한다.
> 
- 디폴트 메서드로 기존 인터페이스에 새로운 메서드를 추가하면 커다란 위험도 딸려 온다.
- 인터페이스에 내재된 작은 결함도 사용자 입장에서는 짜증나는 일인데, 심각하게 잘못된 인터페이스라면 이를 포함한 API에 어떤 재앙을 몰고 올지 알수 없다.

- 새로운 인터페이스라면 릴리즈 전에 반드시 테스트를 해야 한다. 

- 또한 각 인터페이스의 인스턴스를 다양한 작업에 활용하는  여러 개 만들어봐야 한다. 

- 새 인터페이스가 의도한 용도에 잘 부합하는지를 확인하는 길은 이처럼 험난하다. 

- 인터페이스를 릴리스한 후라도 결함을 수정하는게 가능한 경우도 있겠지만, 절대 그 가능성을 기대서는 안된다.
