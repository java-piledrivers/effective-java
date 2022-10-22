### **☑ marker interface**

: **메서드가 포함되어 있지 않은 빈 인터페이스** 

**ex) Cloneable, Serializable** 

![image](https://user-images.githubusercontent.com/46278436/197315242-979eb66b-4f60-4da1-b4ab-aa7515968bf7.png)


**Cloneable 인퍼테이스에 clone 메소드가 없다. 그렇다면 어디에?**

![image](https://user-images.githubusercontent.com/46278436/197315268-8c73b84a-3301-473f-b8a9-fdeddf7e434d.png)

Object 에 구현되어 있다. 접근지정자 protected 사용. 


### ☑ cloneable 문제점. 

**Cloneable 을 구현하는 것만으로는 외부 객체에서 clone 메서드를 호출할 수 없는 문제가 발생한다.** 

- 리플랙션 사용하는 것이 가능하지만 100% 성공하는 것이 아니다.
- 해당 객체가 접근이 허용된 clone 메서드를 제공한다는 보장이 없기 때문이다.

위의 여러 문제점에도 Cloneable 방식은 여러 문제점에도 불구하고 널리 사용한다. 



### **☑ 언제 / 어떻게 사용하는지 ?**

**shallow copy / deep copy 
[참조](https://www.javabrahman.com/corejava/cloning-in-java-shallow-copy-and-deep-copy-tutorial-with-examples/)**  

**객체를 복사하는 방법** 

- **clone 을 사용하는 이유?**
    - **원본객체를 안전하게 보호하기 위해서**

> **clone 메서드는 사실상 생성자와 같은 효과를 낸다. <br>
즉, clone 은 원본 객체에 아무런 해를 끼치지 않는 동시에 복제된 객체의 불변식을 보장해야 한다.**
> 

![image](https://user-images.githubusercontent.com/46278436/197315369-50dce292-3a1c-488d-b6dd-761fa0894478.png)
- **Java에서 클래스의 객체 복제를 하려면 3개의 액터가 함께 동작해야 한다.**
- **Cloneable interface를 상속받지 않으면** $CloneNotSupportedException$ **이 발생한다.**

### **☑Stack 클래스 예시, 해시 테이블용 메서드(내부 버킷)  예시.**

**책 참고**

### ☑Cloneable 사용시 주의사항.

**Cloneable을 구현한 스레드 안전 클래스를 작성할 때에는 clone 메서드 역시 적절히 동기화해줘야 한다. ITEM 78**

**( Object 의 clone 메서드는 동기화를 신경쓰지 않았다. )** 

**Cloneable을 구현하는 모든 클래스는 clone을 재정의해야 한다.** 

**모든 작업에서 이 작업이 필요한 것은 아니다.** 

- **Cloneable을 이미 구현한 클래스를 확장한다면 어쩔 수 없이 clone을 잘 작동하도록 구현해야 한다.**
- **그렇지 않은 상황에서는 복사 생성자와(conversion constructor) 복사 팩터리(conversion factory) 라는 더 나은 객체 복사 방식을 제공할 수 있다 .**

<br>

### ☑ 복사 생성자와 복사 팩터리란

[복사 생성자와 복사 팩터리 참조](https://www.techiedelight.com/ko/copy-constructor-factory-method-java/)

[conversion constructor / conversion factory](https://docs.telerik.com/help/justcode/refactorings-convert-constructor-to-factory-method.html) 

**복사 생성자와 복사 팩터리는 해당 클래스가 구현한 인터페이스 타입의 인스턴스를 받을 수 있다.** 

**예컨대 관례상 모든 범용 컬렉션 구현체는 Collection이나 Map 타입을 받는 생성자를 제공한다.** 

**인터페이스 기반의 복사 생성자와 복사 팩터리의 더 정확한 이름은 ‘변환 생성자’와 ‘변환 팩터리’라고 한다.** 

<br>

### ☑ 정리

**Cloneable이 몰고 온 모든 문제를 되짚어 봤을 때, 새로운 인터페이스를 만들 때는 절대 Cloneable을 확장해서는 안 되며, 새로운 클래스도 이를 구현해서는 안 된다.** 

**final 클래스라면 Cloneable을 구현해도 위험이 크지 않지만, 성능 최적화 관점에서 검토한 후 별 다른 문제가 없으면 드물게 허용해야 한다.  item67**

**기본 원칙은 ‘복제 기능은 생성자와 팩터리를 이용하는게 최고’라는 것이다. 단, 배열만은 clone 메서드 방식이 가장 깔끔한, 이 규칙의 예외라 할 수 있다.**
