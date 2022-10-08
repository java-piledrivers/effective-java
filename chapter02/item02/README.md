생성자에 매개변수가 많다면 빌더를 고려하라
(GOF의 Builder 패턴과 다릅니다)



점층적 생성자 패턴: 매개변수에 따라 생성자의 수를 계속 늘리는 방식으로  코드를 읽기 어렵고, 오류가 발생하기 쉽다.

자바빈즈 패턴: 매개변수 없이 생성자로 객체를 만든후 세터 메서드들을 통해 매개변수 값을 설정하는 방식으로 객체를 완전히 생성하기 전까지는 일관성을 보장할 수 없는 문제가 있다.


빌더 패턴은 점층적 생성자 패턴의 안정성과 자바빈즈의 가독성을 겸비한 패턴이다.

## 빌더 구현 
```java
public class NutritionFacts {
	private final int servingSize;
	private final int servings;
	private final int calories;
	private final int fat;
	private final int sodium;
	private final int carbohydrate;
	
	public static class Builder {
		
		//필수 매개변수는 생성자를 통해 객체 생성할때 받아온다
		private final int servingSize;
		private final int servings;
		
		//객체에 필수 매개변수가 포함됨을 보장할 수 있음
		public Builder(int servingSize, int servings) {
			this.servingSize = servingSize;
			this.servings = servings;
		}
		
		//선택 매개변수 - 기본값으로 초기화
		private int calories = 0;
		private int fat = 0;
		private int sodium = 0;
		private int carbohydrate = 0;
		
		//생성자 대신 메서드를 사용함 - 자바빈즈처럼 가독성을 확보할 수 있음
		public Builder caloried(int val)
		{ calories = val; return this; }
		public Builder fat(int val)
		{ fat = val; return this; }
		public Builder sodium(int val)
		{ sodium = val; return this; }
		public Builder carbohydrate(int val)
		{ carbohydrate = val; return this; }
		
		public NutritionFacts build() {
			return new NutritionFacts(this);
		}
		
	}
	
	//객체 생성은 Builder를 통해서만 이루어진다.
	private NutritionFacts(Builder builder) {
		servingSize = builder.servingSize;
		servings = builder.servings;
		calories = builder.calories;
		fat = builder.fat;
		sodium = builder.sodium;
		carbohydrate = builder.carbohydrate;
		
	}
	
	public String toString() {
		return servingSize +" " + servings + " " + calories + "";
	}
	
	
}

```

## 빌더 패턴을 통해 객체를 만들고 실행했을때
```java
public class Test01 {

	public static void main(String[] args) {
		
		//lombok라이브러리에서 제공하는 builder처럼 동작함을 확인할 수 있다.
		
		System.out.print(
		new NutritionFacts.Builder(100, 50)
		.caloried(25)
		.carbohydrate(30)
		.build().toString());

	}

}
```

# 코멘트 추가 답변

```java

import java.util.EnumSet;
import java.util.Objects;
import java.util.Set;

public abstract class Pizza {

	public enum Topping { HAM, MUSHROOM, ONION, PEPPER, SAUSAGE }
	final Set<Topping> toppings;
	
  /*
  타입 파라미터는 기본적으로 불특정 타입을 받아 처리하기 위해 사용합니다.
  
  하지만 때로는 제네릭으로 처리해야할 타입을 제한해야할때도 있을 것입니다.
  이런 경우에는 제약사항을 추가한 경계가 있는 타입 파라미터(Bounded Type Parameter)를 사용합니다.
  
  
  extends Builder<T>를 통해 제약사항으로 제네릭 타입으로 원본 추상클래스나 이를 구현한 자식 클래스만 들어올 수 있도록 제한했습니다.   
  덕분에 추상클래스 Pizza.Builder 를 구현한 Nypizza와 Calzone 클래스는 제네릭 타입으로 들어올 수 있지만
  구현하지 않은 클래스는 타입 파라미터로 지정할 수 없어 안정성을 확보할 수 있습니다.
  
  */
	abstract static class Builder<T extends Builder<T>> {
		EnumSet<Topping> toppings = EnumSet.noneOf(Topping.class);
		
		public T addTopping(Topping topping) {
			toppings.add(Objects.requireNonNull(topping));
			return self();
		}
		
		abstract Pizza build();
		
		protected abstract T self();
	}
	
	Pizza(Builder<?> builder) {
		toppings = builder.toppings.clone();
	}
}

```

