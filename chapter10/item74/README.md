메서드가 던질 가능성이 있는 모든 예외를 문서화하라. 

검사 예외든 비검사 예외든, 추상 메서드든 구체 메서든 모두 마찬가지이다.

---

검사 예외 : Checked Exception 

비검사 예외 : Runtime Exception

---

[**Javadoc 작성하기**](https://johngrib.github.io/wiki/java/javadoc/)

### 검사 예외는 @throws 태그로 문서화.

- **검사 예외(Checked Exception)는 항상 따로따로 선언하고**,
    
    각 예외가 발생하는 상황을 **자바독의 @throws 태그를 사용**하여 정확히 문서화하자. 
    
    ```jsx
    // java.lang.String.java
    /**
     ...생략
     * @throws  IndexOutOfBoundsException
     *          if {@code beginIndex} or {@code endIndex} is negative,
     *          if {@code endIndex} is greater than {@code length()},
     *          or if {@code beginIndex} is greater than {@code endIndex}
     *
     * @since 1.4
     * @spec JSR-51
     */
    public CharSequence subSequence(int beginIndex, int endIndex) {
        return this.substring(beginIndex, endIndex);
    }
    ```
    
- 공통 상위 클래스 하나로 예외를 던지는 것을 삼가자. ex) Exception, Throwable

- 메서드 사용자에게 각 예외를 대처할 수 있는 힌트를 주지 못할뿐더러 같은 맥락에서 발생할 여지가 있는 다른 예외들 까지 삼켜버릴 수 있기 때문에 API 사용성을 크게 떨어뜨린다.

- 단 Main 메서드는 예외이다. ( main 메서드는 오직 JVM만이 호출 함)

### 비검사 예외도 검사 예외처럼 정성껏 문서화해두면 좋음

- 비검사 예외(Runtime Exception)도 검사 예외(Checked Exception) 처럼 정성껏 문서화 해두면 좋다.

### 비검사 예외은 메서드 시그니처에 추가하지 말자.

- 메서드가 던질 수 있는 예외를 각각 @throws 태그로 문서화하되, **비검사 예외는 메서드 선언의 throws 목록에 넣지 말자.**

- 검사냐 비검사냐에 따라 사용자가 해야할 일이 달라지므로 이 둘을 확실히 구분하는 것이 좋다.

> 한 클래스에 정의된 웬만한 메서드에서 같은 이유로 같은
예외를 던진다면 그 예외를 각각의 메서드가 아니라 클래스 설명에 추가하는 방법도 있다.  ex)NPE
>
