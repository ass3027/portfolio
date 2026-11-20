# AI 에이전트를 활용한 코드 리뷰와 개발 검증

- 적용 범위: 사내 프로젝트 코드 리뷰, 개인 프로젝트 Tag2Now 백엔드

<a id="internal-code-review"></a>

## 사내 코드 리뷰와 이슈 발행

- AI 리뷰 근거를 파일과 줄 번호로 남기고 승인한 항목만 GitLab 이슈로 등록

사내 프로젝트에서 AI 에이전트가 발견 사항을 파일과 줄 번호를 근거로 정리하고 이슈 초안을 작성하도록 리뷰 스킬을 구성했습니다. 검토자가 승인한 항목만 GitLab에 등록하고, 등록된 이슈는 Webhook을 통해 KakaoWork에 공유되도록 했습니다.

<a id="tag2now-development-checks"></a>

## AI 에이전트 공통 개발 절차 설계

- AI 에이전트의 계획 승인과 커밋 전 테스트 및 코드 검사 절차 설계

AI 에이전트가 재사용할 코드와 테스트, 변경 범위를 계획에 적고, 사람이 그 계획을 승인한 뒤 구현하도록 공통 개발 절차를 설계했습니다. 커밋 요청을 받으면 테스트와 코드 검사를 실행하고, 통과 후 제시한 커밋 계획을 사람이 승인하면 커밋하도록 규칙을 정했습니다. Tag2Now 백엔드에 이 절차를 적용해 pytest로 단위 테스트와 통합 테스트를 실행하고, Ruff, Import Linter, basedpyright로 린트와 포맷, 의존성, 타입을 검사하도록 구성했습니다.

## 기술 스택

AI 에이전트, GitLab REST API, Webhook, KakaoWork, pytest, Ruff, Import Linter, basedpyright

[포트폴리오 원고](../../content/v2/03-ai-workflow.md)
