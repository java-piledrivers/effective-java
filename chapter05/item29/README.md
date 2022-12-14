### 정리.

- 클라이언트에서 직접 형변환해야 하는 타입보다 제네릭 타입이 더 안전하고 쓰기 편하다.
    - 그러니, 새로운 타입을 설계할 땐 형변환 없이도 사용할 수 있도록 제네릭을 사용하자.
- 기존 타입 중 제네릭이 있어야 하다면, 제네릭 타입으로 변경하자.
    - 기존 클라이언트에게는 아무 영향을 주지 않으면서, 새로운 사용자를 훨씬 편하게 해주는 길이다.

---

Item 28 에서 - E와 같은 실체화 불가 타입으로는 배열을 만들 수 없다.

### 실체화 불가 타입을 제네릭으로 만들기  - 제네릭 + 배열 이슈

아래 코드 컴파일 에러 발생. 

![image](https://user-images.githubusercontent.com/46278436/201452582-3ddfc38f-660c-475a-ab45-1850014d0098.png)

E와 같은 실체화 불가 타입은 배열을 만들 수 없다.

(제네릭 배열 생성 금지 제약) 이와 같은 문제가 있을 경우 두 가지의 우회방법이 존재한다.


---

### **제네릭 배열 생성을 금지하는 제약을 우회하는 방법 두 가지.**

**[제네릭 배열 생성을 금지하는 제약을 우회하는 방법]**

**방법1** 

![image](https://user-images.githubusercontent.com/46278436/201452591-40031565-7589-4ddb-bc53-1fbcebb64c20.png)
- Object 배열로 생성하고 제네릭 배열 (E)로 캐스팅.

방법2. 

![image](https://user-images.githubusercontent.com/46278436/201452598-0c7fd949-e615-444e-9926-6b8a379d4341.png)
- Element 타입을 E[]에서 Object[]로 변경.

---

### 방법 1, 방법2 장단점

방법1 장점. 

- 가독성이 좋다.
    - 배열의 타입을 E[]로 선언하여 오직 E 타입 인스턴스만 받음을 확실히 어필한다.
- 캐스팅을 배열 생성 시 단 한번만 해주면 된다.

 but

- 배열의 런타임 타임이 컴파일 타임과 달라 힙 오염(item32)을 일으킨다.

방법2 장점

- 힙 오염이 발생하지 않아 이를 더 선호하는 프로그래머들이 있음.

but

- 배열을 원소를 읽을 때마다 형변환 필요.

---

### 제네릭 배열을 사용하는 이유

- 자바가 리스트를 사용하는게 항상 가능하지도, 꼭 더 좋은 것도 아니다. 
리스트도 결국 기본 타입인 배열을 사용해 구현해야 하기 떄문이다.
- 성능향상의 이슈로 배열을 사용할 수 있다.
- 제네릭 타입매개변수에는 primitive 타입을 사용할 수 없다. 박싱된 기본타입으로 우회할 수 있긴하다.
