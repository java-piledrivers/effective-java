# 아이템18 상속보다는 컴포지션을 사용하라

## 상속
- 상위 클래스와 하위 클래스가 같은 개발자가 통제하는 패키지 안이면 안전한 방법
- 상위 클래스와 하위 클래스가 순수한 is-a 관계일 때만 써야 한다. 
- is-a 관계일 때도 패키지가 서로 다르고, 상위 클래스가 확장을 고려해 설계되지 않으면 문제가 될 수 있다.

### 컨포지션 대신 상속을 사용하기로 결정하기 전 자문할 것
1. 확장하려는 클래스의 API에 아무런 결함이 없는가?
2. 결함이 있다면 하위 클래스의 API까지 전파돼도 괜찮은가?


---

## 상속이 가질 수 있는 문제 : 상속은 캡슐화를 깨뜨린다.

### 상속이 가질 수 있는 문제 상황 3가지
1. 상위 클래스 내부 구현 방식에 따른 하위클래스의 구현
2. 상위 클래스에 새로운 메서드 추가 
3. 하위 클래스에 추가한 메서드가 상위 클래스에 생성될 경우


### 1 **상위 클래스 내부 구현 방식에 따른 하위클래스 구현**
- `상위 클래스`는 릴리스마다 `내부 구현이 달라질 수` 있고 그 `여파로 코드가 변경되지 않은 하위 클래스가 오동작할 수` 있기 때문이다.
- 상위클래스가 설계자가 확장을 충분히 고려하고 문서화 해둬야한다. 그렇지 않으면 상위 클래스 수정에 하위 클래스까지 수정되야 한다.

#### 1-1 예시 코드
- 아래 코드의 예상 결과는 3이나, 6이 찍힌다.
- HashSet의 addAll()는 내부적으로 add()를 호출하는데, InstrumentedHashSet에서 add()를 오버라이딩한 부분에 `addCount++` 해당 코드가 있기 때문에 이중으로 addCount가 더해진다.
- 이처럼 **자기 사용(self-use) 는 HashSet의 내부 구현 방식**이라 **다음 릴리스에도 유지될지는 알 수 없다**. 따라서 addAll()이 내부적으로 add()를 호출하는 것을 `가정하여` 하위클래스의 메서드를 `작성`하는 것도 `깨지기 쉽다.`
  

```java
class InstrumentedHashSet<E> extends HashSet<E> {
    //추가된 원소 수
    public int addCount = 0;

    @Override
    public boolean add(E e) {
        addCount++;
        return super.add(e);
    }

    @Override
    public boolean addAll(Collection<? extends E> c) {
        addCount += c.size();
        return super.addAll(c);
    }
    
    //추가된 원소 수 접근 메서드
    public int getAddCount() {
        return addCount;
    }

    public static void main(String[] args) {
        InstrumentedHashSet<String> instrumentedHashSet = new InstrumentedHashSet<>();
        instrumentedHashSet.addAll(List.of("틱", "탁탁", "펑"));
        System.out.println(instrumentedHashSet.getAddCount());
    }
}
```

#### 1-2 나름의 해결책
- 상속을 이용한 나름의 해결 방안으로 아래와 같이 addAll을 재정의하면 상위클래스 동작방식과 상관없이 하위클래스를 올바로 구현되게 할 수 있다.
- `하지만` 상위 클래스의 메서드 동작을 **다시 구현하는 이 방식**은 **어렵고, 시간도 들고, 자칫 오류를 내거나, 성능을 떨어뜨릴 수** 있다.
- 그리고 하위클래스에서 접근할 수 없는 **private 필드를 써야 하는 상황이라면 구현이 불가능**하다.

```java
    // 상속을 이용한 나름의 해결방안
    @Override
    public boolean addAll(Collection<? extends E> c) {
        boolean result = true;
        for (E e : c) {
            result = add(e);
        }
        return result;
    }
```

### 2 **상위 클래스에 새로운 메서드 추가**
- 컬렉션에 추가된 모든 원소가 특정 조건을 만족해야 하는 프로그램 예시. 
- 원소를 추가하는 모든 메서드를 재정의해 필요한 조건을 검사
- 그러나 상위클래스에 또 다른 원소 추가 메서드 생성
- 하위 클래스에서 제대로 재정의하지 않을 경우 허용되지 않은 원소가 추가 될 수 있다.


### 3 **하위 클래스에 추가한 메서드가 상위 클래스에 생성될 경우**
- 하위 클래스에 추가한 메서드가 상위 클래스와 동일한 시그니처이고, 반환타입이 다를 경우 컴파일 조차 되지 않는다.
- 하위 클래스에 추가한 메서드가 상위 클래스와 동일한 시그니처, 반환타입을 가진 메서드가 나중에 추가될 수도 있다. 그 경우 상위 클래스의 메서드가 요구하는 규약을 만족하지 않는 경우일 수 있다.

---

## 컴포지션 
- `컴포지션(composition)` : 기존 클래스가 새로운 클래스의 구성요소로 쓰인다
- `forwarding` : 새 클래스의 인스턴스 메서드가 기존 클래스의 메서드를 호출해 그 결과를 반환하는 것
- `forwarding method` : forwarding하는 메서드 

### 장점
- 그럼으로써 새로운 클래스는 기존 클래스의 **내부 구현 방식의 영향에서 벗어난다.**
- 기존 클래스에 새로운 **메서드가 추가되더라도 영향을 받지 않는다.**

- 상속 방식은 구체 클래스 각각을 따로 확장해야 하며 지원하고 싶은 상위 클래스의 생성자 가각각에 대응하는 생성자를 별도로 정의해줘야 한다. 하지만 컴포지션 방식은 한 번만 구현해두면 어떠한 Set 구현체라도 계측할 수 있으며 기존 생성자들과도 함께 사용할 수 있다.

### 예시 코드 : 컴포지션과 전달방식으로 구현

- InstrumentedSet은 HashSet의 모든 기능을 정의한 Set 인터페이스를 활용해 설계되어 견고하고 유연하다.
- **임의의 Set에 계측 기능을 덧씌워 `새로운 Set`으로 만드는 것이 이 클래스의 핵심이다.**  : `데코레이터 패턴`


- `래퍼클래스(wrapper class)` : 다른 인스턴스를 감싸고 있다.
  - InstrumentedSet는 다른 Set 인스턴스를 감싸고 있다.
- `위임(delegation)` : composition과 forwarding의 조합. 래퍼 객체가 내부 객체에 자기 자신의 참조를 넘기는 경우
- 래퍼클래스 주의점 : 콜백 프레임워크와는 어울리지 않는다.
  
```java
public class InstrumentedSet<E> extends ForwardingSet<E> {
    private int addCount = 0;

    public InstrumentedSet(Set<E> s) {
        super(s);
    }

    @Override
    public boolean add(E e) {
        addCount++;
        return super.add(e);
    }

    @Override public boolean addAll (Collection< ? extends E > c){
        addCount += c.size();
        return super.addAll(c);

    }

    public int getAddCount () {
        return addCount;
    }
}
```

- 재사용할 수 있는 `전달 클래스(forwarding class)`
  - **전달 메서드를 작성하는 게 지루하겠지만 재사용할 수 있는 ㅈ전달 클래스를 인터페이스당 하나씩만 만들어두면 원하는 기능을 덧쒸우는 전달 클래스들을 아주 손쉽게 구현할 수 있다.**
  - 예시 : `구아바`는 모든 컬렉션 인터페이스용 전달 메서들을 전부 구현해뒀다.



```java
class ForwardingSet<E> implements Set<E> {
    private final Set<E> s;
    public ForwardingSet(Set<E> s) { this.s = s;}

    public void clear() { s.clear(); }
    public boolean contains(Object o) {return s.contains(o); }
    public boolean isEmpty() { return s.isEmpty(); }
    public int size() { return s.size(); }
    public Iterator<E> iterator() { return s. iterator(); }
    public boolean add(E e) { return s.add(e); }
    public boolean remove(Object o) {return s.remove(o); }
    public boolean containsAll(Collection<?> c) { return s.containsAll(c); }
    public boolean addAll(Collection<? extends E> c) { return s.addAll(c); }
    public boolean removeAll(Collection<?> c) { return s.removeAll(c); }
    public boolean retainAll(Collection<?> c) { return s.retainAll(c); }
    public Object[] toArray() { return s.toArray(); }
    public <T> T[] toArray(T[] a) { return s.toArray(a); }
    @Override public boolean equals(Object o) { return s.equals(o); }
    @Override public int hashCode() {return s.hashCode(); }
    @Override public String toString() { return s.toString(); }
}
```
