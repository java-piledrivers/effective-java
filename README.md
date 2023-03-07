# 이펙티브 자바 3판 스터디

이 스터디는 [java-squid/effective-java](https://github.com/java-squid/effective-java) 를 기반하여 진행하였습니다.
[👉스터디 회고](https://github.com/java-piledrivers/effective-java/issues/61)

## 목표

- 혼자 독학하기 힘든 이펙티브 자바를 같이 공부한다
- 이해되지 않는 문장이 있다면, 그대로 넘기지 않는다.
- 이 스터디를 통해 java의 전반적인 부분에 대해 이해를 한다.

## 기간

- 5아이템 X 18주
- 2022년 10월 8일 ~ 2023년 2월 18일
- 매주 토요일 (10:00 ~ 11:00) 화상으로 진행
- 첫번째 모임 시작일 2022.10.08 (토)

## 스터디 방식

1. 스터디 진행 방식은 매주 회고를 통해 조금씩 개선하는 방식을 택한다.
    - 예시 : 주간 회고 -> 아이템 공유 -> 스터디에 대한 의견 조율
    - 회고는 일주일간 진행하면서, 아쉬웠던 부분 혹은 어떻게 개선해야 겠다는 생각, 그 주 자신과 관련된 이슈 등을 말하도록 한다.

2. 매주 아이템 5개씩, 담당 아이템이 하나 존재
    - 해당 Item 자체 내용이 적거나, 혹은 너무 어렵다면 item 할당에 대해 스터디원간 합의하여 유동적으로 조율한다
    - 1사람-1아이템해서 돌아간다

3. 이슈 생성 담당자(kyupid)는 repo에 issue를 생성함
    - 이슈 생성 시, 순서에 맞춰 생성한다.
    - Assignees 를 통해 아이템 담당자를 구별한다.
    - 생성된 issue는 팀원들 간에 소통의 공간이다.
    - 팀원들은, 한 주간 정한 item들을 공부하면서 생겼던 질문들을 적합한 issue에 남기고, 해당 item 담당자는 질문에 대한 대답을 남긴다.
        - 대답은, 간단한 줄글 부터 예시 코드까지 작성할 수 있음.
        - 최대한 답변자가 이해할 수 있게 쉽게 설명.
    - 담당자가 해결하지 못하는 질문 또는 내용은 매주 **토요일 미팅** 시간에 공유하여, 해결하도록 한다.
       - 해결되지 못한 item 넘어가자 --> 너무 완벽하게, 깊게 파고들려고 하지 말자.
       - 해당 질문이 간단해서, 검색을 통해 해결될 수 있는 문제라면 조사 후, **issue 댓글로 정리 내용을 공유**하기로 한다.
    - 또한, 공유하고 싶은 문서등을 자신의 담당이 아닌 댓글에 추가할 수 있다.
 
4. Item 담당자는 item에 대한 정리(질답 내용을 포함) 하여 markdown file로 commit 한다.
    - 커밋 메세지는, 아이템명과 동일하다 
        - 메세지 예시 --> `[아이템 08] finalizer와 cleaner 사용을 피하라`
    - 저장 방식(예시)
        - effective-java (repo)
            - chapter2
                - item01.md
    - 해당 commit은 가능한 그 주내에 할 수 있도록 한다.
    - README에 item 정리한다 (스터디 끝나고 진행할 것)
    - branch는 모두 main으로 진행
        - 커밋 title, 은 issue 이름과 동일하게 한다.

5. item 정리 방식
    - 개개인 담당자에게 맡기지만, 남들이 봤을 때도(초급자가 봤을 때도) 이해할 수 있을 정도로 정리.

6. 위 스터디 방식은 팀원과의 합의를 통해, 언제든지 수정 가능하다.

## Ground Rule

- 예치금 30,000원
- 벌금
    - 매주 토요일 스터디 미참가 : 5000원
        - 당일만 제외하고, slack 남길것
        - 당일이어도, 누구나 참작가능하다고 생각하는 이유면 가능함.
            - ex) 경조사, 질병, 약속...
    - 담당부분에 대한 어떤 형태로든 응답이 전혀 없을시 : 5000원

- 다음주 진행자 역할
    - 아이템 issue 만들기 
    - 다음주 스터디를 진행하기 (진행 방식은 각자 재량)
    - 스터디 회고록

## 대화 수칙

- 우린 '근거'를 통해 피드백하며 '무분별한 비난'은 하지 않습니다. 
- 우린 '경청'을 노력하며, '피드백'을 환영합니다. 
- 우린 누구나 자유롭게 의견을 낼 수 있지만 '나만 정답이다'라는 태도를 지양합니다. 


## Reference
- [java-squid/effective-java](https://github.com/java-squid/effective-java)
