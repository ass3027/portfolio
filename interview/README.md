# 면접 준비 자료 안내

## 이력서와 포트폴리오 참조

| 자료 | Markdown | 원문 PDF | 범위 |
| --- | --- | --- | --- |
| 이력서 | [이력서 내용 정리](resume-reference.md) | [기본.pdf](기본.pdf) | 4쪽, 경력과 실무 프로젝트, 학력, 자격증 |
| 포트폴리오 | [포트폴리오 내용 정리](portfolio-reference.md) | [포트폴리오 PDF](<이세진 — 백엔드 개발자 포트폴리오.pdf>) | 14쪽, 프로젝트 구조, 기술 선택, 성과와 검증 수치 |

- 정리 기준일: 2026-10-11.
- 참조 방법: 아래 주제별 목차에서 문서 이동, 각 절의 원문 페이지로 PDF 대조.
- 자료 성격: 해당 PDF 버전의 내용 정리. PDF를 바꾸면 두 참조 문서의 출처와 내용을 함께 갱신.
- 표기 원칙: PDF에 적힌 사실과 수치 유지. 사용자 확인 내용과 문서 간 차이는 아래 표로 분리.
- 포트폴리오 수정 원천: 기존 `content/` Markdown 유지. 참조 문서는 제출 자료 확인용.
- 개인정보: 전화번호와 이메일은 PDF에서 확인. Markdown에는 프로젝트와 면접 관련 내용 중심 기록.
- 원문 보관: PDF는 기존 `.gitignore`에 따라 버전 관리 제외. 다른 환경에서 원문 링크 사용 시 같은 경로에 PDF 필요.

## 주제별 빠른 찾기

| 주제 | 자료 위치 | 원문 페이지 |
| --- | --- | --- |
| 자기소개, 핵심 역량 | [이력서 소개](resume-reference.md#profile), [포트폴리오 소개](portfolio-reference.md#profile) | 이력서 1쪽, 포트폴리오 1쪽 |
| 경력, 학력, 자격증 | [이력서 경력](resume-reference.md#career), [학력과 자격증](resume-reference.md#education), [포트폴리오 경력](portfolio-reference.md#career) | 이력서 1~3쪽, 포트폴리오 2쪽 |
| PL, 배포 표준화, Docker Compose | [Secuwatcher](resume-reference.md#secuwatcher) | 이력서 1쪽 |
| LiDAR SQL, 2,000만 건, 40초에서 1.5초 | [LiDAR 조회 개선](resume-reference.md#lidar), [측정 표현 확인](#differences) | 이력서 1~2쪽 |
| LiDAR gRPC, C++, 메트릭, 객체 TTL | [LiDAR 구조와 성과](portfolio-reference.md#lidar) | 포트폴리오 12쪽 |
| 영상 장애, FFmpeg, Kafka, Avro | [FireWatcher](resume-reference.md#firewatcher) | 이력서 2쪽 |
| 도메인 모델, GitLab CI, Registry | [Hemp](resume-reference.md#hemp) | 이력서 2~3쪽 |
| RPCN 역공학, 매칭 탐지, Redis TTL | [Tag2Now](portfolio-reference.md#tag2now) | 포트폴리오 4~7쪽 |
| ECS Fargate, Lightsail, 비용과 가용성 | [Tag2Now 인프라](portfolio-reference.md#tag2now-infra) | 포트폴리오 6~7쪽 |
| KTX 예매 경로, Redis와 DB 역할 | [KTX 구조](portfolio-reference.md#ktx) | 포트폴리오 8~9쪽 |
| KTX p95, TPS, 실험별 수치 | [KTX 검증 목표와 실측](portfolio-reference.md#ktx-metrics) | 포트폴리오 10쪽 |
| single-flight, 입장 제어, 불일치 보정 | [KTX 개선 사례](portfolio-reference.md#ktx-improvements) | 포트폴리오 11쪽 |
| AI 리뷰, 승인, GitLab과 카카오워크 | [AI 워크플로우](portfolio-reference.md#ai-workflow) | 포트폴리오 13~14쪽 |

<a id="differences"></a>

## 문서 간 차이와 면접 답변 시 확인 사항

이 표는 PDF 원문을 수정하지 않는다. 과거 작성된 면접 메모와 새 PDF가 다르면 출처별 표현을 유지하고, 사용자 확인 내용은 별도로 적용한다.

| 항목 | PDF 원문 표기 | 기존 자료 또는 확인 내용 | 면접 준비 기준 |
| --- | --- | --- | --- |
| LiDAR 기간 | 이력서 1쪽: 2025.08~2026.03. 포트폴리오 3·12쪽: 2025.09~2026.02 | [기존 확인 항목](questions.md#면접-전-확인할-문서수치)에도 기간 차이 기록 | 실제 참여 기간과 각 문서의 기간 기준 확인 |
| LiDAR 40초에서 1.5초 | 이력서 1쪽: 쿼리 응답 시간. 2쪽: 대량 데이터 조인 조회 응답 시간 | [사용자 확인 답변 Q13](answers.md#q13): API와 SQL 개선을 합친 화면 결과 표시 시간 | 답변에는 사용자 확인 범위 적용. SQL 단독 성과로 단정 금지 |
| 학력 | 새 이력서 3쪽: 2026.03 전문학사 졸업 | 기존 `questions.md`에는 졸업 예정 표기 확인 메모 존재 | 현재 PDF는 졸업으로 기록. 졸업 예정이라는 과거 메모와 구분 |
| gRPC 기여 범위 | 포트폴리오 2쪽: gRPC 기반 실시간 데이터 연계 설계 | [회사별 Q31](interview-answers-artificial-society.md#q31): 최초 선택 주체, 설계 및 구현 범위 확인 필요 | 최초 도입, 연계 설계, 기존 수신 서비스 유지보수의 본인 범위 확인 |
| LiDAR 메모리 70%, 코드량 50% | 이력서 2쪽은 여러 개선을 함께 서술. 포트폴리오 12쪽은 로직 통합과 객체 수명 정리의 성과로 배치 | C++ 누수 해결과 로직 통합 사례 별도 | 두 수치를 C++ 누수 수정만의 효과로 합쳐 설명하지 않음 |
| 두 종류의 80% | LiDAR: 분석 및 개선 시간. FireWatcher: 영상 장애 확인 및 조치 시간 | [LiDAR Q16](answers.md#q16), [FireWatcher 원문 정리](resume-reference.md#firewatcher) | 측정 업무와 프로젝트를 각각 구분 |
| KTX 성능 수치 | 포트폴리오 10쪽: L1, L2, L3, L5b로 분리 | [기존 질문 목록](questions.md)에 실험 조건 구분 항목 존재 | 833~863 TPS, 2,497 TPS, 약 189 TPS를 같은 시나리오 결과로 혼합 금지 |
| Tag2Now 비용 | 포트폴리오 6~7쪽: free tier, free tier 수준 비용 | 월별 청구 내역과 무료 혜택 기간은 PDF에 미기재 | 영구 무료 또는 특정 월 비용으로 확대 해석 금지 |
| 재직 기간과 경력 연차 | 이력서 1쪽: 4년 3개월, 2022.08부터 재직 중. 포트폴리오 1쪽: 4년 차 | PDF 발급일과 경력 개월 수 산식 미기재 | 원문 표기를 보존하고 실제 지원 시점 기준 연차 별도 확인 |

## 기존 면접 질문과 답변

- [공통 예상 질문](questions.md)
- [공통 답변과 사용자 확인 사실](answers.md)
- [자기소개](introduction.md)
- [아티피셜소사이어티 질문](interview-questions-artificial-society.md)
- [아티피셜소사이어티 답변](interview-answers-artificial-society.md)
- [알피(ARPI) JD와 경험 연결](jd-arpi.md)
- [알피(ARPI) 예상 기술면접 질문](interview-questions-arpi.md)
- [알피(ARPI) 내 답변 작성](interview-answers-arpi.md)
- [gRPC와 REST 비교](lidar-grpc.md)
- [Spring 트랜잭션](spring-transactions.md)

참조 문서는 제출 자료의 내용 확인용이며, 질문과 답변은 기존 문서에서 관리한다. 새 면접 답변에 들어갈 실제 경험, 측정 조건, 본인 기여는 확인된 범위에서 작성한다.
