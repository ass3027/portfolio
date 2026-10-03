# Spring 트랜잭션 설계와 동작

[회사별 답변 Q03](interview-answers-artificial-society.md#q03) | [예상 질문](interview-questions-artificial-society.md)

일반적인 JDBC 또는 JPA 기반 서비스와 Spring의 기본 프록시 방식을 기준으로 설명한다. 아래 상황은 학습용 예시이며 실무 경험으로 주장하지 않는다.

## 면접 질문의 설계 기준

트랜잭션의 동작 원리를 나열하는 것만으로 범위 설계 질문에 답할 수는 없다. 무엇을 함께 확정할지와 분리한 작업의 실패를 어떻게 처리할지 설명해야 한다.

| 판단 기준 | 확인할 내용 |
| --- | --- |
| 업무 규칙 | 작업 완료 시 반드시 지켜야 할 상태, 허용할 수 없는 부분 저장 |
| 원자적 저장 범위 | 같은 DB에서 함께 확정할 변경, 변경 판단에 필요한 상태 조회 |
| 자원 점유 | DB와 무관한 형식 검증, 장시간 계산, 외부 API 대기의 분리 가능성 |
| 실패 정책 | 전체 취소 대상, 보존 대상, 별도 재시도 대상 |
| 구현 검증 | 프록시 호출 경로, 전파와 롤백 규칙, 실제 커밋 경계 |

여러 Repository를 사용하는 업무는 서비스 계층에서 하나의 경계로 묶는 것을 출발점으로 삼는다. 기존 트랜잭션에 참여하는 메서드는 자신이 반환됐다고 커밋되지 않으므로 호출 흐름 전체를 확인한다. [Spring 전파 방식 문서](https://docs.spring.io/spring-framework/reference/data-access/transaction/declarative/tx-propagation.html)

DB 상태를 읽고 판단한 뒤 변경하는 작업은 조회와 변경의 관계까지 검토한다. 단순히 같은 트랜잭션에 넣는 것만으로 다른 요청과의 경쟁이 해결되지는 않는다. 업무 규칙에 맞는 격리 수준과 락, 버전 검증 또는 DB 제약을 선택한다. [MySQL 잠금 조회 문서](https://dev.mysql.com/doc/refman/8.4/en/innodb-locking-reads.html)

외부 처리를 분리하는 기준은 지연뿐 아니라 실패의 의미다. AI가 실패해도 제출을 보존해야 한다면 제출과 AI 결과를 같은 성공 조건으로 묶지 않는다. 대신 제출과 처리 대기 기록을 같은 DB 트랜잭션에 저장하고, 후속 처리에서 대기 기록을 읽어 실행하는 흐름을 고려한다.

커밋 후 메모리에서 호출이나 이벤트 발행만 수행하면 프로세스 종료로 작업을 놓칠 수 있다. 외부 이벤트 전달의 유실 방지가 필요하다면 변경 데이터와 Outbox 기록을 같은 트랜잭션에 저장하는 방식을 검토한다. 중복 전달에 대한 멱등 처리도 필요하다. [AWS Transactional Outbox 설명](https://docs.aws.amazon.com/prescriptive-guidance/latest/cloud-design-patterns/transactional-outbox.html)

정합성 경계를 먼저 정하고, 그 경계를 유지하면서 자원 점유 시간을 줄인다. 트랜잭션을 짧게 만들기 위해 함께 확정해야 하는 데이터를 쪼개거나, 작은 작업마다 `REQUIRES_NEW`를 붙이는 것을 기본 해법으로 삼지 않는다.

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

외부 API 실패 시 앞선 DB 변경까지 취소해야 한다면, DB 트랜잭션을 유지한 채 외부 호출 결과를 확인하는 설계가 가능하다. 외부 실패 응답을 정상 반환하면 롤백이 자동으로 발생하지 않으므로 실패를 롤백 규칙에 맞게 처리한다. 이때 자원 점유 비용과 네트워크 타임아웃을 함께 검토한다.

```text
DB 트랜잭션 시작 -> DB 변경 -> 외부 API 호출
성공 응답: DB 커밋 시도
실패 판단: 해당 DB 트랜잭션 롤백
```

| 상황 | 로컬 DB 트랜잭션만으로 가능한 범위 |
| --- | --- |
| 외부 실패 확인, DB 미커밋 | 앞선 DB 변경 롤백 |
| 외부 성공, 이후 DB 커밋 실패 | 완료된 외부 작업 취소 보장 불가. DB 커밋 결과 불명 시 DB 상태 확인 필요 |
| 외부 호출 타임아웃 | DB 롤백 가능. 외부 처리 성공 여부 미확정 |

따라서 호출을 트랜잭션 안에 넣어도 DB와 외부 시스템의 원자적 완료가 보장되지는 않는다. 외부 상태 조회와 멱등성 키, 외부 시스템이 제공하는 취소 또는 환불 같은 보상 동작을 검토한다. 보상은 이미 커밋한 작업을 다른 작업으로 되돌리는 절차이며 DB 롤백과 다르다. 보상 자체의 실패와 재시도도 고려한다. [AWS Saga 설명](https://docs.aws.amazon.com/prescriptive-guidance/latest/cloud-design-patterns/saga-orchestration.html)

호출이 선행 DB 변경이나 잠금에 의존하지 않는다면 외부 API를 먼저 수행하고 성공 후 짧은 DB 트랜잭션을 여는 대안도 있다. 이 순서 역시 외부 성공 뒤 DB 실패 문제를 해결하지는 않는다. 업무 의존 관계와 허용할 수 있는 실패 상태로 선택한다.

교육 서비스의 제출과 AI 피드백을 분리한다면 다음 흐름을 고려할 수 있다.

```text
트랜잭션 A: 제출 내용과 처리 대기 기록 저장 후 커밋
외부 처리: AI API 호출
트랜잭션 B: 피드백 또는 실패 상태 저장 후 커밋
```

이 흐름은 제출 보존을 우선하는 설계 예시다. 실제 구현에서는 대기 작업 유실 방지, 재시도, 중복 처리 방지까지 함께 설계한다. API를 트랜잭션 밖으로 옮기는 것만으로 전체 작업의 정합성이 완성되지는 않는다.
