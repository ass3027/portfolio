# Spring 트랜잭션 기초

[회사별 답변 Q03](interview-answers-artificial-society.md#q03) | [예상 질문](interview-questions-artificial-society.md)

일반적인 JDBC 또는 JPA 기반 서비스와 Spring의 기본 프록시 방식을 기준으로 설명한다. 아래 상황은 학습용 예시이며 실무 경험으로 주장하지 않는다.

## 1. 트랜잭션과 범위

트랜잭션은 여러 DB 변경을 하나의 작업으로 묶는 단위다. 커밋은 변경 확정, 롤백은 해당 트랜잭션의 변경 취소다.

과제 제출 내용과 제출 이력을 반드시 함께 저장해야 한다면 두 저장을 같은 트랜잭션에 넣는다. 이력 저장에 실패했을 때 제출 내용만 남는 상태를 막기 위한 선택이다. 범위는 메서드 길이보다 함께 성공하거나 실패해야 하는 데이터 변경을 기준으로 정한다.

`@Transactional`은 Spring에 이 메서드의 트랜잭션 처리를 맡기는 선언이다. 기본 전파 방식인 `REQUIRED`는 기존 트랜잭션이 있으면 참여하고, 없으면 새로 시작한다. 기존 트랜잭션에 참여했다면 내부 메서드 종료가 곧 최종 커밋은 아니다. [Spring 전파 방식 문서](https://docs.spring.io/spring-framework/reference/data-access/transaction/declarative/tx-propagation.html)

## 2. 프록시와 같은 클래스 내부 호출

프록시는 실제 서비스 호출 앞뒤에 트랜잭션 처리를 붙이는 대리 객체다.

```text
외부 호출 -> Spring 프록시 -> 실제 서비스 메서드
                            트랜잭션 시작 또는 기존 트랜잭션 참여
```

같은 객체 안에서 다른 메서드를 직접 호출하면 그 호출은 프록시를 거치지 않는다. 따라서 내부 메서드의 `@Transactional` 설정이 적용되지 않는다. [Spring 어노테이션 문서](https://docs.spring.io/spring-framework/reference/data-access/transaction/declarative/annotations.html)

```java
@Service
public class SubmissionService {
    public void submit() {
        saveSubmission(); // 같은 객체 내부 호출
    }

    @Transactional
    public void saveSubmission() {
        // 제출 내용과 제출 이력 저장
    }
}
```

- **예시 조건:** 호출자의 기존 트랜잭션 없음. 클래스 단위 트랜잭션 선언 없음.
- **결과:** `saveSubmission()`의 선언으로 두 저장 전체를 묶는 서비스 트랜잭션 생성 불가.
- **해결 방향:** 전체 저장을 묶을 `submit()`에 선언하거나, 저장 책임을 별도 Spring 빈으로 분리해 주입된 빈 호출.
- **구분:** 외부 메서드에서 시작한 트랜잭션이 이미 있다면 내부 DB 작업도 그 트랜잭션에 참여 가능. 내부 어노테이션 설정 무시와 트랜잭션 부재는 별개.

## 3. 예외와 롤백

기본 규칙에서는 `RuntimeException`과 `Error`가 트랜잭션 경계 밖으로 전파되면 롤백 대상이다. 체크 예외는 기본적으로 롤백 대상이 아니며, 필요한 예외를 `rollbackFor`로 지정할 수 있다. 프로젝트에서 기본 규칙을 변경했는지도 확인한다. [Spring 롤백 문서](https://docs.spring.io/spring-framework/reference/data-access/transaction/declarative/rolling-back.html)

- **체크 예외 예시:** `IOException`.
- **런타임 예외 예시:** `IllegalArgumentException`.
- **예외 처리 주의:** 예외를 내부에서 잡고 정상 반환하면 프록시가 그 예외를 통한 롤백 판단을 할 수 없음.
- **예외 처리의 예외:** 참여한 다른 트랜잭션 메서드 등이 이미 롤백 전용 상태로 표시했다면 정상 반환해도 커밋 불가.

## 4. 트랜잭션 안의 외부 API 호출

DB 처리 후 외부 API 응답을 오래 기다리면 DB 연결과 이미 획득한 락을 오래 점유할 수 있다. 또 일반적인 로컬 DB 트랜잭션의 롤백은 외부 API가 완료한 작업을 취소하지 않는다. 원격 호출까지 같은 트랜잭션이라고 가정하지 않는다. [Spring 선언적 트랜잭션 문서](https://docs.spring.io/spring-framework/reference/data-access/transaction/declarative.html)

교육 서비스의 제출과 AI 피드백을 분리한다면 다음 흐름을 고려할 수 있다.

```text
트랜잭션 A: 제출 내용과 처리 대기 기록 저장 후 커밋
외부 처리: AI API 호출
트랜잭션 B: 피드백 또는 실패 상태 저장 후 커밋
```

이 흐름은 제출 보존을 우선하는 설계 예시다. 실제 구현에서는 대기 작업 유실 방지, 재시도, 중복 처리 방지까지 함께 설계한다. API를 트랜잭션 밖으로 옮기는 것만으로 전체 작업의 정합성이 완성되지는 않는다.
