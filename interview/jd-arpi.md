# 알피(ARPI) Backend Engineer 공고 정리

[면접 자료 목차](README.md) | [예상 질문](interview-questions-arpi.md)

- 출처: [원티드 공고 389685](https://www.wanted.co.kr/wd/389685)
- 확인일: 2026-10-11
- 확인 방법: 원티드 본문 및 상세 정보 펼침
- 포지션: Backend Engineer, 경력 3~10년
- 제품 맥락: 의료진 대상 AI 심전도 및 의료 영상 분석
- 전형: 서류, 실무진 면접, 임원 면접, 최종 합격 통보
- 문서 성격: 면접 준비를 위한 공고 요약. 실제 내부 아키텍처나 면접 기출의 근거로 사용하지 않음

## 주요업무와 요구 역량

| 구분 | 공고 요약 |
| --- | --- |
| 주요업무 | AI 분석 백엔드, 대규모 의료 데이터 API, 병원 EMR 연동, 멀티테넌트, 데이터 보안, 성능 및 안정성 |
| 필수 | Spring Boot 개발·운영 3년 이상 또는 동등 역량, JPA·QueryDSL, RDBMS 모델링·쿼리 최적화 |
| 필수 | 웹·분산·메시지 아키텍처, 대용량 처리 경험, 트랜잭션·동시성·비동기, 테스트와 코드 리뷰 |
| 우대 | Kubernetes, 클라우드 운영, OAuth2/OIDC와 Keycloak, 글로벌 멀티리전, HIPAA·GDPR 관련 개발 경험 |
| 기술 목록 | GCP, Java, Kotlin, PostgreSQL, Spring Boot, Kubernetes |

기술 목록만으로 각 기술의 실제 사용 범위나 Kotlin 필수 여부를 확정하지 않는다. 우대사항의 규제 명칭만으로 회사 서비스의 법적 적용 범위를 판단하지 않는다.

## 제출 자료와 연결

| 공고 관심사 | 연결할 경험과 원문 | 준비 방향 |
| --- | --- | --- |
| 대용량 조회 | [이력서 LiDAR](resume-reference.md#lidar), 1~2쪽 | 실행 계획, 복합 인덱스, API 호출 개선의 효과 분리 |
| AI 분석 연계 | [이력서 FireWatcher](resume-reference.md#firewatcher), 2쪽 | Kafka·Avro 구현 범위, 전달 보장, 장애와 적체 |
| 데이터 파이프라인 안정성 | [포트폴리오 LiDAR](portfolio-reference.md#lidar), 12쪽 | 수신 장애, 메트릭, 객체 수명과 메모리 |
| 트랜잭션·동시성 | [포트폴리오 KTX](portfolio-reference.md#ktx), 8~11쪽 | DB 최종 정합성, Redis 실패, 보정의 한계 |
| JPA·도메인 설계 | [이력서 Hemp](resume-reference.md#hemp), 2~3쪽 | 상태 전이와 정책 응집, 영속성과 트랜잭션 |
| 배포·운영 | [이력서 Secuwatcher](resume-reference.md#secuwatcher), 1쪽 | 환경 설정, 배포 재현성, 현장 장애 대응 |
| 클라우드 선택 | [포트폴리오 Tag2Now](portfolio-reference.md#tag2now-infra), 6~7쪽 | 비용과 SPOF, 의료 서비스에 필요한 판단 변화 |
| 테스트·리뷰 | [Tag2Now](portfolio-reference.md#tag2now), 4~7쪽, [AI 워크플로우](portfolio-reference.md#ai-workflow), 13~14쪽 | 실제 검증 사례, 리뷰 판단, AI 오탐 확인 |

## 경험 확인이 필요한 영역

- QueryDSL의 실제 사용 여부와 구현 사례
- Kubernetes의 자격증 학습, 개인 실습, 실무 운영 범위
- PostgreSQL의 사용 범위와 직접 수행한 최적화
- GCP 및 Kotlin 사용 경험
- OAuth2/OIDC 및 Keycloak 구축·운영 경험
- 의료 도메인, EMR 연동, 멀티테넌트 구현 경험
- 의료 데이터 규제 대응 및 멀티리전 운영 경험
- Kafka의 offset 커밋, 재처리, 중복 방지, 스키마 변경 구현 범위

자료에 미기재된 경험은 경험 없음으로 단정하지 않는다. 사용자 확인 전에는 수행한 경험으로 답변에 넣지 않는다.
