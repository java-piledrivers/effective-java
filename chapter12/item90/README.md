> **Serializable을 구현하기로 결정한 순간 언어의 정상 매커니즘인 생성자 이외의 방법으로 인스턴스를 생성할 수 있게 된다.
이렇게 되면 버그와 보안 문제를 야기하게 되는데 
위 문제를 크게 줄여줄 수 있는 기법이  직렬화 프록시(serialization proxy pattern) 패턴이다.**
> 

****

**방법**

**Step1. 중첩 클래스를 설계한다.  private static으로 선언한다.**

**Step2. 바깥 클래스에 writeReplace/ readResolve 메서드 추가**

         

**장점** 

- **이전 아이템에서 나온 readObejct 의 방어적 복사보다 강력하다.**
- **멤버 필드를 final 로 선언할 수 있기 때문에 진정한 불변으로 만들 수 있다.**

```jsx
// 직렬화 프록시를 사용한 Period 클래스 (479-480쪽)

import java.util.*;
import java.io.*;

// 방어적 복사를 사용하는 불변 클래스
public final class Period implements Serializable {
    private final Date start;
    private final Date end;

    /**
     * @param  start 시작 시각
     * @param  end 종료 시각; 시작 시각보다 뒤여야 한다.
     * @throws IllegalArgumentException 시작 시각이 종료 시각보다 늦을 때 발생한다.
     * @throws NullPointerException start나 end가 null이면 발행한다.
     */
    public Period(Date start, Date end) {
        this.start = new Date(start.getTime());
        this.end   = new Date(end.getTime());
        if (this.start.compareTo(this.end) > 0)
            throw new IllegalArgumentException(
                    start + "가 " + end + "보다 늦다.");
    }

    public Date start () { return new Date(start.getTime()); }

    public Date end () { return new Date(end.getTime()); }

    public String toString() { return start + " - " + end; }

    // 코드 90-1 Period 클래스용 직렬화 프록시 (479쪽)
    private static class SerializationProxy implements Serializable {
        private final Date start;
        private final Date end;

        SerializationProxy(Period p) {
            this.start = p.start;
            this.end = p.end;
        }

        private static final long serialVersionUID =
                234098243823485285L; // 아무 값이나 상관없다. (아이템 87)
    }

    // 직렬화 프록시 패턴용 writeReplace 메서드 (480쪽)
    private Object writeReplace() {
        return new SerializationProxy(this);
    }

    // 직렬화 프록시 패턴용 readObject 메서드 (480쪽)
    private void readObject(ObjectInputStream stream)
            throws InvalidObjectException {
        throw new InvalidObjectException("프록시가 필요합니다.");
    }
}
```

**한계** 

- **Client가 멋대로 확장할 수 있는 클래스에는 적용할 수 없다. (final로 선언되어 있지 않은 클래스)**
- **객체 그래프에 순환이 있는 클래스에도 적용할 수 없다. ( 객체가 서로 참조하는 상황을 말한다. )**
- **방어적 복사보다 상대적으로 속도가 느리다.**
