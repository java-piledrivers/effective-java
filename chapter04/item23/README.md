# 아이템23. 태그 달린 클래스보다는 클래스 계층구조를 활용하라

### 태그 달린 클래스란?

두 가지 이상의 의미를 표현할 수 있으며, 그 중 현재 표현하는 의미를 태그값으로 알려주는 클래스

---

태그 달린 클래스 예시 )

```java
class Figure {
    enum Shape { RECTANGLE, CIRCLE };

    // Tag field - the shape of this figure
		final Shape shape;

		// These fields are used only if shape is RECTANGLE
		double length;
		double width;

		// This field is used only if shape is CIRCLE
		double radius;

    // Constructor for circle
    Figure(double radius) {
	    shape = Shape.CIRCLE;
	    this.radius = radius;
    }

    // Constructor for rectangle
    Figure(double length, double width) {
	    shape = Shape.RECTANGLE;
	    this.length = length;
	    this.width = width;
    }

    double area() {
	    switch(shape) {
	        case RECTANGLE:
		        return length * width;
	        case CIRCLE:
		        return Math.PI * (radius * radius);
	        default:
		        throw new AssertionError(shape);
    }
}
```

---

### 왜 사용하면 안될까?

1. 열거 타입 선언, 태그 필드, swicth문 등 `쓸데없는 코드 증가`
2. 한 클래스 내에 여러 구현이 섞여 `가독성 저하`
3. 다른 의미를 위한 코드도 항상 함께 있으니 객체 생성에 필요한 `메모리 사용량 증가`
4. 새로운 의미를 추가할 때 마다 모든 switch문을 찾아 코드를 추가해야함 → `런타임 에러 가능성 증가`
5. 인스턴스 타입만으로 현재 나타내는 의미를 알 길이 없다.

> **태그 달린 클래스는 장황하고, 오류를 내기 쉽고 비효율적이다.**
> 

### 어떻게 처리해야 좋을까?

클래스 계층 구조를 활용하는 `서브타이핑(subtyping)(타입 계층을 구성하기 위해 상속을 사용하는 경우)` 을 활용하자. 계층 클래스의 루트가 되어줄 추상 클래스를 정의하고, 태그 값에 따라 동작이 달라지는 메서드들은 루트 클래스의 추상 메서드로 선언한다.위 코드에서 area()가 이런 경우에 해당한다.

Figure 클래스에는 태그 값에 상관없는 메서드가 하나도 없고, 모든 하위 클래스에서 사용하는 공통 데이터 필드도 없다. 그 결과로 루트 클래스에는 추상 메서드인 area만 남게 된다.

다음으로 루트 클래스를 확장하면 되는데, 원 클래스와 사각형 클래스를 만들고, 각자에 의미에 맞는 데이터 필드를 넣자. 그 후 추상메서드 area()를 알맞게 구현하면 된다.

```java
abstract class Figure {
    abstract double area();
}

class Circle extends Figure {
    final double radius;
    Circle(double radius) { this.radius = radius; }
    @Override double area() { return Math.PI * (radius * radius); }
}

class Rectangle extends Figure {
    final double length;
    final double width
    Rectangle(double length, double width;) {
	    this.length = length;
	    this.width = width;
    }
    @Override double area() { return length * width; }
}
```

타입이 의미 별로 따로 존재하니 변수의 의미를 명시하거나 제한할 수 있고, 또 특정 의미만 매개 변수로 받을 수 있다. 또한, 타입 사이의 자연스러운 계층 관계를 반영할 수 있어서 `유연성`은 물론 `컴파일 타임` 타입 검사 능력을 높여줄 수 있다.

```java
class Square extends Rectangle {
    Square(double side) {
	    super(side, side);
    }
}
```

## 정리

**태그 달린 클래스를 써야 하는 상황은 거의 없다.** 새로운 클래스를 작성하는 데 `태그 필드`가 등장한다면 태그를 없애도 `계층 구조`로 대체하는 방법을 생각해보자.
