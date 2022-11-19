

## 가변인수

- 메서드에 넘기는 인수의 개수를 클라이언트가 조절
- 가변인수 메서드를 호출하면 가변인수를 담기 위한 **배열**이 자동으로 하나 만들어짐
- 이 배열을 클라이언트에 노출하는 문제가 생김

## 실체화불가 타입, 실체화 타입

실체화 타입: 런타임에 타입 정보가 있는 것
실체화불가 타입: 런타임에 타입 정보가 없는 것

## 제네릭 매개변수 
![IMG_46F9DA753C45-1](https://user-images.githubusercontent.com/59721293/202831031-c7cde322-b830-476b-ad49-bd1e126bbb3b.jpeg)
![IMG_ADB6F81873BC-1](https://user-images.githubusercontent.com/59721293/202831038-479c36cb-3b8e-4c84-a4f5-dce1e0a0724d.jpeg)




- T... 로 받을 때 내부적으로 가변인수를 받기 위하여Object[] 를 생성한다
- 위에 코드도 toArray에서 Object[] 를 생성
- toArrasy는 늘 Object[] 를 반환
	- 런타임때 어떤 타입이 들어올지 모르고, 특정 타입으로 정한다는 것은 제네릭이 아니다
	- 떄문에 Object로 반환하는 것
![IMG_8E2407F65CC0-1](https://user-images.githubusercontent.com/59721293/202831032-c38c4bed-b7f2-4d51-ae8c-09a1fa123389.jpeg)
- 위에 코드를 실행하면?
	- ClassCastException 발생 why?
	- pickTwo의 반환값을 attributes 에 저장하기 위하여 컴파일러가 String[] 으로 형변환한다는 것을 놓쳤기 때문이다.
	- Onjrvy[]는 String[]의 하위 타입이 아니므로 이 형변환은 실패한다
		- 의문점: String str = (String) obj; 되지 않나?
- 그런데 만약에 안전하게 접근하여 사용한다고 개발자가 가정한다면 @SafeVarargs 를 사용하면 된다
	- 코드 예시
![IMG_B872B8A8EED6-1](https://user-images.githubusercontent.com/59721293/202831082-88f600a0-126b-4cc0-a360-3a4f43e87596.jpeg)
	  - @SafeVarags 사용은 그냥 모든 varargs 타입파라미터를 가질때 쓰면된다
	  -  이 말은 곧 varargs 를 쓸 때 절대 안전하지않을때 사용하면 안된다는 것이다
	  - 참고: 재정의할수없는 메서드에만! 
