# 아이템 87 커스텀 직렬화 형태를 고려해보라

### 결론
1. 클래스 직렬화하기로 했다면 어떤 직렬화 형태를 사용할지 심사숙소해라
  - 자바의 `기본 직렬화 형태`는 객체 직렬화 결과가 해당 객체의 **논리적 표현에 부합할 경우**에만 사용하자
  - 그렇지 않으면 `커스텀 직렬화 형태` 고안하라. 이상적인 직렬화 형태는 물리적인 모습과 독립된 놀리적인 모습만을 표현해야 한다.
2. 직렬화 형태도 공개 메서드 설계하는 것처럼 시간들여 설계해야 한다.
  - 직렬화 형태에 포함된 필드는 **직렬화 호환성 유지를 위해 마음대로 제거할 수 없다.**
  - 잘못된 직렬화 형태는 클래스의 복잡성과 성능에 부정적인 영향을 남긴다.

---

## 1 기본 직렬화 형태에 적합한 후보
성명은 논리적으로 이름, 성, 중간이름이라는 3개의 문자열로 구성된다.   
인스턴스 필드는 이 논리적 구성요소를 정확히 반영했기에 기본 직렬화 형태에 적합한 후보이다.


- 필드가 모두 **private임에도 문서화 주석**이 달려있는데, 그 이유는 `직렬화 형태 공개 API`에 속하기 때문이다.
- `@serial` 태그로 기술한 내용은 API문서에서 직렬화 형태 설명 페이지에 기록된다. 
```java
class Name implements Serializable {    
    /**
     * 성, null이 아니여야 함.
     * @serial
     */
    private final String lastName;

    /**
     * 이름, null이 아니여야 함.
     * @serial
     */
    private final String firstName;

    /**
     * 중간이름, 중간이름이 없다면 null.
     * @serial
     */
    private final String middleName;

    public Name(String lastName, String firstName, String middleName) {
        this.lastName = lastName;
        this.firstName = firstName;
        this.middleName = middleName;
    }
}
```


### 1-1 기본 직렬화 형태에 적합한 후보일지라도 고려할 사항 : **불변식 보장, 보안**
- 불변식 보장과 보안을 위해 `readObject`메서드를 제공해야 할 때가 많다.
- [ ] Name 클래스 예시에서는 **readObject메서드**가 lastName, firstName 필드가 **null이 아님을 보장해야** 한다.(아이템88, 90참고)


---

## 2 기본 직렬화 문제점(1) 

### 기본 직렬화 형태에 적합하지 않은 예시
- `StringList` 
  - 문자열 리스트 표현한 코드 
  - **논리적**으로 일련의 문자열 표현
  - **물리적**으로는 문자열들을 이중 연결 리스트로 연결 
  - **기본 직렬화 형태 사용 할 경우** : 각 노드의 양방향 연결 정보가 포함된다.

```java
class StringList implements Serializable {
    private int Size = 0;
    private Entry head = null;

    private static class Entry implements Serializable {
        String data;
        Entry next;
        Entry previous;
    }
}
```

### 2-1 공개 API가 현재 내부 표현 방식에 영구히 묶인다.
- StringList 예시에서 private 클래스 StringList.Entry가 공개 API가 되어 버린다.
- 다음 릴리즈에서 내부 표현방식을 바꿔도 StringList 클래스는 여전히 연결 리스트로 표현된 입력도 처리할 수 있어야 한다. 

### 2-2 너무 많은 공간을 차지할 수 있다.
- StringList는 연결 리스트의 모든 엔트리와 연결 정보 기록. 
- 그러나 연결 정보는 내부 구현에 해당하므로 직렬화 형태에 포함할 필요없다.
- 즉, 불필요한 정보가 포함되어 직렬화 형태가 커지면 디스크 저장하거나 네트워크 전송 속도 느려진다.

### 2-3 시간이 너무 많이 걸릴 수 있다.
- 직렬화 로직은 객체 그래프의 위상 정보가 없으니 그래프를 직접 순회할 수 밖에 없다. 
- StringList는 간단히 다음 참조를 따라 가보는 정도로 충분하다.

### 2-4 스택 오버플로를 일으킬 수 있다.
- 기본 직렬화 과정은 객체 그래프를 재귀 순회한다. 그 과정에서 스택 오버플로를 일으킬 수 있다.

---

## 3 커스텀 직렬화 

### 3-1 `StringList`의 커스텀 직렬화 형태
- writeObject 메서드의 문서화 주석 : private 메서드이지만 직렬화 형태 포함되는 `공개 API`임으로 문서화 주석 필요
- `@serialData` : 직렬화 형태 페이지에 추가


```java
class StringListCustom implements Serializable {
    private transient int size = 0;         // transient 키워드를 통해 Serialize 하는 과정에서 제외
    private transient Entry head = null;    // transient 키워드를 통해 Serialize 하는 과정에서 제외

    private static class Entry {    // 이제는 직렬화되지 않는다.
        String data;
        Entry next;
        Entry previous;
    }

    // 문자열을 리스트에 추가한다.
    public final void add(String s) { ... }

    /**
     * 이 {@code StringList} 인스턴스를 직렬화한다.
     *
     * @serialData 이 리스트의 크기(포함된 문자열의 개수)를 기록한 후
     * ({@code int}), 이어서 모든 원소를 (각각은 {@code String})
     * 순서대로 기록한다.
     */
    private void writeObject(ObjectOutputStream s) throws IOException {
        s.defaultWriteObject();     // 인스턴스 필드가 모두 trasient라도 향후 릴리즈를 위해 무조건 하라고 직렬화 명세에 있다. 
        s.writeInt(size);

        // 모든 원소를 올바른 순서로 기록한다
        for (Entry e = head; e != null; e = e.next) {
            s.writeObject(e.data);
        }
    }

    private void readObject(ObjectInputStream stream)
        throws IOException, ClassNotFoundException {
        stream.defaultReadObject(); // 인스턴스 필드가 모두 trasient라도 향후 릴리즈를 위해 무조건 하라고 직렬화 명세에 있다. 
        int numElements = stream.readInt();
        
        // 모든 원소를 읽어 이 리스트에 삽입한다.
        for (int i = 0; i < numElements; i++) {
            add((String) stream.readObject());
        }
    }

    // ... 생략
}
```

### 3-2 커스텀 직렬화한 `StringList`의 직렬화 상태의 성능 
- 원래 버젼의 절반 정도의 공간 차지 : 스택 오버 플로 미발생 -> 직렬화 크기 제한 없음
- 저자 컴퓨터에서 2배 정도 빠르게 수행

---

## 4 기본 직렬화의 문제점(2) - 역직렬화 

### 4-1 역직렬화 시 불변식의 정확성
- 역직렬화할 때, 불변식이 세부 구현에 따라 달라지는 객체에서는 불변식의 정확성이 깨질 수 있다.
  - 예시 : 해시테이블

#### 해시테이블
- 해시테이블은 `key-value` `엔트리`를 담은 `해시 버킷`을 차례로 나열한 형태

    ```
       해시 버킷        해시 버킷
     +----------+     +----------+
     | key-value|     | key-value|              
     +----------+     +----------+ 
    ```
- **어떤 엔트리를 어떤 버킷에 담을지**는 key에서 구현한 `해시코드가 결정`한다. **해시코드 계산 방식은 구현에 따라 달라질 수** 있고 계산할 때마다 달라지기도 한다.
- 이런 경우 해시테이블을 직렬화한 후 역직렬화하면 불변식이 심각하게 훼손된 객체들이 생겨날 수 있다.

### 4-2 역직렬화 시  trasient 필드
- `transient`
  - 기본 직렬화 대상이 아님을 지정하는 한정자
  - 해당 객체의 논리적 상태와 무관한 필드일 경우 사용
- 기본 직렬화를 사용하면 **trasient 필드는 역직렬화될 때 기본값으로 초기화**된다.
- 기본 값을 그대로 사용하면 안되는 경우는 readObject메서드에서 defaultReadObject 호출 후, 해당 필드를 원하는 값으로 복원한다. 또는 그 값을 처음 사용할 때 초기화 하는 방법도 있다.

---

## 5 직렬화 방법이 무엇이든 고려해야 할 사항

### 5-1 객체의 전체 상태를 읽는 메서드에 적용해야 하는 동기화 매커니즘을 직렬화에도 적용해야 한다
- 모든 메서드를 synchronized로 선언하여 스레드 안전하게 만든 객체에서 기본 직렬화를 사용하려면 **writeObject도 synchronized로 선언해야** 한다.
  
### 5-2 어떤 직렬화 형태를 택하든 직렬화 가능 클래스 모두에 직렬 버전 UID를 명시적으로 부여하자
- 직렬 버전 UID를 명시하지 않은 경우 런타임에 값을 무작위로 생성하는데 비용이 크다
- 직렬 버전 UID는 고유할 필요는 없고, 기존 클래스 호환성을 끊고 싶을 경우 변경하는 것으로 해결할 수 있다 
- 구버전으로 직렬화된 인스턴스와의 호환성을 끊으려는 경우를 제외하고는 직렬 버전 UID를 절대 수정하지 말 것

```java
private static final long serialVersionUID = 9349139012902L; // 무작위로 고른 long 값
```

---

## 그외 참고 사항
### Javadocs 직렬화 표현
- `@serial`: javadoc 필드 명시
- `@serialData` : javadoc 메서드 명시
