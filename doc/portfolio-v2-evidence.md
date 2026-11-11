# 포트폴리오 v2 작성 근거

이 문서는 원고의 출처와 확인 범위를 관리하는 편집 자료다. 제출용 페이지에 포함하지 않는다.

## 작성 범위

- v1 원고: 기존 `content/` 최상위 파일 유지
- v2 원고: `content/v2/` 분리
- 콘텐츠 원천: Markdown 초안
- HTML 산출물: [index-v2.html](../index-v2.html)
- 기존 HTML: `index.html` 유지

## 출처

외부 구현 경로는 [BE 저장소](https://github.com/tag2now/tag2now-BE)와 [save-admin 저장소](https://github.com/tag2now/tag2now-save-admin)를 기준으로 표기한다.

| 식별자 | 원천 | 사용 범위 |
|---|---|---|
| P1 | [기존 Tag2Now 원고](../content/01-tag2now.md) | 개인 프로젝트 구분, 서비스 링크, 프로토콜 분석, 기존 운영 선택 |
| P2 | [기존 KTX 원고](../content/02-ktx-ticketing.md) | 실험 수치, 시나리오 구분, NAT 문제, 설계 판단 |
| P3 | [기존 AI 원고](../content/04-workflow.md) | 사내 리뷰 자동화, 승인 경계, 작업 목록과 Git 이력 |
| P4 | [기존 소개](../content/layout.md) | 이름, 연락처, 경력과 핵심 기술 |
| P5 | 현재 대화의 Tag2Now 아키텍처 검토 | 사용자 지적과 코드 대조, 주체 기준 화살표, 원본 접근과 큐 역할 보완 |
| P6 | 서비스 관계도 v4: `output/imagegen/tag2now-service-architecture-v4.png`, 전적 처리도 v4: `output/imagegen/tag2now-record-processing-v4.png` | P5에서 수정한 이미지 산출물 |
| P7 | KTX L1 시나리오와 결과: `ktx-ticketing-be/load-tests/scenarios/L1_single_seat_race.js`, `ktx-ticketing-be/docs/results/P4_Result.md` | 입장 토큰 사전 발급, k6 1,000 VU, 동일 좌석 경합, 부하 후 DB 검증 |
| T1 | 실시간 방 명세: `tag2now-BE/docs/spec/01-live-rooms.md` | 방 관측과 Phantom Room 판정, TTL과 단일 프로세스 제약 |
| T2 | 전적 명세: `tag2now-BE/docs/spec/11-records.md` | 세이브 비교, 상대 매칭, 큐 상태와 재처리 |
| T3 | 전적 서비스 구현: `tag2now-BE/src/records/service.py` | 상태와 이벤트의 원자적 저장, 이벤트 처리 트랜잭션 |
| T4 | PostgreSQL 어댑터: `tag2now-BE/src/records/adapters/postgresql.py` | SKIP LOCKED, 방 단위 잠금, 재시도와 dead 상태 |
| T5 | E2E 리플레이 안내: `tag2now-BE/tests/e2e/records/README.md` | 장애와 재시작 재현 시나리오, 검증 한계 |
| T6 | 세이브 관리 명세: `tag2now-BE/docs/spec/09-save-admin.md` | 수정 미리보기, SHA-256 검증, 세이브 조회와 디코딩 측정 |
| T7 | 운영 compose: `tag2now-BE/compose.prod.yml` | Lightsail 서비스 구성, 운영 Redis 부재 |
| T8 | 공유 RPCN 세션: `tag2now-BE/src/matching/rpcn_lifecycle.py` | 공유 세션, 잠금과 재연결 쿨다운 |
| T9 | 태스크 시작 코드: `tag2now-BE/src/app.py` | Collector, SaveCapture, EventConsumer의 BE 내부 실행 |
| T10 | save-admin 안내: `tag2now-save-admin/README.md` | RPCN DB의 읽기 전용 조회와 TUS 파일 접근 |
| T11 | save-admin 수정 구현: `tag2now-save-admin/script/tdt_admin.py`, `tag2now-save-admin/script/tdt_admin_server.py` | 접속 확인, 백업과 체크섬, 파일 재읽기와 최신 세이브 ID 확인, 감사 로그 |
| T12 | save-admin 테스트: `tag2now-save-admin/script/tests/test_server.py`, `tag2now-save-admin/script/tests/test_cli.py` | 미리보기 이후 원본 변경 거부, 접속 중 계정 차단, 백업 복구 테스트 구성 |
| T13 | RPCN 클라이언트 구현: `tag2now-BE/src/rpcn_client/client.py` | TLS 연결, ServerInfo 버전 검증, 로그인, 패킷 헤더와 길이 기반 수신, Protobuf 조회 |

## 수치의 사용 범위

| 수치 | 원천 | 원고 표현 |
|---|---|---|
| 60초 | T1 | 매칭 추론의 기본 TTL |
| 5회 | T4 | 처리 실패의 dead 전환 기준 |
| 6MB와 60ms | T6 | 697명 스냅샷의 전체 키 비교안 JSON 크기와 디코딩 시간 |
| 38KB와 0.3ms | T6 | 같은 스냅샷의 변경 후 최대 계급 키와 디코딩 시간 |
| 1,000 VU와 3회 | P2, P7 | k6 L1 동일 좌석 경합 검증 |
| 348~408ms와 833~863 TPS | P2 | L2 예매 부하 결과 |
| 8~9초와 26ms | P2 | L3 조회 캐시 OFF/ON 비교 |
| 85.4% | P2 | 입장 제어 실험의 초과 요청 반려 비율 |

## 확인 경계

- 기존 문서의 측정 기록과 현재 구현의 대조 수행
- 이번 작업에서 애플리케이션 테스트와 부하 실험 미실행
- E2E는 재현 시나리오의 존재로 표현
- 최근 실행 결과로서의 테스트 통과 주장 제외
- 세이브 파서가 공유하는 상수로 작성한 리플레이의 포맷 검증 한계 유지
- API 전체 응답 성능과 캐시 디코딩 비용 구분
- 승인된 예매의 성능과 반려 응답을 포함한 전체 요청 성능 구분
- Route 53, CloudFront, ALB, ECS Fargate, RDS와 ElastiCache Redis는 [초기 AWS 운영 기록](../PORTFOLIO.md#6-8-비용-최적화--ecs에서-lightsail로-전환)에 근거해 전환 전 구성으로 표기
- 전환 이후 운영 구성은 BE의 운영 Compose와 배포 워크플로 기준
- 무료 혜택 기간과 청구 내역이 없는 비용 수치 제외
- 기여 범위가 확인되지 않은 세이브 포맷 분석 표본 수치 제외
- Tag2Now의 종료 시점을 최신 기능까지 확대 추정하지 않고 기간 표기 보류

## 사용자 확인 대기

- Tag2Now 개발 중 AI 제안을 수정 또는 기각한 실제 사례
- 당시 문제, AI 제안, 본인 판단과 검증 방법
- 현재 AI 두 번째 사례: 대화에서 확인되는 포트폴리오 설계도 검토
- 질문의 목적: 더 구체적인 개발 사례가 있으면 원고의 대체 여부 판단
- 확인 전 Tag2Now의 특정 코드 구현이나 포맷 분석을 AI 협업 성과로 단정하지 않음

## 그림의 사용 범위

- 기존 v4 이미지: 서비스 관계와 호출 방향 검토용
- v2 본문: RpcnClient와 매칭 추론을 포함한 전체 구조 필요
- 기존 세로 처리도: A/B/C 세부 작업 검토용
- 제출용 도표: 가로 페이지 크기에서 별도 검토 필요
- 모든 연결선: 주체 → 접근 대상
- 응답 화살표: 생략

## 초안 점검 결과

| 항목 | 결과 | 범위 |
|---|---|---|
| 금지 표기 | 통과 | v2 원고 4개 파일의 대시, 중단점, 강조 따옴표와 이모지 검사 |
| 그림과 표 설명 | 검토 완료 | 개조식과 명사형 중심 구성 |
| 정보 압축 | 검토 완료 | 반복 맥락과 과장 표현 제거 |
| 항목별 정보 | 검토 완료 | 문제, 해결과 성과의 역할 분리 |
| 명사구 표현 | 검토 완료 | 설명 항목의 연결어미 축소 |
| 부사 | 검토 완료 | 판정 범위에 필요한 기본값과 근사값 유지 |
| 수치 강조 | 통과 | 수치 옆 강조 부사 미사용 |
| 서술식 판단 | 검토 완료 | 개요와 설계 판단에 서술식 사용 |
| 사실과 논지 | 검토 완료 | 기능과 측정 조건은 항목, 트레이드오프는 문단 |
| 용어 | 통과 | 개조식 표기 사용 |
| 페이지 구분 | 통과 | 소개 1쪽, Tag2Now 5쪽, KTX 2쪽, AI 2쪽 |
| 사례 구조 | 통과 | 개선 사례별 문제, 해결, 성과 확인 |
| 링크 | 통과 | 원고 링크와 근거 파일의 존재 확인 |
| Mermaid | 통과 | 전체 아키텍처 도표의 파서 검사 |
| 현재 덱 | 보존 | v1 원고와 `index.html` 수정 없음 |
| 화면 배치 | 통과 | 가로 10쪽 브라우저 확인, 데스크톱과 390px 화면 및 인쇄 모드의 넘침 없음 |
| AI 개발 사례 | 추가 확인 대기 | 두 번째 사례를 실제 설계도 검토 범위로 한정 |

## HTML 산출물 검증

- 문서 구조: A4 가로 크기의 고정 페이지 10개
- 페이지 대응: Markdown H2와 HTML 페이지의 일대일 매핑
- 구성: 인라인 스타일 1개와 탐색 스크립트 1개
- 도표: HTML에 포함한 SVG
- AWS 도식과 전환 주석의 Lightsail 표기: [AWS Architecture Icons](https://aws.amazon.com/architecture/icons/) 2026-07-31 패키지의 SVG
- 프로필 이미지: HTML에 포함한 데이터 URL
- 외부 표시 의존성: Google Fonts
- 탐색: 버튼, 방향키, 표지 링크와 직접 스크롤 확인
- 인쇄: 화면과 같은 페이지 크기, 탐색 UI 숨김 확인
- 원고 보존: v2 원고 4개 파일의 SHA-256 동일
- 기존 덱 보존: `index.html`의 SHA-256 동일
- 브라우저 검증 결과: 데스크톱, 390px 화면과 인쇄 모드의 10쪽 넘침 없음
- AWS 도식 대조: 초기 운영 기록의 서비스 구성과 배포 흐름 확인
- 검증 스크립트와 스크린샷: 로컬 임시 산출물, 버전 관리 제외
