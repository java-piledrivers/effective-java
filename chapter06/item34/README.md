# 아이템34 int 상수대신 열거 타입을 사용하라

## 열거타입의 장점

### 자바의 열거 타입은 완전한 형태의 클래스이다.
#### 1. 상수 하나당 자신의 인스턴스를 하나씩 만들어 public static final 필드로 공개한다.
#### 2. 컴파일 타임 안전성을 제공한다. : 참조는 정의한 상수 중 하나임이 확실하다.
#### 3. 각자의 이름공간이 있다.
- 이름이 같은 상수도 평화롭게 공존한다.
- 상수 추가, 순서 변경에도 **재 컴파일하지 않아도 된다.**     
공개되는 것이 오직 **필드의 이름 뿐**이라 클라이언트로 컴파일되어 각인되지 않기 때문이다.

---

## 열거타입에 메서드나 필드 추가는 어떨 때 필요한 기능일까
- 상수와 연관된 데이터를 해당 상수 자체에 내재시킨다.

### **태양계의 여덟 행성** 예시

#### 1 행성에 대한 정보
- 각 행성에는 `질량`, `반지름`이 있다. 
- 두 속성을 이용해 `표면 중력`을 계산할 수 있다.
- 객체의 질량이 주어지면 `행성 표면에 있을 때의 무게`를 계산할 수 있다.

#### 2 행성을 Java enum으로 코드화
```java
public enum Planet {
	MERCURY(3.302e+23,2.439e6),
	VENUS(4.869e+24,6.052e6),
	EARTH(5.975e+24, 6.378e6),
	MARS(6.419e+23,3.393e6),
	JUPITER(1.899e+27,7.149e7),
	SATURN(5.685e+26,6.027e7),
	URAUS(8.683e+25,2.556e7),
	NEPTUNE(1.024e+26,2.477e7);

	private final double mass; //질량
	private final double radius; //반지름
	private final double surfaceGravity; //표면중력
 
    // 생성자에서 인스턴스 필드값을 계산해 넣어줄 수 있다.
    Planet(double mass, double radius) {
        this.mass = mass;
        this.radius = radius;
        surfaceGravity = G * mass / (radius * radius);
    }
    
	public double surfaceWeight(double mass) {
		return mass * surfaceGravity;
	}

    //각 필드의 getter제공
}
```

#### 3 열거타입에 메서드나 필드 추가는 어떨 때?
- ⭐ **열거 타입 상수 각각을 특정 데이터와 연결지으려면 생성자에서 데이터를 받아 인스턴스 필드에 저장하면 된다.**
- 열거 타입은 근본적으로 불변이라서 모든 필드는 final 이여야한다. 필드는 public으로 선언해도 되지만 private으로 두고 getter처럼 별도의 public method를 두는 게 좋다.
-  surfaceGravity(표면중력)은 언제든 계산할 수 있지만 **최적화**를 위해 계산해 저장해둔다.
-  선언하지 않아도 기본적으로, 자신 안에 정의된 상수들의 값을 배열에 담아 반환하는 정적 메서드 `values()`를 제공한다.
-  자동 생성 메서드 `valueOf(String)`, 상수 이름을 받아 그 이름에 해당하는 상수 반환 메서드를 제공한다. 

---

## 열거 타입에서 상수 하나 제거했을 때 클라이언트에 벌어지는 일
- 제거된 상수를 참조하지 않은 클라이언트 : 아무 영향 없다!
- 제거된 상수를 참조하는 클라이언트 
  - 재컴파일 하면 **상수를 참조하는 줄에서** `컴파일 오류`가 발생한다.
  - 재컴파일 안하면 **상수를 참조하는 줄에서** `런타임`에러가 발생한다.
  - 상수 참조 줄에서 오류가 발생하므로 바람직한 대응이다!

---

## 열거 타입 잘 사용하기

### 1 private , package-private 메서드로 구현한다
- 열거 타입을 선언한 클래스나 패키지에서만 유용한 기능은 **private , package-private 메서드로 구현한다**
  - [참고] package-private에 대한 생각 : https://hyeon9mak.github.io/Java-dont-use-package-private/

### 2 더 다양한 기능을 제공하는 상수
- 상수마다 동작이 달라져야 하는 상황
- 참고 : https://techblog.woowahan.com/2527/

#### 2-1. Java8버젼부터 : 상수가 계산식을 갖도록 지정
상수가 계산식 갖도록 enum 정의

```java
public enum CalculatorType {
    CALC_A(value -> value), 
    CALC_B(value -> value * 10), 
    CALC_C(value -> value * 3), 
    CALC_D(value -> 0L);
    
    private Function<Long, Long> expression;
    
    CalculatorType(Function<Long, Long> expression) {
      this.expression = expression;
    }
    
    public Long calculate(Long value) {
      return expression.apply(value);
    }
}
```

- 실행 코드
  
```java
public class EnumExam3 {
	public static void main(String[] args) {
		Long originValue = 10_000L;

		CalculatorType codeA = CalculatorType.CALC_A;
		Long resultA = codeA.calculate(originValue);

		CalculatorType codeC = CalculatorType.CALC_C;
		Long resultC = codeC.calculate(originValue);

		System.out.println("resultA:" + resultA + ", resultC:" + resultC);
	}
}
```

- output
  - resultA:10000, resultC:30000


#### 2-2. Java7버젼까지 : 추상메서드 선언
열거형에 추상메서드를 선언하면 각 열거형 상수가 추상 메서드를 반드시 구현해야 합니다. 
그럼으로써 클래스의 행위를 표현할 수 있습니다. 

```java
public enum CalculatorType {

  CALC_A{Long calculate(Long value){return value;}},
  CALC_B{Long calculate(Long value){return value * 10;}},
  CALC_C{Long calculate(Long value){return value * 3;}},
  CALC_D{Long calculate(Long value){return 0L;}};
  
  abstract Long calculate(Long value);
}
```

### 3 상수끼리 코드를 공유해야 할 때는 
#### 비추천 : 상수끼리 코드를 공유해야 할 때 switch문

```java
enum PayrollDay {
    MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY, SUNDAY

    private static final MINS_PER_SHIFT: Int = 8 * 60 // enums must not contatin stored properties

    int pay(minutesWorked: Int, payRate: Int) {
        int basePay: Int = minutesWorked * payRate;

        int overtimePay;
        switch(this) {
        case SATURDAY: case SUNDAY: // 주말
            overtimePay = basePay / 2;
            break;
        default: // 주중
            overtimePay =
                minutesWorked <= MINS_PER_SHIFT ?
                    0 : (minutesWorked - MINS_PER_SHIFT) * payRate / 2;
        }
        return basePay + overtimePay;
    }
}
```
- 그러나 상수별 동작을 혼합해 넣을 때는 switch문이 좋은 선택이 될 수 있다. 


#### 비추천 : 도우미 메서드로 작성 후, 상수가 자신에게 필요한 메서드를 적절히 호출
- switch와 동일한 단점 
- 새로운 상수를 추가하면서 메서드를 재정의하지 않으면 주말용 코드를 제공받지 못하고 평일용 코드를 물려받는다.

#### 추천 : **전략 열거 타입**
- 상수 추가할 때 잔업수당 `전략`을 선택하도록 하는 것

```java
enum PayrollDay {
    MONDAY(WEEKDAY), TUESDAY(WEEKDAY), WEDNESDAY(WEEKDAY), THURSDAY(WEEKDAY), FRIDAY(WEEKDAY), SATURDAY(WEEKEND), SUNDAY(WEEKEND);
  
    private final PayType payType;
  
    PayrollDay(PayType payType) { this.payType = payType; }
  
    int pay(int minutesWorked, int payRate) {
        return payType.pay(minutesWorked, payRate); 
    }
  
    //전략 열거 타입
    enum PayType {
        WEEKDAY {
            int overtimePay(int minutesWorked, int payRate) {
               return minutesWorked <= MINS_PER_SHIFT ? 0 : (minutesWorked - MINS_PER_SHIFT) * payRate / 2;
            }
        },
        WEEKEND {
            int overtimePay(int minutesWorked, int payRate) {
                return minutesWorked * payRate / 2;
            }
        };
      
        abstract int overtimePay(int mins, int payRate);
        private static final int MINS_PER_SHIFT = 8 * 60;
      
        int pay(int minsWorked, int payRate) {
            int basePay = minutesWorked * payRate;
            return basePay + overtimePay(minsWorked, payRate);
        }
    }
}
```


