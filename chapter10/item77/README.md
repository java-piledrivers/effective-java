```java
// catch에서 에러를 무시함
try{
  
}catch(SomeException e){

}
```
- catch 블럭을 비워두면 예외의 존재 이유가 없어진다
- 만약 예외를 무시하기로 결정했으면 이유를 주석으로 남기고 예외 변수 이름도 ignored로 바꿔놓도록 하자.

```java

try{
  
}catch(SomeException ignored){
  // ignore, ......
}
```
