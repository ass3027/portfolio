# Tag2Now: 철권 태그2 실시간 대시보드

- **구분**: 개인 프로젝트, 2026.03 ~ 2026.05, 1인 풀스택
- **GitHub**: https://github.com/orgs/tag2now/repositories
- **서비스**: https://match.tag2now.click/

---

## 개요, 핵심 목표, 기술 스택

### 개요

RPCS3 에뮬레이터로 TTT2를 즐기는 커뮤니티는 활성 유저 수, 매칭 상대, 랭킹 정보를 한눈에 볼 도구가 없었다.

RPCN은 오픈소스지만 게임의 호출 방식이 문서화돼 있지 않고 공식 SDK도 없다. Rust로 된 RPCN 서버를 직접 실행하고 분석해 호출 프로토콜을 역공학하고, 단위 테스트와 통합 테스트로 검증한 Python 클라이언트를 구현하여 실시간 대시보드를 구축했다.

**구현에 그치지 않고 AWS에 직접 배포하여 실제 도메인(`match.tag2now.click`)으로 서비스를 운영하는 것**이 이 프로젝트의 목표였다. keyless CI/CD로 배포를 자동화하고, 오픈 카카오톡방의 실사용자 피드백을 받아 오류를 수정하고 개선하며 서비스를 유지했다.

### 핵심 목표

- **SDK 없는 RPCN 프로토콜 역공학**: Rust 서버 직접 분석, **프로토콜 명령 7종 테스트 검증**
- **실시간 대시보드 실서비스화**: **match.tag2now.click** 운영 중
- **AWS 운영과 자동 배포**: AWS OIDC keyless 인증으로 **ECR→ECS 배포 자동화**

### 기술 스택

`Python / FastAPI` `SQLAlchemy 2.0 (async)` `PostgreSQL` `Redis` `DynamoDB` `Docker` `Nginx` `GitHub Actions` `AWS ECS` `Lightsail` `Route53` `CloudFront` `React / TypeScript`

---

## 시스템 아키텍처

```
            ┌──────────────────────────────────────┐
            │       RPCN Server (PSN)              │
            │   TCP/TLS, Binary + Protobuf         │
            └───────────────────┬──────────────────┘
                                │ RpcnClient (역공학, 검증)
                                ▼
     ┌────────────────────────────────────────────────────┐
     │              FastAPI Application                   │
     │        Matching, History, Community                │
     └──────────┬──────────────────────────┬──────────────┘
                ▼                          ▼
        ┌──────────────┐         ┌──────────────────────┐
        │    Redis     │         │ PostgreSQL / DynamoDB│
        │  단기 캐시    │         │      영속 저장        │
        └──────────────┘         └──────────────────────┘
```

### 모듈 구성

| 모듈 | 역할 |
|---|---|
| **Matching** | RPCN 서버에서 방/리더보드 실시간 수집, Redis 캐시, 매칭 탐지 |
| **History** | 스냅샷 → PostgreSQL 저장, 시간별/일별 통계 집계 |
| **Community** | 게시판 CRUD (PostgreSQL / DynamoDB 어댑터 교체 가능) |

### 운영 현황

- **호스팅** AWS Lightsail 단일 서버 + Docker Compose. 실서비스 운영 중(`match.tag2now.click`)
- **다운스케일** ECS 대비 비용을 낮추려 규모에 맞춰 축소
- **피드백 반영** 오픈 카카오톡방의 실사용자 제보로 **오류 수정과 기능 개선 반복**

---

## 인프라 비교 (ECS Fargate ↔ Lightsail)

### 공통: 전환 전후 동일

- **① 런타임 진입 (요청)**: 사용자 → Route 53 → CloudFront
- **② CI/CD (빌드와 ECR 푸시)**: 개발자 → GitHub Actions → ECR
- 배포 단계(③)는 런타임별로 상이

### BEFORE: ECS Fargate (AWS 관리형 서비스 조합)

- ③ 배포: 태스크 정의 갱신 → 서비스 업데이트(롤링)
- ALB (Elastic Load Balancing)
- ECS Fargate ← 동일 ECR 이미지(③)
- 관리형 데이터 계층: RDS, DynamoDB, ElastiCache

### AFTER: Lightsail (단일 노드, free tier)

- ③ 배포: SSH 접속 → `docker compose pull && up -d`
- Lightsail 인스턴스, docker-compose
  - nginx (FE + 역방향 프록시)
  - backend (FastAPI) ← 동일 ECR 이미지(③)
  - 데이터 계층(컨테이너): postgresql, redis

### 트레이드오프

관리 요소 **다수(ALB, 태스크 정의, 서비스, 관리형 DB) → Docker Compose 하나**, 고정 과금 → **free tier**, 운영 복잡도↓. 가용성은 단일 노드(SPOF)로 감수한 트레이드오프다. 파이프라인 ①테스트와 빌드(**E2E 게이트** 통과 필수) ②ECR 푸시(OIDC keyless)는 그대로 재사용하고 **③배포 단계만 교체**했다. 배포 실패는 `wait-for-service-stability`로 워크플로우 레벨에서 감지했다.

---

## 핵심 기술적 도전

### 1. RPCN 호출 프로토콜 역공학 & 검증된 클라이언트

- **문제**: **게임 데이터를 가져올 공식 SDK와 문서 미존재.** 서버는 오픈소스지만, 게임이 실제로 호출하는 방식(요청 순서, 바이너리 헤더 규칙)은 어디에도 기록되지 않은 상태.
- **성과**: 서버 목록, 방 검색, 리더보드, 스코어 조회 등 **7종 프로토콜 명령 재현 및 검증**. 저수준 TLS/TCP 통신은 AI 협업으로 구현, **단위 테스트와 통합 테스트로 동작 실측 검증**.

### 2. 스냅샷 Diff 기반 매칭 탐지 (Phantom Room)

- **문제**: 상대를 찾기 시작하면 그 방은 목록에서 빠짐 → **조회 API만으로는 매칭 중인 유저를 볼 수 없어** 실제 활동 인원이 통째로 누락.
- **성과**: 직전 스냅샷과 **diff로 사라진 RANK_MATCH 방을 매칭 시작으로 판정**(재등장이나 TTL 만료로 해제) → **매칭 중 유저 수를 정확히 집계**, 가상 방(Phantom Room)으로 실시간 노출.

### 3. 비동기 선언적 트랜잭션 데코레이터

- **문제**: 서비스 메서드마다 **세션 열기, 커밋, 롤백 코드가 동일하게 반복**. 정작 중요한 비즈니스 로직이 그 사이에 묻히는 구조.
- **성과**: Spring의 `@Transactional` 패턴을 Python async에 적용한 `@transactional` 데코레이터로 세션 관리 코드를 제거, 예외 시 자동 롤백 보장.

### 4. Redis 공유 캐시: 같은 결과를 반복 조회하는 트래픽

- **문제**: **같은 화면인데 조회는 사람 수만큼 반복.** 매 요청마다 RPCN 재조회 시, 내 서버는 물론 **통제 불가한 외부 RPCN 서버까지 부하 전가 위험.**
- **성과**: 데이터 수명에 맞춘 **Redis TTL 캐시**(방 목록 10s, 리더보드 60s, 서버 목록 1h)로 중복 조회 제거. 캐시를 프로세스 밖 Redis에 두어 **백엔드를 여러 인스턴스로 늘려도 캐시 공유**.

---

## 배운 점 & 성장

| 항목 | 내용 |
|---|---|
| 프로토콜 역공학 & 검증 | 오픈소스 RPCN(Rust) 서버를 직접 실행하고 분석해 미문서화된 호출 방식을 역공학. 저수준 통신은 AI 협업으로 구현하고 단위 테스트와 통합 테스트로 검증, 명세 struct는 재사용하고 수정. |
| 실제 서비스 운영 & 피드백 | 배포로 끝내지 않고 실서비스로 운영. 오픈 카카오톡방에서 받은 실사용자 제보로 오류 수정과 기능 개선을 반복. |
| 다양한 DB 사용 경험 | Redis(TTL 캐시), PostgreSQL(시계열 집계), DynamoDB(게시판)를 한 서비스에서 함께 운용하며 각 DB의 제약을 체감, 데이터 특성에 맞게 선택. |
| CI/CD 자동화 | GitHub Actions로 PR → 통합 테스트, 태그 → ECR + ECS 배포 완전 자동화. |
| AWS 인프라 이해 | ECR, ECS(Fargate), ALB, ElastiCache, Route53, CloudFront, IAM을 직접 구성하고 Lightsail로 옮기며 배포 경로 전체를 이해. 최종적으로 OIDC keyless 배포로 귀결. |
| 비용 최적화 | CloudWatch + Billing 분석으로 오버스펙 인식 → 서비스 규모에 맞는 인프라 판단. |
