# 아이템10. equals는 일반 규약을 지켜 재정의하라


## equals 재정의 하지 않는 경우

1. 각 인스턴스가 본질적으로 고유하다
  값을 표현하는 것이 아니라 동작하는 개체를 표현하는 클래스일 경우. 예를들어 Thread

2. 인스턴스의 '논리적 동치성'을 검사할 일이 없다.    
   논리적 동치성 : 내용이 같은지 확인하는 것

3. 상위 클래스에서 재정의한 equals가 하위 클래스에도 딱 들어맞는다.    
    아래 코드처럼 List구현체 들은 AbstractList로부터 equals를 상속받는다.

    ```java
    public abstract class AbstractList<E> extends AbstractCollection<E> implements List<E> {
        public boolean equals(Object o) {
            if (o == this)
                return true;
            if (!(o instanceof List))
                return false;

            ListIterator<E> e1 = listIterator();
            ListIterator<?> e2 = ((List<?>) o).listIterator();
            while (e1.hasNext() && e2.hasNext()) {
                E o1 = e1.next();
                Object o2 = e2.next();
                if (!(o1==null ? o2==null : o1.equals(o2)))
                    return false;
            }
            return !(e1.hasNext() || e2.hasNext());
        }
    }
    ```
4. 클래스가 private이거나 package-private이고 equals 메서드를 호출할 일이 없다.   
   equals 자체를 호출하는 걸 막고 싶다면 다음처럼 구현한다.

    ```java
    @Override
    public boolean equals (Object object) {
        throw new AssertionError();
    }
    ```

---

## equals 메서드를 재정의해야 할 때

- 객체 식별성이 아니라 논리적 동치성을 확인해야 하는데 상위 클래스의 equals가 논리적 동치성을 비교하도록 재정의되지 않았을 때
- 값 클래스 
- 값 클래스여도 인스턴스가 2개 이상 만들어지지 않는 **인스턴스 통제 클래스**일 경우. 예) Enum, 싱글톤

## equals 메서드 재정의 일반 규약

equals 메서드는 동치 관계를 구현하며 다음을 만족한다.


### 1. 반사성(reflexivity)

null이 아닌 모든 참조값에 대해 `x.equlas(x) == true`이다.


### 2. 대칭성(symmetry)

서로에 대한 동치 여부에 똑같이 답해야 한다. 
`x.equals(y) == y.equals(x)`
