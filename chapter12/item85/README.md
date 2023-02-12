아이템85 자바 직렬화의 대안을 찾으라

요약:
- 자바 직렬화는 바이트 스트림을 역직렬화하는 과정에서 타입들 안의 모든 코드를 수행할 수 있기 때문에 위험하다
- 역직렬화를 자체를 하지 않는 것이 가장 좋은 방어 수단이다.
- JSON 같은 다른 cross-platform 구조화된 데이터를 사용하라.

직렬화란 무엇인가?
- 다른 데이터 형태로 변환하는 것을 말한다.

자바 직렬화란 무엇인가?
- 자바에서 제공하는 라이브러리를 통해서 직렬화하는 것을 말한다.
- Serializable 를 구현하면 객체를 바이트 스트림으로 직렬화할 수 있다.

직렬화 및 역직렬화 코드 참고용
```java
@DisplayName("Serializable을 구현한 Person 객체 직렬화 후, 역직렬화 테스트")
@Test
void writeObjectTest2() throws IOException {
    Person person = new Person("bingbong", 21);

    byte[] serializedPerson;
    try (ByteArrayOutputStream baos = new ByteArrayOutputStream()) {
        try (ObjectOutputStream oos = new ObjectOutputStream(baos)) {
            oos.writeObject(person);
            // 직렬화된 Person 객체
            serializedPerson = baos.toByteArray();
        }
    }

    Person deSerializedPerson = null;

    try (ByteArrayInputStream bais = new ByteArrayInputStream(serializedPerson)) {
        try (ObjectInputStream ois = new ObjectInputStream(bais)) {
            // 역직렬화된 Person 객체
            deSerializedPerson = (Person) ois.readObject();
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }
    }

    assertThat(deSerializedPerson).isNotNull();
    assertThat(deSerializedPerson.name).isEqualTo("bingbong");
    assertThat(deSerializedPerson.age).isEqualTo(21);
}

```

왜 문제인가?
- readObject 가 거의 모든 타입의 객체를 만들 수 있다면 모든 코드를 수행할 수 있기 때문에 문제
- 역직렬화 폭탄

```java
@DisplayName("역직렬화 폭탄 테스트")
@Test
void deserializeBomb() {
    byte[] bomb = bomb();

    // 역직렬화를 하면 엄청 많은 시간이 걸린다.
    deserialize(bomb);
    assertThat(bomb).isNotEmpty();
}

static byte[] bomb() {
    Set<Object> root = new HashSet<>();
    Set<Object> s1 = root;
    Set<Object> s2 = new HashSet<>();
    for (int i = 0; i < 100; i++) {
        Set<Object> t1 = new HashSet<>();
        Set<Object> t2 = new HashSet<>();
        t1.add("foo");
        s1.add(t1);
        s1.add(t2);
        s2.add(t1);
        s2.add(t2);
        s1 = t1;
        s2 = t2;
    }
    return serialize(root); // 직렬화 수행
}
```


해결 방법
- 아무것도 역직렬화 하지마라.
- 정말 해야한다면, 자바9에 추가된 ObjectInputFilter 를 사용하라. 특정 black list, white list 화 시킬수 있다.
