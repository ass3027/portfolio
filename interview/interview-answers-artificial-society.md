# 아티피셜소사이어티 백엔드 개발자 면접 답변

[예상 질문과 자기소개](interview-questions-artificial-society.md)

질문 문서의 번호에 맞춰 답변을 정리한다. 확인되지 않은 경험과 수치는 추가 확인 항목으로 구분한다.

## Q01

**가장 주도적으로 개발한 백엔드 프로젝트를 설명해 주세요.**

### 내 답변

가장 주도적으로 참여했던 프로젝트는 김포공항 Lidar 프로젝트 였습니다. 여객 흐름과 대기시간을 파악하는 프로젝트로, 후반에 들어와 프로그램 개발 및 오류 수정을 혼자 맡게 되었습니다. 또한 담당자와 소통/히스토리 파악/API 설계 변경들을 수행하였습니다. 익숙하지 않던C++/gRPC 중계서버 메모리 누수 디버깅도 하게 되었고, 통계 API 재설계를 통헤 대규모 데이터를 처리하면서 sql 최적화에 대해서도 더 알게 되었습니다.

## Q02

**실제로 성능 문제를 발견하고 개선한 경험이 있나요?**

### 내 답변
김포공항 LiDAR 프로젝트에서 통계 차트 화면 표출이 40초 정도 걸리는 문제가 있었습니다.
해당 문제를 개선하기 위해 실제로 어느부분에서 시간이 걸리는지 측정을 하였고, 원인으로는 API에 있던 불필요한 쿼리와 쿼리 자체의 속도가 주 원인이었습니다. API 재설계를 통해 불필요한 쿼리 호출과 응답 데이터를 제거하고, 느린 쿼리의 비교대상칼럼에 걸린 함수 제거와 인덱스를 조정하여 해결하였습니다. 복합인덱스 순서는 동등조건을 앞에, 범위조건을 뒤에 배치하였습니다. explain 키워드를 통해 인덱스가 잘 타는지 검증하였고, 이를 통해 40초대의 화면 표출을 1.5초대로 개선할 수 있었습니다.

## Q03

**여러 DB 변경이 모두 성공하거나 모두 취소돼야 한다면, `@Transactional`을 어느 메서드에 붙이겠어요?**

### 내 답변
여러 DB 변경 즉 spring 기준 repository들을 호출 하는 메서드를 서비스 레이어에 위치시키고, 해당 메소드에 @Transactional을 달겠습니다.  만약 외부 API 호출도 작업 흐름에 포함되어있고, 외부 API 실패 시 DB 변경 사항도 롤백 되야 되는지 확인하고 불필요하다면 트랜잭션에서 제외 시켜 트랜잭션을 최대한 가볍게 하겠습니다.

## Q04

**여러 요청이 동시에 같은 데이터를 수정하는 상황을 어떻게 처리했나요?**

### 내 답변
ktx 프로젝트에서 동시 요청을 DB 낙관적 락과 입장 제어로 처리하였습니다. DB 낙관적 락을 jpa @version 을 이용해서 동시성을 보장하였습니다. 같은 좌석의 요청이 발생되면 DB까지 갔다 취소 되기에 처리량 하락이 발생하였고, 이를 줄이기 위해 앞단에 Redis를 이용한 동시성 제어를 추가했습니다. 좌석을 Set으로 관리하고 SREM을 성공했을 시에만 DB에 접근합니다. 이를 통해 처리량과 동시성을 향상 시켰고, k6를 통해 1000명이 경쟁하는 부하 상황을 만들고 초당 요청 처리량과 2번이상 처리된 좌석이 0임을 측정하였습니다.


### 핵심 사항

- **역할:** Redis 좌석 선점으로 DB 진입 요청 제한, DB `@Version`으로 변경 충돌 감지.
- **락 선택:** 선점 이후 낮은 DB 경합은 설계 가설. 낙관적 락이 비관적 락보다 빠르다는 비교 결과 없음.
- **Redisson:** 비교용 구현과 PoC 검증 경험 있음. 현재 Set은 선점, 자동 배정, 잔여석 조회에 함께 활용.
- **Redis와 DB:** 하나의 트랜잭션으로 묶이지 않으므로 DB 기준 보정 필요. JPA 충돌 시 DB 롤백과 Redis 복원은 별개.
- **검증:** 동일 좌석 1,000명 경합에서 성공 1건, 초과 판매와 중복 0건 확인. 요청 처리량과 예매 확정 건수 구분.
- **다중 서버:** JVM 내부 락의 적용 범위는 한 프로세스. DB 또는 공유 저장소에서 동시 접근 제어 필요.
- **중복 요청:** 요청 키와 DB 유니크 제약을 이용한 멱등 처리는 추가 설계안. 실제 구현 경험과 구분.

## Q05

**MySQL 쿼리가 느릴 때 어떤 순서로 분석하나요?**

### 내 답변
먼저 sql 일단 보고 불필요한 join, 비교대상칼럼에 함수가 걸려있는지 확인하고, 그 후 explain 키워드를 사용하여 key 선택된 인덱스, type으로 접근방식, rows로 예상 탐색행 수를 보고 index가 제대로 사용되고 있는지 확인할 것 같습니다. 확인한 결과를 기반으로 인덱스와 쿼리를 조정하고 쿼리 실행시간이 줄었는지 확인하겠습니다.



### 핵심 사항

- **실행계획:** key는 선택된 인덱스, rows는 예상 탐색 행 수. Extra에서 추가 정렬과 임시 테이블 사용 여부 확인.
- **접근 방식:** type의 ALL은 전체 테이블 스캔, range는 인덱스 범위 탐색, ref는 비고유 인덱스를 이용한 동등 조건 조회.
- **복합 인덱스:** 동등 조건을 앞에, 범위 조건을 뒤에 두는 순서를 후보로 검토. 정렬, 다른 조회 패턴, 데이터 분포도 고려.
- **인덱스 효과가 없는 원인:** 조건과 인덱스 불일치, 많은 탐색 행, 정렬과 조인 비용, 락 대기와 I/O 등 확인.
- **검증:** 같은 데이터와 조회 조건에서 결과 동일성과 실행 시간 비교. 캐시 상태와 반복 횟수 구분. 실행계획의 인덱스 사용만으로 성능 개선 단정 불가.
- **경험 범위:** LiDAR의 40초대에서 약 1.5초는 API와 SQL을 함께 개선한 화면 결과 표시 시간. SQL 단독 실행 시간과 구분.

## Q06

**AWS에서 서비스를 운영하면서 장애를 해결한 경험을 설명해 주세요.**

### 내 답변
Tag2now 프로젝트를 ECS에 배포 실패한 적이 있습니다. 로컬에서 redis에 잘연결되었고, parameter store의 변수도 잘 불러왔지만, 배포 시 redis 연결 시 connection timed out 이 발생하였습니다. 로그만으로는 원인 분석이 어려워 클라우드 환경에서 직접 확인을 위해, ecs에서 제공하는 container cmd 수정 기능을 통해 컨테이너가 죽지 않게 한뒤 직접 연결하여 디버깅을 하였습니다. linux 명령어들을 통해 DNS, tcp 연결 문제는 아니었단것을 확인했고, redis cli로 직접 연결을 시도한 결과 운영 ElastiCache는 TLS 연결이 필요하다는 것을 확인했습니다. Redis 연결 URL의 스킴을 rediss://로 지정하지 않아 TLS가 적용되지 않은 것이 원인이었고, 이를 rediss://로 수정하여 배포에 성공했습니다.

## Q07

**학교 수업 시작 시 학생들이 동시에 접속한다면 어떻게 대비하겠어요?**

### 내 답변
서버, DB의 메트릭을 수집하며 모니터링을 해두겠습니다.  기본적으로 인스턴스의 cpu, memory 점유율을 기준으로 수평 auto scaling을 이용해 유동적으로 자원량을 조절할 수 있도록 하겠습니다. 또한 학교 수업 시작 시 라면 특정 시간대에 트래픽이 몰릴 것으로 예상되기에 몰리는 시간대를 확인 후 미리 인스턴스 수를 늘리겠습니다. 만약 DB에 읽기 부하가 발생을 한다면 특정 시간 대 read replica도 고려할 것 같습니다.

## Q08

**기획자와 협의하면서 요구사항이나 구현 방향을 바꾼 경험이 있나요?**

기획자와 일해본적은 없습니다. 하지만 고객과 직접소통하며 요구사항들을 받고, 좀 더 나은 방향으로 개선한 점은 있습니다. LiDAR 프로젝트에서 공항 공식 페이지와 연계해야하는 작업이 있었습니다. 기존 구조에서는 제가 작업한 파일들을 공식페이지 담당자에게 전송하여 코드에 반영하는 구조로 작업되고 있었습니다. 하지만 이 경우 저희 프로젝트가 업데이트 될 때 마다 공식페이지 또한 업데이트 해야하는 불편함이 존재했습니다. 제가 공식 페이지 담당자에게 저희 프로젝트 api로 부터 데이터를 받아 화면에 뿌려주는 방식으로 제안하였습니다. 이를 통해 불편했던 기존 작업 방식을 바꾸고 유지보수성을 향상시켰습니다

### 내 답변


## Q09

**Spring MVC에서 요청이 들어와 응답이 나가기까지의 흐름을 설명해 주세요.**

[질문과 꼬리 질문](interview-questions-artificial-society.md#9-spring-mvc에서-요청이-들어와-응답이-나가기까지의-흐름을-설명해-주세요)

### 내 답변

### 모범 답변

일반적인 동기 요청을 기준으로, 서블릿 컨테이너가 요청을 받고 Filter 체인을 거쳐 DispatcherServlet으로 전달합니다. DispatcherServlet은 HandlerMapping으로 요청을 처리할 Controller와 Interceptor를 찾습니다. Interceptor의 preHandle을 통과하면 HandlerAdapter가 요청 파라미터와 본문을 메서드 인자로 변환하고, 필요한 검증을 수행한 뒤 Controller를 호출합니다.

Controller는 Service와 Repository를 통해 작업을 수행합니다. REST API라면 반환값을 HttpMessageConverter로 JSON 등의 응답 본문으로 변환하고, Interceptor와 Filter의 후처리를 거쳐 응답을 반환합니다. 화면을 반환하는 MVC라면 ViewResolver로 View를 찾아 렌더링합니다.

Filter는 DispatcherServlet 바깥에서 인코딩이나 보안 같은 공통 처리를 담당하고, Interceptor는 선택된 Handler를 기준으로 호출 전후의 공통 처리를 담당합니다. Controller Advice는 요청이 항상 통과하는 단계가 아니라 여러 Controller에 적용할 예외 처리나 바인딩 설정 등을 모으는 기능입니다. 특히 @RestControllerAdvice의 @ExceptionHandler는 MVC의 예외 처리 과정에서 예외를 공통 오류 응답으로 변환합니다. Filter에서 발생한 예외까지 자동으로 처리하는 것은 아닙니다.

참고: [Spring MVC 처리 구조](https://docs.spring.io/spring-framework/reference/6.2/web/webmvc/mvc-servlet/special-bean-types.html), [응답 본문 변환](https://docs.spring.io/spring-framework/reference/web/webmvc/mvc-controller/ann-methods/responsebody.html), [예외 처리](https://docs.spring.io/spring-framework/reference/web/webmvc/mvc-controller/ann-exceptionhandler.html)

## Q10

**Spring Bean은 기본적으로 싱글턴인데, 요청마다 달라지는 값을 필드에 저장하면 어떤 문제가 생기나요?**

싱글턴의 경우 여러 요청이 한 인스턴스를 공유합니다. 따라서 Bean에 동시에 요청이나 호출이 들어올 경우 필드값에 동시성문제가 발생 할 수 있습니다. A요청의 응답값이 동시에 들어온 B 요청의 결과값이 될 수 있습니다
[질문과 꼬리 질문](interview-questions-artificial-society.md#10-spring-bean은-기본적으로-싱글턴인데-요청마다-달라지는-값을-필드에-저장하면-어떤-문제가-생기나요)

참고: [Spring Bean Scope](https://docs.spring.io/spring-framework/reference/core/beans/factory-scopes.html)

## Q11

### 내 답변
**Java의 `equals()`와 `hashCode()`는 어떤 관계인가요?**
equals 가 참인 2 객체는 같은 hashCode를 반환해야합니다. 역은 성립할 필요는 없습니다. 이게 지켜지지 않으면 HashSet, HashMap등 hashCode()를 조회/저장 위치로 사용하는 자료구조에서 중복 제거나, 객체 조회가 제대로 되지 않을 수 있습니다.

[질문과 꼬리 질문](interview-questions-artificial-society.md#11-java의-equals와-hashcode는-어떤-관계인가요)


참고: [Java Object의 equals와 hashCode 계약](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/Object.html)

## Q12

**JPA를 사용한다면 N+1 문제를 어떻게 발견하고 해결하나요?**

[질문과 꼬리 질문](interview-questions-artificial-society.md#12-jpa를-사용한다면-n1-문제를-어떻게-발견하고-해결하나요)

### 내 답변

### 모범 답변

N+1은 목록 조회 후 각 항목의 연관 데이터를 가져오면서 추가 쿼리가 반복되는 문제입니다. SQL 로그로 반복되는 쿼리와 한 요청의 쿼리 수를 확인하겠습니다.

필요한 연관 데이터를 Fetch Join으로 함께 조회하고, 적용 전후의 쿼리 수와 응답 시간을 비교하겠습니다.

다만 컬렉션 Fetch Join에 페이징을 적용하면 메모리에서 페이징할 수 있어, 부모 목록은 DB에서 페이징하고 연관 데이터는 Batch Fetch로 묶어 조회하겠습니다. 여러 컬렉션의 동시 Fetch Join도 결과 행이 곱으로 늘어날 수 있어 피하겠습니다.

참고: [Hibernate Fetch Join과 페이징 제약](https://docs.jboss.org/hibernate/orm/7.0/querylanguage/html_single/Hibernate_Query_Language.html), [Hibernate 조회 전략과 Batch Fetch](https://docs.hibernate.org/orm/6.6/userguide/html_single/)

## Q13

**API의 검증 오류와 서버 내부 오류를 어떻게 구분하고 응답하나요?**

[질문과 꼬리 질문](interview-questions-artificial-society.md#13-api의-검증-오류와-서버-내부-오류를-어떻게-구분하고-응답하나요)

### 내 답변

### 모범 답변

필수 값 누락이나 형식 오류는 Bean Validation으로 검증해 400으로 응답하고, 예상하지 못한 서버 내부 오류는 500으로 구분하겠습니다.

@RestControllerAdvice로 응답 형식을 통일하고, 오류 코드와 메시지, 추적 ID를 반환하겠습니다. 검증 오류에는 필드와 실패 이유도 넣어 클라이언트가 해당 입력란에 표시할 수 있게 하겠습니다. 클라이언트는 고정된 오류 코드로 처리하고, 내부 예외의 상세 내용은 응답 대신 추적 ID와 함께 서버 로그에 남기겠습니다.

참고: [Spring MVC 공통 예외 처리](https://docs.spring.io/spring-framework/reference/web/webmvc/mvc-controller/ann-exceptionhandler.html)

## Q14

**배포 후 특정 API의 응답 시간이 갑자기 증가했다면 무엇부터 확인하나요?**

[질문과 꼬리 질문](interview-questions-artificial-society.md#14-배포-후-특정-api의-응답-시간이-갑자기-증가했다면-무엇부터-확인하나요)

### 내 답변

### 모범 답변

먼저 배포 전후 해당 API의 응답 시간과 오류율, 요청량을 비교하고 변경된 코드와 설정을 확인하겠습니다. 영향이 크고 배포가 원인으로 의심되면 호환성을 확인한 뒤 롤백으로 복구를 우선하겠습니다.

이후 구간별 시간 측정으로 지연 위치를 좁히겠습니다. 스레드 풀은 사용량과 스레드 덤프로 대기 원인을, DB는 커넥션 획득 대기와 SQL 실행 시간을 구분해 확인하겠습니다. GC 중단 시간과 외부 API 호출 시간도 지연 시점과 비교하겠습니다. 원인 수정 후 같은 요청 조건에서 응답 시간이 회복됐는지 확인하겠습니다.

참고: [Spring Boot의 HTTP, JVM, DB 연결 메트릭](https://docs.spring.io/spring-boot/reference/actuator/metrics.html), [HikariCP 연결 풀 설정](https://github.com/brettwooldridge/HikariCP#configuration-knobs-baby)

## Q15

**학생·교사·학교·학급·과제·제출 이력을 저장하는 DB를 설계해 주세요.**

### 내 답변


## Q16

**학생이 제출 버튼을 여러 번 눌러도 결과가 중복 저장되지 않도록 어떻게 설계하나요?**

[질문과 꼬리 질문](interview-questions-artificial-society.md#16-학생이-제출-버튼을-여러-번-눌러도-결과가-중복-저장되지-않도록-어떻게-설계하나요)

### 내 답변

### 모범 답변

같은 제출 시도와 재시도에는 동일한 멱등 키를 보내도록 하고, 사용자와 요청 키의 조합에 DB 유니크 제약을 두겠습니다. 제출 내용과 처리 결과는 같은 트랜잭션으로 저장해, 응답을 받지 못해 재시도하더라도 기존 결과를 반환하겠습니다. 같은 키로 다른 내용을 보내면 거절하고, 처리 중이면 진행 상태를 안내하겠습니다. 과제당 한 번만 제출할 수 있는 정책이라면 학생과 과제의 조합에도 유니크 제약을 두겠습니다.

참고: [멱등 API와 재시도](https://aws.amazon.com/builders-library/making-retries-safe-with-idempotent-APIs/), [MySQL 유니크 제약](https://dev.mysql.com/doc/refman/8.4/en/constraint-primary-key.html)

## Q17

**교사용 학습 현황 화면에서 통계 조회가 느려지면 어떻게 개선하겠어요?**

[질문과 꼬리 질문](interview-questions-artificial-society.md#17-교사용-학습-현황-화면에서-통계-조회가-느려지면-어떻게-개선하겠어요)

### 내 답변
느린 API가 정확히 뭔지 브라우저에서 확인 후, 해당 API가 어떤 부분에서 느린지 확인하겠습니다. 내부 로직에서 시간복잡도 관련 문제라면 자료구조와 로직 개선을 통해 개선하고, 쿼리문제라면 sql 점검, explain을 활용한 인덱스 사용 상태를 확인 후 개선하겠습니다.

### 모범 답변

먼저 API 내부 처리와 SQL 실행 시간을 나눠 측정하고, 쿼리가 느리면 EXPLAIN으로 조회 범위와 인덱스 사용을 확인하겠습니다. 쿼리 개선 후에도 반복 집계 비용이 크면 학급이나 날짜별로 사전 집계하겠습니다. 방금 제출한 결과처럼 최신성이 중요한 값은 실시간으로 조회하고, 추세 통계는 허용 가능한 지연에 맞춰 주기적으로 갱신하겠습니다. 화면에는 집계 기준 시각을 표시하고, 응답 시간과 원본 대비 집계 결과의 정확성을 함께 확인하겠습니다.

참고: [MySQL 실행계획 분석](https://dev.mysql.com/doc/refman/8.4/en/using-explain.html)

## Q18

**서비스 운영 중 큰 테이블의 스키마를 변경해야 한다면 어떻게 진행하나요?**

[질문과 꼬리 질문](interview-questions-artificial-society.md#18-서비스-운영-중-큰-테이블의-스키마를-변경해야-한다면-어떻게-진행하나요)

### 내 답변

### 모범 답변

먼저 운영 DB 버전에서 해당 변경의 Online DDL 지원 여부와 락, 작업 시간을 사전 검증하겠습니다. 컬럼을 교체한다면 기존 컬럼을 유지한 채 새 컬럼을 추가하고, 새 버전은 양쪽에 값을 기록하면서 구버전과 호환되도록 하겠습니다. 기존 데이터는 작은 배치로 보정하고, 구버전 종료 후 추가 변경분까지 맞춰 누락과 불일치를 검증하겠습니다. 새 컬럼으로 읽기를 전환하고 안정화한 뒤, 별도 배포에서 기존 컬럼을 제거하겠습니다.

참고: [MySQL Online DDL](https://dev.mysql.com/doc/refman/8.4/en/innodb-online-ddl-operations.html)

## Q19

**REST API에서 리소스, HTTP 메서드, 상태 코드를 어떻게 결정하나요?**

[질문과 꼬리 질문](interview-questions-artificial-society.md#19-rest-api에서-리소스-http-메서드-상태-코드를-어떻게-결정하나요)

### 내 답변

### 모범 답변

리소스는 명사로 표현하고, 조회는 GET, 생성은 POST처럼 요청의 의미에 맞는 메서드를 선택하겠습니다. 과제 제출은 POST /assignments/{id}/submissions로 처리하고 생성되면 201을 반환하겠습니다. 채점은 POST /submissions/{id}/grading-jobs로 요청받아 비동기 처리라면 작업 ID와 함께 202를 반환하고, GET /grading-jobs/{id}로 상태와 결과를 조회하게 하겠습니다. 오류는 잘못된 입력 400, 없는 리소스 404, 현재 상태와의 충돌 409처럼 의미에 맞게 구분하겠습니다.

참고: [HTTP 메서드와 상태 코드](https://www.rfc-editor.org/rfc/rfc9110.html)

## Q20

**EKS에 Spring Boot 서비스를 배포한다면 어떤 구성 요소가 필요한가요?**

[질문과 꼬리 질문](interview-questions-artificial-society.md#20-eks에-spring-boot-서비스를-배포한다면-어떤-구성-요소가-필요한가요)

### 내 답변

### 모범 답변

Spring Boot 이미지를 ECR에 올리고, Deployment로 Pod 수와 롤링 업데이트를 관리하겠습니다. Service는 바뀌는 Pod 주소 대신 안정적인 접근 지점을 제공하고, Ingress는 외부 HTTP 요청을 도메인이나 경로에 따라 Service로 연결합니다. EKS에서는 AWS Load Balancer Controller를 설치해 Ingress에 맞는 ALB를 구성할 수 있습니다. 여기에 설정과 비밀 값의 관리, AWS 접근 권한, 리소스 제한, 상태 점검과 로그 수집을 함께 구성하겠습니다.

참고: [Kubernetes Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/), [Service](https://kubernetes.io/docs/concepts/services-networking/service/), [EKS Load Balancer Controller](https://docs.aws.amazon.com/eks/latest/userguide/aws-load-balancer-controller.html)

## Q21

**Readiness Probe와 Liveness Probe는 어떻게 다르게 설정하나요?**

[질문과 꼬리 질문](interview-questions-artificial-society.md#21-readiness-probe와-liveness-probe는-어떻게-다르게-설정하나요)

### 내 답변

### 모범 답변

Readiness는 현재 요청을 받을 수 있는지 확인해 실패한 Pod를 트래픽 대상에서 제외하고, Liveness는 재시작으로 회복할 수 있는 애플리케이션 이상을 확인하겠습니다. DB가 필수인 서비스라면 DB 장애를 Readiness에 반영할 수 있지만, Liveness가 DB 연결 실패만으로 실패하도록 만들지는 않겠습니다. DB 장애 때 모든 Pod가 재시작되면 연결 재시도가 몰리고 복구도 늦어질 수 있기 때문입니다. 시작 시간이 긴 서비스는 Startup Probe로 초기 기동 시간을 별도로 보장하겠습니다.

참고: [Kubernetes 상태 점검](https://kubernetes.io/docs/concepts/workloads/pods/probes/)

## Q22

**무중단 배포 중 처리하던 요청이 끊기지 않도록 어떻게 하겠어요?**

[질문과 꼬리 질문](interview-questions-artificial-society.md#22-무중단-배포-중-처리하던-요청이-끊기지-않도록-어떻게-하겠어요)

### 내 답변

### 모범 답변

새 Pod가 Ready 상태가 된 뒤 기존 Pod를 종료하도록 롤링 업데이트를 설정하겠습니다. 종료되는 Pod로 신규 요청이 전달되지 않도록 라우팅 제외와 로드밸런서의 연결 종료 대기를 확인하겠습니다. 애플리케이션은 SIGTERM을 받으면 신규 요청 수락을 중단하고 진행 중인 요청을 마치는 Graceful Shutdown을 사용하겠습니다. 라우팅 변경의 전파 시간과 요청 처리 시간을 고려해 preStop과 terminationGracePeriodSeconds를 설정하고, 배포 중 요청이 끊기지 않는지 검증하겠습니다.

참고: [Pod 종료 과정](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/), [Spring Boot Graceful Shutdown](https://docs.spring.io/spring-boot/reference/web/graceful-shutdown.html)

## Q23

**서버 수를 늘렸는데 RDS 연결 오류가 증가했습니다. 원인은 무엇일까요?**

[질문과 꼬리 질문](interview-questions-artificial-society.md#23-서버-수를-늘렸는데-rds-연결-오류가-증가했습니다-원인은-무엇일까요)

### 내 답변
서버 수를 늘렸는데 RDS 연결오류가 발생한다면 max-connection이 원인으로 추정됩니다. 각 서버마다 connection pool size를 확인 한 후 최대 서버 수일 경우에도 max-connection-size를 안넘기게 조정하겠습니다.

### 모범 답변

먼저 오류 메시지와 RDS 연결 수를 확인해 max_connections 초과인지 판단하겠습니다. 서버마다 커넥션 풀이 있으므로 최대 Pod 수에 각 풀 크기를 곱하고, 다른 서비스와 관리용 연결도 합산하겠습니다. 롤링 배포 때 추가로 뜨는 Pod까지 포함해 DB 한도 안에 여유를 두도록 풀 크기를 정하겠습니다. 이 값은 연결 수의 상한이므로, 부하 테스트에서 커넥션 획득 대기와 DB 처리 여력을 보고 실제 크기를 조정하겠습니다.

참고: [RDS 최대 연결 수](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/CHAP_Limits.html#RDS_Limits.MaxConnections)

## Q24

**S3에 학생의 제출 파일을 저장한다면 업로드와 접근 권한을 어떻게 설계하나요?**

[질문과 꼬리 질문](interview-questions-artificial-society.md#24-s3에-학생의-제출-파일을-저장한다면-업로드와-접근-권한을-어떻게-설계하나요)

### 내 답변

### 모범 답변

S3 버킷은 비공개로 두고, 서버가 로그인 사용자와 과제 제출 권한을 확인한 뒤 업로드용 Presigned URL을 발급하겠습니다. 저장할 객체 키는 서버가 정하고, 업로드 완료 시 실제 객체와 크기 등을 확인한 뒤 제출 기록에 연결하겠습니다. 다운로드도 파일 ID로 소유자나 담당 교사 여부를 확인한 뒤 짧은 유효기간의 URL을 발급하겠습니다. 클라이언트가 보낸 학생 ID나 객체 키만 믿고 다른 학생의 파일에 접근을 허용하지 않겠습니다.

참고: [S3 Presigned URL](https://docs.aws.amazon.com/AmazonS3/latest/userguide/using-presigned-url.html)

## Q25

**AI 피드백 생성에 수십 초가 걸린다면 API와 처리 흐름을 어떻게 설계하겠어요?**

[질문과 꼬리 질문](interview-questions-artificial-society.md#25-ai-피드백-생성에-수십-초가-걸린다면-api와-처리-흐름을-어떻게-설계하겠어요)

### 내 답변

### 모범 답변

수십 초가 걸리는 AI 처리는 비동기로 분리하겠습니다. 처리할 입력과 작업을 DB에 저장한 뒤 작업 ID와 함께 202를 반환하고, 별도 작업자가 대기 중인 작업을 가져와 AI를 호출하도록 하겠습니다. 사용자는 상태 조회 API를 주기적으로 호출해 대기, 처리 중, 완료, 실패를 확인하고 완료된 결과를 받게 하겠습니다. 타임아웃과 재시도 횟수, 작업자 동시 처리 수를 제한해 외부 AI 호출이 몰리지 않도록 하겠습니다.

참고: [HTTP 202의 의미](https://www.rfc-editor.org/rfc/rfc9110.html#section-15.3.3)

## Q26

**AI 서비스가 타임아웃되거나 호출 제한에 걸려도 학생의 제출 내용은 보존해야 합니다. 어떻게 설계하겠어요?**

[질문과 꼬리 질문](interview-questions-artificial-society.md#26-ai-서비스가-타임아웃되거나-호출-제한에-걸려도-학생의-제출-내용은-보존해야-합니다-어떻게-설계하겠어요)

### 내 답변

### 모범 답변

제출 내용과 AI 처리 요청을 같은 DB 트랜잭션으로 저장하고, AI 호출은 커밋 후 별도 작업자가 수행하도록 하겠습니다. 실패해도 제출 내용은 유지하고, 일시적인 오류만 간격을 늘리며 제한된 횟수로 재시도하겠습니다. 제출 버전과 피드백 종류를 작업의 식별 기준으로 삼고, 유니크 제약과 원자적인 상태 변경으로 중복 작업과 결과 저장을 막겠습니다. 타임아웃 뒤 외부 AI 호출 자체의 중복까지 막으려면 해당 서비스의 멱등 키 지원이 필요하며, 지원하지 않으면 중복 호출 가능성은 남습니다.

참고: [Transactional Outbox](https://docs.aws.amazon.com/prescriptive-guidance/latest/cloud-design-patterns/transactional-outbox.html), [멱등 API](https://aws.amazon.com/builders-library/making-retries-safe-with-idempotent-APIs/)

## Q27

**교사는 담당 학급만, 학생은 본인의 학습 결과만 볼 수 있어야 합니다. 권한을 어떻게 설계하나요?**

[질문과 꼬리 질문](interview-questions-artificial-society.md#27-교사는-담당-학급만-학생은-본인의-학습-결과만-볼-수-있어야-합니다-권한을-어떻게-설계하나요)

### 내 답변

### 모범 답변

로그인과 역할 확인에 더해, 요청한 데이터와 사용자의 관계를 서버에서 검증하겠습니다. 학생은 제출 기록의 소유자와 로그인 사용자 ID가 같은지, 교사는 해당 학급을 담당하는지와 학교 범위가 맞는지 확인하겠습니다. 목록 조회에도 같은 권한 조건을 적용해 허용된 데이터만 DB에서 조회하겠습니다. 클라이언트가 보낸 사용자 ID나 역할은 신뢰하지 않고, 권한 검증을 공통화해 개별 API에서 빠지지 않도록 하겠습니다.

참고: [Spring Security 메서드 권한 검증](https://docs.spring.io/spring-security/reference/servlet/authorization/method-security.html)

## Q28

**한국과 미국의 교육과정·언어·시간대가 다르면 어떤 부분을 공통화하고 어떤 부분을 분리하겠어요?**

[질문과 꼬리 질문](interview-questions-artificial-society.md#28-한국과-미국의-교육과정언어시간대가-다르면-어떤-부분을-공통화하고-어떤-부분을-분리하겠어요)

### 내 답변

### 모범 답변

사용자, 과제, 제출, 피드백의 기본 처리 흐름은 공통화하겠습니다. 교육과정과 평가 기준은 국가별 설정과 버전으로, 콘텐츠와 번역은 교육과정과 언어별로 분리하겠습니다. 언어와 국가를 같은 값으로 취급하지 않고 별도로 관리하겠습니다. 과제 마감은 학교의 현지 시각과 시간대에서 UTC 시각으로 변환해 저장하고, 화면에는 사용자 시간대로 표시하겠습니다. 시간대는 America/New_York 같은 지역 ID를 사용하고, 서머타임으로 중복되거나 존재하지 않는 시각의 처리 기준도 정하겠습니다.

참고: [Java ZoneId와 시간대 규칙](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/time/ZoneId.html)

## Q29

**1주 단위 스프린트에서 기능 개발, 장애 대응, 기술 부채의 우선순위를 어떻게 정하나요?**

[질문과 꼬리 질문](interview-questions-artificial-society.md#29-1주-단위-스프린트에서-기능-개발-장애-대응-기술-부채의-우선순위를-어떻게-정하나요)

### 내 답변

### 모범 답변

데이터 유실이나 수업 중단을 일으키는 장애를 먼저 처리하고, 기능 개발은 이번 스프린트 목표와 사용자 영향에 따라 우선순위를 정하겠습니다. 기술 부채는 장애 재발이나 다음 개발을 막는 항목부터 함께 계획하겠습니다. 일정이 부족하면 핵심 사용자 흐름을 유지하면서 부가 기능을 줄이는 안과 일정 영향을 기획자에게 공유하겠습니다. 정확성과 권한 검증은 유지하고, 실제 완료 가능한 범위를 합의하겠습니다.

참고: [Scrum의 목표와 범위 협의](https://scrumguides.org/scrum-guide.html)

## Q30

**아티피셜소사이어티에 지원한 이유와, 입사 후 기여할 수 있는 부분을 설명해 주세요.**

[질문과 꼬리 질문](interview-questions-artificial-society.md#30-아티피셜소사이어티에-지원한-이유와-입사-후-기여할-수-있는-부분을-설명해-주세요)

### 내 답변

### 모범 답변

Learney처럼 학생의 학습 진행에 맞춰 AI 지원을 제공하는 서비스에서, 백엔드 개선이 실제 학습 경험으로 이어지는 일을 하고 싶습니다. 저는 LiDAR 프로젝트에서 API와 SQL을 개선해 통계 화면 표시 시간을 40초대에서 약 1.5초로 줄였고, Tag2Now를 AWS에서 운영하며 사용자 제보를 반영했습니다. 이 경험을 바탕으로 교사용 통계 조회의 성능과 수업 중 서비스 안정성을 개선하는 데 기여하고 싶습니다. 입사 후에는 주요 요청과 지표를 먼저 파악해 사용자가 겪는 지연과 실패부터 개선하겠습니다.

참고: [아티피셜소사이어티 서비스 소개](https://www.artificial.sc/)

### 확인 필요

- 지원 동기 문구는 초안. 교육 서비스에 관심을 갖게 된 개인적인 계기와 위 동기의 일치 여부 확인.

## Q31

**gRPC는 REST API와 무엇이 다르고, 어떤 상황에서 선택하겠나요?**

[질문과 꼬리 질문](interview-questions-artificial-society.md#q31) | [gRPC와 REST API 비교 자료](lidar-grpc.md)

### 모범 답변

REST는 리소스 중심의 아키텍처 스타일이고, gRPC는 원격 메서드 호출을 위한 프레임워크입니다. 일반적인 REST API는 URL과 HTTP 메서드로 리소스를 조회하거나 변경하고 JSON을 주고받습니다. gRPC는 `.proto`에 서비스와 메시지 계약을 정의하고, 생성된 Stub으로 원격 메서드를 호출하며 보통 HTTP/2와 Protobuf를 사용합니다.

gRPC는 계약을 공유하는 언어별 코드 생성과 클라이언트, 서버 및 양방향 스트리밍이 장점입니다. 내부 시스템에서 빈번한 데이터 교환이나 연속 전송이 필요하면 검토하겠습니다. REST API는 브라우저 및 외부 소비자와의 연동, 일반 HTTP 도구를 통한 조사, 조회 응답의 HTTP 캐시 활용이 편리해 공개 API에 우선 검토하겠습니다. 다만 REST도 HTTP/2와 바이너리 형식을 사용할 수 있고, OpenAPI로 계약과 코드를 생성할 수 있으므로 gRPC가 항상 더 빠르거나 REST에 타입 계약이 없다고 설명하지 않겠습니다. 실제 선택은 소비자 환경, 메시지 특성, 성능 측정과 운영 비용을 기준으로 판단하겠습니다.

### 프로젝트와 연결할 사실

LiDAR 데이터 서버와 C++ 데이터 중계 서버 사이에서 gRPC를 사용했고, 중계 서버와 백엔드는 REST API로 연계했습니다. 당시 최초 도입 이유와 RPC 유형은 코드 및 이력을 확인해 설명합니다.

참고: [gRPC 소개](https://grpc.io/docs/what-is-grpc/introduction/), [gRPC HTTP/2와 REST 비교](https://grpc.io/docs/what-is-grpc/faq/), [RPC 유형](https://grpc.io/docs/what-is-grpc/core-concepts/)

### 확인 필요

- 최초 gRPC 선택 주체와 도입 이유, 본인의 설계 및 구현 범위.
- `.proto`의 RPC 선언, 요청 시작 주체, 클라이언트와 서버 역할.
- 원시 포인트 클라우드 또는 처리된 위치 및 인원 데이터 여부, 수신 주기와 크기.
- 동기 또는 비동기 API, 수신 스레드와 처리 큐의 구조.
- Deadline, Keepalive, 재접속, 중복 및 유실 처리의 실제 구현.
- C++ 중계 서버 유지 이유와 REST API 호출 방향.

일반적인 선택 기준과 실제 도입 경험은 구분한다. 스트리밍 사용과 성능 비교 결과는 확인 전에는 경험으로 추가하지 않는다.
