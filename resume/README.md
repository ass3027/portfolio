# 이력서 원고

- [소개와 핵심 역량](profile.md)
- [회사 경력](career.md)
- [학력, 자격증, 공개 링크](credentials.md)
- 연락처는 로컬의 `contact.local.md`에서 관리합니다. 제출본을 만들 때 필요한 항목을 옮깁니다.

## 실무 프로젝트

| 프로젝트 | 골라 쓸 경험 |
| --- | --- |
| [김포공항 LiDAR](projects/gimpo-lidar.md) | [통계 조회 성능](projects/gimpo-lidar.md#query-performance), [관측 환경과 수신 안정성](projects/gimpo-lidar.md#operational-stability) |
| [Secuwatcher](projects/secuwatcher.md) | [배포 표준화와 CI/CD](projects/secuwatcher.md#deployment-standardization), [PL과 내부망 구축](projects/secuwatcher.md#project-leadership) |
| [발전사 REC 구매 시스템](projects/rec-purchase.md) | [전표 처리와 대금 지급 연계](projects/rec-purchase.md#purchase-eai), [거래 데이터 이관 및 검증](projects/rec-purchase.md#migration-validation) |
| [FireWatcher](projects/firewatcher.md) | [영상 장애 대응과 출력 안정성](projects/firewatcher.md#video-reliability), [Kafka와 Avro 데이터 연계](projects/firewatcher.md#metadata-streaming) |
| [Hemp](projects/hemp.md) | [도메인 모델링](projects/hemp.md#domain-modeling), [GitLab CI/CD 최적화](projects/hemp.md#ci-cd) |

## 개인 프로젝트와 AI 활용 사례

| 프로젝트 | 골라 쓸 경험 |
| --- | --- |
| [Tag2Now](projects/tag2now.md) | [RPCN 연계](projects/tag2now.md#rpcn-integration), [전적 복원과 이벤트 재처리](projects/tag2now.md#match-recovery), [AWS 운영](projects/tag2now.md#aws-operations) |
| [KTX 예매 재현](projects/ktx-ticketing.md) | [동시 예매 제어](projects/ktx-ticketing.md#booking-concurrency), [반복 조회 캐시](projects/ktx-ticketing.md#lookup-cache) |
| [AI 개발 워크플로우](projects/ai-workflow.md) | [사내 코드 리뷰](projects/ai-workflow.md#internal-code-review), [AI 에이전트 공통 개발 절차 설계](projects/ai-workflow.md#tag2now-development-checks) |

## 기업별 제출본 만들기

1. [기업별 조합 양식](applications/template.md)을 `applications/<기업명>.md`로 복사합니다.
2. 공고의 주요 요구사항에 맞는 성과 문단을 고르고, 관련성이 높은 순서대로 배치합니다. 한 프로젝트에서 필요한 문단만 가져올 수 있습니다.
3. 첫 사례에서 보여줄 강점이 소개에도 드러나도록 소개와 핵심 역량을 조정합니다. 소개에서 성과를 미리 언급했다면 본문에서는 본인 역할과 해결 과정을 보여줍니다.
4. 선택한 문단을 양식의 제출용 원고에 옮긴 뒤 전체를 읽으며 중복과 문장 흐름을 확인합니다. 문장을 다듬을 때 본인 역할과 수치의 측정 범위는 원고를 기준으로 확인합니다.

데이터 성능을 강조하는 공고에는 [LiDAR 통계 조회 성능](projects/gimpo-lidar.md#query-performance)을, 운영과 배포를 강조하는 공고에는 [Secuwatcher](projects/secuwatcher.md)와 [FireWatcher](projects/firewatcher.md)를 앞에 배치할 수 있습니다.
