# Tag2Now 실시간 매칭 및 대전 전적 서비스

- 구분: 개인 프로젝트
- 역할: 1인 풀스택 개발 및 운영

<a id="rpcn-integration"></a>

## RPCN 바이너리 프로토콜 연계

공식 SDK와 게임별 호출 명세가 없는 RPCN에 연동하기 위해 Rust 서버를 실행하고 구현을 분석했습니다. TLS 연결, 로그인, 바이너리 조회를 처리하는 Python 클라이언트를 구현하고, 실제 RPCN 연동 테스트에서 방 정보와 랭킹 조회를 검증했습니다.

<a id="match-recovery"></a>

## 대전 전적 복원과 이벤트 재처리

RPCN에서 대전별 결과를 바로 조회할 수 없어 게임 세이브의 누적 승패 변화와 방의 참가자 기록을 결합해 상대와 대전 결과를 복원했습니다. PostgreSQL에서 플레이어 상태 갱신과 이벤트 적재를 한 트랜잭션에 묶고, 처리하지 못한 이벤트를 다시 시도할 수 있게 작업 큐를 구성했습니다. save-admin 중단과 백엔드 재시작을 로컬 리플레이에서 재현해 전적이 누락되거나 중복되지 않고 복원되는지 확인했습니다.

<a id="aws-operations"></a>

## AWS 공개 서비스 운영과 배포 전환

공개 서비스의 프런트엔드와 백엔드를 ECS Fargate에 배포하고, GitHub Actions와 ECR로 이미지 빌드와 서비스 갱신을 자동화했습니다. CloudWatch에서 확인한 낮은 CPU 사용률과 AWS Billing에 기록된 비용을 근거로 Lightsail 단일 서버와 Docker Compose로 전환했습니다. 이미지 빌드와 ECR 등록 절차를 유지하고 마지막 배포 단계에서 Lightsail의 Compose 서비스를 갱신하도록 바꿨습니다.

## 기술 스택

Python, FastAPI, PostgreSQL, Docker, AWS, GitHub Actions

[포트폴리오 원고](../../content/v2/01-tag2now.md), [검증 범위와 측정 기록](../../doc/portfolio-v2-evidence.md#수치의-사용-범위)
