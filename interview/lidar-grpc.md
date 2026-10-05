# gRPC와 REST API 비교 및 LiDAR 면접 준비

[회사별 답변 Q31](interview-answers-artificial-society.md#q31) | [예상 질문 Q31](interview-questions-artificial-society.md#q31)

작성일: 2026-10-06

학습 중심은 gRPC 자체와 REST API의 차이다. 프로젝트 사실은 [LiDAR 원천 문서](../content/03-lidar.md)를 기준으로 정리한다. 아래 일반 원리와 설계 예시는 당시 구현을 증명하지 않는다.

## 1. 먼저 설명할 프로젝트 구조

```text
LiDAR 데이터 서버 -> gRPC -> C++ 데이터 중계 서버 -> REST API -> Backend
```

- 확인된 통신 구간: LiDAR 데이터 서버와 C++ 중계 서버 사이의 gRPC.
- 확인 필요: RPC 요청 시작 주체와 서버 역할. 데이터 흐름 화살표만으로 클라이언트와 서버 판정 불가.
- 확인 필요: 원시 포인트 클라우드 수신 여부. 프로젝트 목적만으로 데이터 형식 판정 불가.
- 확인 필요: 기존 도입 배경과 C++ 중계 유지 이유. 후반 합류 경험과 최초 설계 책임 구분.

면접에서는 gRPC 개념과 REST API 비교를 먼저 준비한다. 프로젝트 질문에서는 실제 통신 구간과 사용한 RPC 유형으로 연결한다.

## 2. gRPC와 REST API 비교

gRPC는 원격 메서드 호출을 위한 프레임워크다. REST는 리소스 중심의 인터페이스 설계 방식이므로 동일한 수준의 기술 분류는 아니다. 면접에서는 흔히 쓰는 HTTP와 JSON 기반 REST API와 일반적인 HTTP/2 및 Protobuf 기반 gRPC를 비교한다. [gRPC FAQ](https://grpc.io/docs/what-is-grpc/faq/)

| 비교 항목 | 일반적인 REST API | 일반적인 gRPC |
| --- | --- | --- |
| 인터페이스 | URL, HTTP 메서드, 요청 및 응답 정의 | `.proto`의 서비스, RPC 메서드, 메시지 정의 |
| 데이터 표현 | JSON 등 | Protobuf 바이너리 |
| 호출 코드 | HTTP 클라이언트 또는 생성 SDK | 계약에서 생성한 Stub |
| 연속 전송 | SSE 등 별도 방식으로 구현 가능 | RPC 유형으로 Streaming 지원 |
| 조사와 연계 | 일반 HTTP 도구와 브라우저 활용 용이 | 스키마와 gRPC 지원 도구 활용 |
| 계약과 코드 생성 | OpenAPI 등의 명세와 생성 도구 선택 가능 | `.proto`와 생성 Stub 중심의 개발 흐름 |
| 오류 표현 | HTTP 상태 코드와 오류 본문 | gRPC Status와 필요 시 오류 상세 |
| 조회 캐시 | GET 응답의 HTTP 캐시 활용 가능 | 일반 HTTP 캐시보다 애플리케이션 캐시 검토 |

- HTTP/2: 하나의 연결에서 여러 스트림 전송 지원. RPC 메시지 스트리밍과 전송 계층 스트림 구분.
- Protobuf: 스키마 기반 직렬화와 언어별 코드 생성. 효율적인 데이터 표현 가능.
- 성능 판단: 메시지 크기, 빈도, 직렬화 비용, 네트워크 및 처리 로직 측정 필요. 기술 이름만으로 성능 우위 단정 불가.
- 도입 이유 답변: 일반적인 적합성과 당시 실제 선택 근거 구분.

참고: [gRPC 소개](https://grpc.io/docs/what-is-grpc/introduction/), [gRPC C++ 기본 안내](https://grpc.io/docs/languages/cpp/basics/)

### 같은 기능의 호출 형태

공항 구역의 현재 인원 조회를 가정한 학습 예시다.

```text
REST: GET /zones/A/passenger-count -> {"count": 120}
gRPC: stub.GetPassengerCount({zone_id: "A"}) -> PassengerCountResponse
```

REST는 구역의 인원 수라는 리소스와 GET의 의미를 조합한다. gRPC는 GetPassengerCount라는 원격 메서드와 요청 및 응답 타입을 계약에 정의한다. Stub은 실제 네트워크 요청을 수행하므로 로컬 호출처럼 보여도 지연과 실패가 발생한다. [REST 원전](https://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm), [gRPC 소개](https://grpc.io/docs/what-is-grpc/introduction/)

### Protobuf와 JSON의 차이

- JSON: 텍스트 표현, 필드 이름과 값을 전송, 사람이 본문 확인 가능.
- Protobuf: 스키마 기반 바이너리 표현, 필드 번호와 타입에 따른 인코딩, 해석 시 스키마 및 도구 활용.
- 코드 생성: `.proto`에서 메시지 클래스와 RPC Stub 생성. 언어별 타입 계약 공유.
- 계약 수정: 새 필드 번호, 기존 번호 보존, 삭제 번호 예약 등 호환성 관리.
- 검증 범위: 타입 계약과 업무 규칙 검증 구분. 좌표 범위와 필수 업무 값은 별도 검증 필요.

Protobuf는 JSON보다 메시지 크기와 직렬화 비용을 줄일 수 있다. 효과는 데이터 구조와 구현에 따라 달라지므로 동일 조건에서 측정한다. [Protobuf 인코딩](https://protobuf.dev/programming-guides/encoding/), [proto3 계약](https://protobuf.dev/programming-guides/proto3/)

REST API도 OpenAPI로 요청 및 응답 타입과 인터페이스를 정의하고 클라이언트 및 서버 코드를 생성할 수 있다. 계약과 타입 안전성의 유무만으로 두 방식을 구분하지 않는다. [OpenAPI 명세](https://spec.openapis.org/oas/latest.html), [OpenAPI 코드 생성](https://openapi-generator.tech/docs/generators/)

### HTTP/2가 주는 것

HTTP/2는 하나의 연결에서 여러 스트림을 다중화하고 헤더를 압축한다. 여러 요청을 같은 연결에서 처리할 수 있지만, TCP 패킷 손실 시 그 연결의 여러 스트림이 영향을 받을 수 있다. REST API도 HTTP/2를 사용할 수 있다. HTTP 버전과 API 설계 방식은 별개다. [HTTP/2 표준](https://www.rfc-editor.org/rfc/rfc9113.html)

HTTP/2의 스트림은 전송 계층 개념이고, gRPC Streaming은 한 RPC에서 여러 메시지를 주고받는 호출 모델이다. Unary gRPC도 HTTP/2 스트림을 사용한다. [gRPC 핵심 개념](https://grpc.io/docs/what-is-grpc/core-concepts/)

### 브라우저, 캐시와 오류 처리

- 브라우저: REST API는 일반 HTTP 요청으로 연동. 네이티브 gRPC는 브라우저에서 동일 방식으로 직접 호출하기 어려워 gRPC-Web 등 별도 연동 방식 검토.
- gRPC-Web: 네이티브 gRPC와 기능 동일성 가정 금지. 공식 구현의 Client Streaming 및 Bidirectional Streaming 지원 제한 확인.
- 캐시: REST의 GET 응답은 Cache-Control과 검증자를 활용한 HTTP 캐시 검토. 모든 GET의 무조건 캐시 가능 의미는 아님.
- gRPC 캐시: RPC별 애플리케이션 캐시 및 명시적인 캐시 계층 검토. 일반 HTTP GET 캐시와 방식 차이.
- 오류: REST에서는 HTTP 400, 404, 500 등과 본문, gRPC에서는 INVALID_ARGUMENT, NOT_FOUND, INTERNAL 등 Status 활용.
- 디버깅: JSON은 본문 확인 용이. gRPC는 grpcurl 등 지원 도구와 스키마 또는 서버 Reflection 활용.

참고: [gRPC-Web 공식 저장소](https://github.com/grpc/grpc-web), [HTTP 의미론](https://www.rfc-editor.org/rfc/rfc9110.html), [gRPC 상태 코드](https://grpc.io/docs/guides/status-codes/)

### 선택 기준

| 조건 | 우선 검토할 방식과 이유 |
| --- | --- |
| 내부 시스템, 계약 배포 통제 가능 | gRPC의 언어별 생성 코드와 공통 계약 활용 |
| 빈번한 메시지 교환 | gRPC 및 Protobuf의 크기와 직렬화 비용 측정 |
| 연속 또는 양방향 데이터 교환 | gRPC Streaming 검토, 소비자 환경 확인 |
| 브라우저 및 외부 공개 소비자 | REST API의 접근성과 HTTP 도구 활용 |
| 캐시 가능한 조회 | REST GET과 HTTP 캐시 검토 |
| 단순 CRUD, 처리 비용이 DB에 집중 | REST API 유지 가능, 전송 방식 변경 효과 측정 |

이는 비교에 따른 설계 판단이다. 내부 시스템의 REST API도 유효하고 외부 소비자에 gRPC를 제공할 수도 있다. 시스템 경계마다 다른 방식을 함께 사용하는 것도 가능하다.

## 3. 네 가지 RPC 유형과 실제 유형 확인

| 유형 | 요청과 응답 | 예시 |
| --- | --- | --- |
| Unary | 요청 1개, 응답 1개 | 현재 인원 수 한 번 조회 |
| Server Streaming | 요청 1개, 응답 여러 개 | 한 번 구독 후 위치 정보 연속 수신 |
| Client Streaming | 요청 여러 개, 응답 1개 | 측정 데이터 연속 업로드 후 요약 결과 수신 |
| Bidirectional Streaming | 요청 여러 개, 응답 여러 개 | 제어 요청과 측정 데이터 양방향 교환 |

실시간 데이터라는 이유만으로 Streaming 사용을 단정하지 않는다. Unary를 반복 호출하는 폴링도 가능하다. [gRPC 핵심 개념](https://grpc.io/docs/what-is-grpc/core-concepts/)

아래는 Server Streaming 판별용 예시이며 실제 프로젝트 스키마가 아니다.

```proto
rpc Subscribe(SubscribeRequest) returns (stream LidarFrame);
```

- 응답 앞 `stream`: Server Streaming.
- 요청 앞 `stream`: Client Streaming.
- 요청과 응답 앞 `stream`: Bidirectional Streaming.
- 양쪽 모두 `stream` 없음: Unary.

`.proto` 선언과 C++ 호출 코드를 함께 확인한다. Server Streaming이라면 클라이언트가 RPC를 시작하고 서버가 데이터를 연속 반환하므로, 데이터 송신자와 연결 시작 주체가 다를 수 있다. [C++ 기본 안내](https://grpc.io/docs/languages/cpp/basics/)

## 4. 코드에서 알아볼 구성 요소

| 구성 요소 | 역할과 확인 지점 |
| --- | --- |
| `.proto` | 메시지 필드와 RPC 계약 정의 |
| `protoc`와 gRPC 플러그인 | 메시지 코드와 RPC 호출 코드 생성 |
| Channel | 대상 서버와 통신하는 논리적 채널. 생성 위치와 재사용 여부 확인 |
| Stub | 원격 메서드를 호출하는 클라이언트 객체 |
| `ClientContext` | RPC별 Deadline, Metadata, 취소 등 관리 |
| `ClientReader`와 `Read()` | 동기 Server Streaming 클라이언트의 메시지 수신 |
| `Finish()`와 `Status` | 동기 스트림 종료 후 최종 결과 확인 |

`Read()`가 false를 반환했다는 사실만으로 정상 종료를 판단하지 않는다. 동기 Server Streaming 예제에서는 `Finish()`가 반환한 최종 상태를 확인한다. 비동기 API는 완료 이벤트와 객체 수명을 별도로 확인한다. [C++ 기본 안내](https://grpc.io/docs/languages/cpp/basics/)

## 5. 연결 단절과 데이터 중단

- Deadline: RPC 전체 완료 기한. 기본값에 의존하면 무기한 대기 가능. 장기 스트림에서 메시지별 무수신 감지와 별도 설계.
- Keepalive: HTTP/2 PING을 통한 연결 확인. 센서 데이터의 정상 생성이나 업무 처리 상태는 별도 확인.
- 상태 코드: `UNAVAILABLE`, `DEADLINE_EXCEEDED`, `CANCELLED`, `RESOURCE_EXHAUSTED`의 의미와 처리 분기 확인.
- 재접속: 스트림 종료 상태 확인, 이전 호출과 자원 정리, Backoff와 Jitter를 통한 재시도 간격 분산.
- 데이터 복구: 시퀀스나 시각을 활용한 누락 및 중복 감지 검토. 재구독만으로 중단 구간 데이터 복원 불가.

채널의 연결 회복과 종료된 RPC의 재생성은 구분한다. gRPC의 내장 Retry도 진행 중이던 스트림의 임의 지점부터 복구하는 기능은 아니다. 응답 헤더 수신 이후에는 RPC가 확정되어 추가 자동 Retry가 중단된다. [gRPC Retry](https://grpc.io/docs/guides/retry/)

Deadline 초과는 메시지가 일정 시간 오지 않았다는 뜻과 다르다. 장기 스트림은 전체 수명 정책과 마지막 메시지 수신 후 경과 시간을 함께 설계한다. [gRPC Deadline](https://grpc.io/docs/guides/deadlines/)

Keepalive 응답이 있어도 새 LiDAR 데이터가 멈출 수 있다. 연결 상태와 마지막 데이터 수신 시각을 함께 본다. [gRPC Keepalive](https://grpc.io/docs/guides/keepalive/), [상태 코드](https://grpc.io/docs/guides/status-codes/)

## 6. 수신이 처리보다 빠를 때

예를 들어 초당 100개를 수신하고 60개를 처리하면 별도 제한 없이 큐가 초당 40개씩 증가한다. 이는 해제 누락이 없어도 메모리를 늘릴 수 있는 학습용 사례다.

gRPC는 스트리밍 전송의 흐름 제어를 제공한다. 애플리케이션이 메시지를 계속 읽어 무제한 큐에 넣으면 업무 처리 속도에 맞는 제한은 별도로 필요하다. [gRPC Flow Control](https://grpc.io/docs/guides/flow-control/)

| 데이터 목적 | 검토할 정책 |
| --- | --- |
| 최신 위치 화면 | 메시지가 완전한 스냅샷이면 오래된 대기 데이터 교체 검토 |
| 이동 이력이나 인원 변화 집계 | 이벤트 유실 시 오차 검토, 저장 및 재처리 가능성 확인 |
| 제한된 처리 큐 | 큐 상한과 포화 시 대기, 거부, 저장 정책 결정 |

증분 이벤트를 최신 데이터 하나로 덮어쓰면 이동 경로나 집계가 손상될 수 있다. 데이터의 의미를 확인한 뒤 정책을 선택한다.

## 7. Protobuf 계약 변경

- 필드 번호: 전송 데이터의 필드 식별자. 배포된 필드 번호 변경과 재사용 금지.
- 필드 삭제: 삭제한 번호의 `reserved` 지정. 이름 예약도 함께 검토.
- 새 필드: 새 번호 사용과 구버전 동작 검증. 전송 호환성과 업무 의미 호환성 각각 확인.
- 기본값: proto3의 implicit scalar 필드는 미전달과 기본값 전달의 구분 불가. 예를 들어 인원 0명과 미수신 구분 필요 시 presence 활용 검토.

참고: [Protobuf proto3 가이드](https://protobuf.dev/programming-guides/proto3/)

## 8. 면접 직전 확인 순서

1. REST 리소스 설계와 gRPC 메서드 호출 차이 설명.
2. Protobuf, HTTP/2, 코드 생성과 Streaming의 역할 구분.
3. 브라우저 연동, 캐시와 오류 처리 비교.
4. 성능 측정 조건과 기술 선택 기준 정리.
5. 실제 `.proto`의 RPC 유형과 수신 데이터 확인.
6. 최초 선택 이유와 본인 기여 범위 확인.

우선 목표는 기술의 차이와 선택 근거 설명이다. CompletionQueue 내부 구현 같은 심화 주제는 당시 비동기 API 사용 여부를 확인한 뒤 준비한다.
