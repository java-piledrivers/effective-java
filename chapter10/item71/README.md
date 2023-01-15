# 아이템71 필요없는 검사 예외 사용은 피하라


## 검사 예외 

검사 예외는 처리하여 안전성을 높일 수 있기 때문에 잘 사용하면 API와 프로그램의 질을 높인다.

**검사 예외를 사용하는 경우**
1. API를 제대로 사용해도 발생할 수 있는 예외
2. 프로그래머가 의미 있는 조치를 취할 수 있는 경우 


## 검사 예외의 한계
- 검사 예외는 호출자가 처리해야하는 강제성을 지니기 때문에 부담을 주게 된다.
- 검사 예외를 던지는 메서드는 스트림 안에서 직접 사용할 수 없다.
- 특히 메서드가 단 하나의 검사 예외만 던질 때 부담이 크다.   
  그 예외 하나 때문에 API 사용자는 try블록을 추가하고 스트림에서 사용하지 못하게 된다. 이런 상황에선 검사예외를 안 던질 방법이 없는지 고민해보자

## 검사 예외 회피 방법

### 1. Optional 
- 적절한 결과 타입을 담은 옵셔널을 반환하는 것은 검사 예외를 회피하는 가장 쉬운 방법
- 검사 예외를 던지는 대신 단순히 **빈 옵셔널 반환**
- 단점 : 예외 발생 이유에 대한 부가정보를 담을 수 없다

- 검사 예외 대신 적절한 결과 타입을 전달하는 메서드 예시 
  - `sendMulticastAsync()`는 메서드 실행 성공여부를 `ApiFuture<BatchResponse>`에 담아서 보내준다.
  - `send()`는 메서드 실행 성공여부를 void 또는 예외를 던지는 것으로 보내준다. 그러므로 검사 예외를 발생하는 메서드인데 이러한 메서드를 직접 작성할 때 검사예외 대신 메서드 실행결과를 Optional로 전달 할 수 있을지 고민해보면 좋을 것 같다.

```java
    public ApiFuture<BatchResponse> sendGroupMessageAsync(
            List<String> tokenList, String title, String content, String imageUrl) {
        MulticastMessage multicast =
                MulticastMessage.builder()
                        .addAllTokens(tokenList)
                        .setNotification(
                                Notification.builder()
                                        .setTitle(title)
                                        .setBody(content)
                                        .setImage(imageUrl)
                                        .build())
                        .setApnsConfig(
                                ApnsConfig.builder()
                                        .setAps(Aps.builder().setSound("default").build())
                                        .build())
                        .build();

        return FirebaseMessaging.getInstance().sendMulticastAsync(multicast);
    }

    public void sendMessageSync(String token, String content) throws FirebaseMessagingException {
        Message message =
                Message.builder()
                        .setToken(token)
                        .setNotification(Notification.builder().setBody(content).build())
                        .setApnsConfig(
                                ApnsConfig.builder()
                                        .setAps(Aps.builder().setSound("default").build())
                                        .build())
                        .build();

        FirebaseMessaging.getInstance().send(message);
    }
```


### 2.검사 예외를 던지는 메서드를 2개로 쪼개 비검사 예외로 바꾸기
- 이 방식에서 첫 번째 메서드는 예외가 던져질지 여부를 boolean값으로 반환한다.
- 예시 코드 : 리팩토링 후가 더 유연한 코드   
```java
// 검사 예외를 던지는 메서드 - 리팩토링 전
try {
	obj.action(args);
} catch (TheCheckedException e) {
	... // 예외 처리
}

// 상태 검사 메서드와 비검사 예외를 던지는 메서드 - 리팩토링 후
// actionPermitted : 상태검사 메서드
if(obj.actionPermitted(args)){
	obj.action(args);
} else {
	... // 예외 처리
}
```

- 모든 상황에서 이 리팩터링을 할 수 있는 것은 아니다.
- `상태 검사 메서드`의 단점도 그대로 적용된다. (아이템 69)
- 외부 동기화 없이 여러 스레드가 동시에 접근할 수 있거나 **외부 요인에 의해 상태가 변할 수 있다면** 이 리팩터링은 **적절하지 않음**
- 상태 검사 메서드(actionPermitted)가 상태 의존적 메서드(action)의 작업 일부를 중복 수행한다면 성능 손해

## 결론
- 꼭 필요한 곳에만 사용한다면 검사 예외는 안전성을 높여주지만, 남용하면 사용자에게 큰 부담을 준다.
- API 호출자가 예외 상황에서 복구할 방법이 없다면 비검사 예외를 던지자.
- 복구가 가능하고 API 호출자가 그 처리를 하길 바란다면 옵셔널 반환을 먼저 고민해보자.
- 옵셔널만으로 상황 처리에 충분한 정보를 줄 수 없을 때만 검사 예외를 던지자.


