# 아이템79 과도한 동기화는 피하라


## 결론
- 동기화 기본 규칙은 동기화 영역에서 가능한 한 일을 적게 하는 것이다.
- 가변 클래스 설계 시 내부 동기화 여부를 고민하자. 그리고 동기화 여부를 반드시 문서화하자.
  - `과도한 동기화의 비용`은 병렬로 실행할 기회를 잃고 `모든 코어가 메모리를 일관되게 보기 위한` 지연시간이다.

--- 

## 1 `동기화 블록 안`에서는 `제어권`을 `클라이언트에게 양도하면 안된다.`

### 🏴 참고 
- 옵저버패턴 간단하게 소개한 글 : https://pjh3749.tistory.com/266

#### 옵저버패턴이란
`옵저버 패턴(observer pattern)`은 객체의 상태 변화를 관찰하는 관찰자들,    
즉 옵저버들의 목록을 객체에 등록하여 상태 변화가 있을 때마다 메서드등을 통해 `객체가 직접 목록의 각 옵저버에게 객체의 상태 변화를 통지하도록` 하는 디자인 패턴이다.     
주로 분산 이벤트 핸들링 시스템을 구현하는 데 사용된다. `발행/구독` 모델로 알려져 있기도 하다.


#### 예시  
객체의 상태가 변화될 때 연관된 객체들에게 알림을 보내는 예시

베디라는 객체는 코치이다. (코치 인터페이스를 구현해야 한다)
- 코치(코치 인터페이스)의 기능은 크루들을 등록한다, 
- 크루들을 등록 해제한다
- 크루들에게 행동을 알린다. 


조금 더 "옵저버" 답게 이름을 리팩토링을 하면    
- `Crew 인터페이스`는 Observer 인터페이스가 되고  : 상태변화를 통보받는 `구독자`
- `Coach 인터페이스`는 Observable 인터페이스가 되겠다. : 객체의 상태변화가 감지되는 `발생자`



### 1-1 동기화 블록 내에서 제어권을 클라이언트에 양도하는 예시
- 동기화된 영역 안에서 `재정의할 수 있는 메서드 호출하면 안됨`

#### 1-1-1 실행되는 코드 

- `ObservableSet<E>`
  - Set을 감싼 래퍼 클래스 
  - 이 클래스의 클라이언트는 집합에 원소가 추가되면 알림을 받을 수 있다.

```java
public class ObservableSet<E> extends ForwardingSet<E> { //발행자
    public ObservableSet(Set<E> set) { super(set); }

    private final List<SetObserver<E>> observers = new ArrayList<>(); //구독자들 = 옵저버들

    //옵저버 구독(등록)
    public void addObserver(SetObserver<E> observer) {
        synchronized(observers) { 
            observers.add(observer); 
        }
    }

    //옵저버 구독해지(삭제)
    public boolean removeObserver(SetObserver<E> observer) {
        synchronized(observers) { 
            return observers.remove(observer); 
        }
    }

	// 외계인 메서드 호출 부분
	// observer의 added : 동기화된 영역에서 재정의할 수 있는 메서드로 제어권이 클라이언트에 있다
    private void notifyElementAdded(E element) {
        synchronized (observers) {
            for (SetObserver<E> observer : observers) 
                observer.added(this, element); //재정의할 수 있는 메서드(외계인 메서드). 즉, 알림 방법을 옵저버가 구현.
        }
    }

    @Override
    public boolean add(E element) {
        boolean added = super.add(element);
        if (added) 
            notifyElementAdded(element); //상태변화가 있으면(Set에 요소가 추가되는 상태 변화) 옵저버들에게 알림호출
        return added;
    }

    @Override
    public boolean addAll(Collection<? extends E> c) {
        boolean result = false;
        for (E element : c) 
            result |= add(element); //add() 메서드를 통해 notifyElementAdded 호출됨.
        return result;
    }
}
```

- `SetObserver<E>`
  - 옵저버 인터페이스(함수형)
  - [x] 옵저버인 SetObserver를 왜 콜백인터페이스로 구현했을까? 
    - 알림 방법을 옵저버가 구현하도록 하기 위해
    - 재정의 메서드로 표현하기 위해 
```java
@FunctionalInterface
public interface SetObserver<E> {
	void added(ObservableSet<E> set, E element); //added : 집합에 원소가 추가됨
}
```

- `실행 코드` main : 0부터 99까지 출력하는 코드
```java
public static void main(String[] args) {
	ObservableSet<Integer> set = new ObservableSet<>(new HashSet<>());
	
	set.addObserver((s, e) -> System.out.println(e));
	
	for(int i = 0; i < 100; i++)
		set.add(i);
}
```

#### 1-1-2 메인 변형 코드
- 집합에 23이 추가되면 구독자 자신을 구독해지
```java
    public static void main(String[] args) {
        ObservableSet<Integer> set = new ObservableSet<>(new HashSet<>());

        set.addObserver(new SetObserver<>() {
            @Override
            public void added(ObservableSet<Integer> set, Integer element) {
                System.out.println(element);
                if (element == 23) //집합에 23이 추가되면 구독자 자신을 구독해지
                    set.removeObserver(this); //구독해지를 통해 재정의 메서드(added)가 호출된다 
            }
        });

        for (int i = 0; i < 100; i++) {
            set.add(i);
        }
    }
```

- 실행결과
  - `ConcurrentModificationException` 발생
  - added() 메서드 호출이 일어난 시점이 `notifyElementAdded`가 관찰자들의 리스트를 순화하는 도중이기 때문이다.  
  - 즉, 리스트에서 원소를 제거하려 하는데 마침 리스트를 순회하는 도중이다.(허용되지 않은 동작)
  - ⭐ `notifyElementAdded`메서드에서 수행하는 순회는 동기화 블록 안에 있으므로 동시 수정이 일어나지 않도록 보장하지만, **정작 자신이 콜백을 거쳐 되돌아와 수정하는 것까지 막지는 못한다.**

```java
Exception in thread "main" java.util.ConcurrentModificationException
	at java.base/java.util.ArrayList$Itr.checkForComodification(ArrayList.java:1042)
	at java.base/java.util.ArrayList$Itr.next(ArrayList.java:996)
	at synchronization.itme7902.ObservableSet.notifyElementAdded(ObservableSet.java:31)
	at synchronization.itme7902.ObservableSet.add(ObservableSet.java:40)
	at synchronization.itme7902.Main.main(Main.java:20)
```

#### 1-2-1 해결방법 - 외계인 메소드 호출을` 동기화 블록 바깥으로 이동


```java
private void notifyElementAdded(E element) {
	List<SetObserver<E>> snapshot = null;
	synchronized(observers) {
		snapshot = new ArrayList<>(observers); //리스트 복사 
	}

	for (SetObserver<E> observer : snapshot) //락없이도 안전한 순회 가능
		observer.added(this, element);
}
```

#### 1-2-2 더 나은 해결방법 - CopyOnWriteArrayList 
- `CopyOnWriteArrayList`는 ArrayList를 구현한 클래스
- 내부를 변경하는 작업은 복사본을 만들어 수행하도록 구현되어 있다.
- 내부 배열이 절대 수정되지 않으니 순회할 때 락이 필요없어 매우 빠르다.
- 수정할 일은 드물고 순회만 빈번히 일어나는 관찰자 리스트 용도로 최적

- `ObserverSet을 CopyOnWriteArrayList를 사용해 구현한 예시`  
  - **명시적으로 동기화한 곳 없어졌다.**
  - 스레드에 안전하고 관찰 가능한 Set
```java
private final List<SetObserser<E>> observers = new CopyOnWriteArrayList<>();

public void addObserver(SetObserver<E> observer) {
    observers.add(observer);
}

public boolean removeObserver(SetObserver<E> observer) {
    return observers.remove(observer);
}

public void notifyElementAdded(E element) {
    for (SetObserver<E> observer : observers) {
         observers.added(this, element);
    }
}
```


---

## 2 가변 클래스를 작성하려거든 두 선택지 중 하나를 따르자
- 가변 클래스 대표적 예시 StringBuffer, StringBuilder. String은 불변(immutable)클래스

### 2-1 **클래스 외부 동기화** : 클래스 내부에서 동기화 하지 말고, 그 클래스를 동시에 사용해야 하는 클래스가 외부에서 동기화하도록 하자
- StringBuffer의 내부적으로 동기화 수행을 해결하고자 등장한 `StringBuilder`. StringBuilder는 `동기화하지 않은` StringBuffer이다. 
- `java.util.concurrent` 이 선택한 방법
  - java.util.Random은 동기화하지 않은 버젼인 `java.util.concuurent.ThreadLocalRandom`으로 대체되었다.

### 2-2 **클래스 내부 동기화** : 동기화를 내부에서 수행해 스레드 안전한 클래스로 만들자
- 클라이언트가 외부에서 객체 전체에 락을 거는 것보다 성능이 더 나아지는 경우에만 이 방법을 선택하자
- `StringBuffer` 인스턴스는 거의 단일 스레드에서 쓰였음에도 `내부적으로 동기화를 수행`했다.
- `java.util`이 선택한 방법

### 참고 사항

#### StringBuilder와 StringBuffer의 차이

https://madplay.github.io/post/difference-between-string-stringbuilder-and-stringbuffer-in-java

#### 클래스를 내부에서 동기화하는 기법
- 락 분할(lock splitting)
- 락 스트라이핑(lock striping)
- 비차단 동시성 제어(nonblocking concurrency control)

#### 동기화 기법 참고 코드
- 코드 예시 : https://github.com/KakaoEnt-Study/Effective-Java-Study/issues/69 
