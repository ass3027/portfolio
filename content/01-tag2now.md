# Tag2Now: 철권 태그2 실시간 대시보드

- **구분**: 개인 프로젝트, 2026.03 ~ 2026.05, 1인 풀스택
- **GitHub**: https://github.com/orgs/tag2now/repositories
- **서비스**: https://match.tag2now.click/

---

## 개요, 핵심 목표, 기술 스택

### 개요

RPCS3 에뮬레이터로 TTT2를 즐기는 커뮤니티에는 활성 유저 수, 매칭 상대, 랭킹 정보를 한눈에 볼 도구가 없었다.

RPCN은 오픈소스지만 게임별 호출 방식은 문서화돼 있지 않고 공식 SDK도 없다. Rust 서버 실행과 분석으로 호출 프로토콜을 역공학했다. 단위 테스트와 통합 테스트로 검증한 Python 클라이언트로 실시간 대시보드를 구축했다.

### 핵심 목표

- **SDK 없는 RPCN 프로토콜 역공학**: Rust 서버 직접 분석, 프로토콜 명령 7종 테스트 검증
- **실시간 매칭 상태 시각화**: 스냅샷 Diff 기반 매칭 탐지와 Phantom Room 실시간 노출
- **AWS 서비스 조합 기반 실서비스 운영**: 인프라 구성, 자동 배포, 도메인 연결, 운영 비용 조정 직접 수행

### 기술 스택

`FastAPI` `Redis` `PostgreSQL` `AWS` `GitHub Actions`

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
| **Matching** | 방과 리더보드 실시간 수집, Redis 캐시, 매칭 탐지 |
| **History** | PostgreSQL 스냅샷 저장, 시간별 및 일별 통계 집계 |
| **Community** | 게시판과 댓글 CRUD |

### 운영 현황

- **호스팅**: AWS Lightsail 단일 서버와 Docker Compose 기반 `match.tag2now.click` 운영
- **비용 조정**: 서비스 규모 기반 ECS에서 Lightsail로 다운스케일
- **피드백 반영**: 오픈 카카오톡방 실사용자 제보 기반 오류 수정과 기능 개선

---

## 인프라 비교: ECS Fargate와 Lightsail

### 공통: 전환 전후 동일

- **런타임 진입**: 사용자 → Route 53 → CloudFront
- **CI/CD**: 개발자 → GitHub Actions → ECR
- **배포 단계**: 런타임별 상이

### BEFORE: ECS Fargate

- **배포**: 태스크 정의 갱신 → 서비스 업데이트, 롤링 배포
- **애플리케이션 계층**: ALB, ECS Fargate, 동일 ECR 이미지
- **데이터 계층**: RDS, DynamoDB, ElastiCache

### AFTER: Lightsail

- **배포**: SSH 접속 → `docker compose pull && up -d`
- **애플리케이션 계층**: Lightsail 인스턴스, Docker Compose, Nginx, FastAPI Backend
- **데이터 계층**: 컨테이너 PostgreSQL, Redis

### 트레이드오프

- **관리 요소**: ALB, 태스크 정의, 서비스, 관리형 DB → Docker Compose
- **비용**: 고정 과금 → free tier
- **파이프라인**: 테스트, ECR 푸시, AWS OIDC keyless 인증 유지. 배포 단계만 교체
- **배포 실패 감지**: `wait-for-service-stability` 기반 워크플로우 레벨 확인

운영 복잡도와 비용을 낮추는 대신 단일 노드 SPOF를 감수한 선택이다.

---

## 핵심 개선과 성과

### 1. RPCN 호출 프로토콜 역공학과 클라이언트 검증

- **문제**: 공식 SDK와 게임별 호출 명세 부재로 요청 순서와 바이너리 헤더 규칙 확인 불가.
- **해결**: Rust RPCN 서버 분석 기반 호출 프로토콜 역공학 및 Python 클라이언트 구현.
- **성과**: 서버 목록, 방 검색, 리더보드, 스코어 조회 등 프로토콜 명령 7종의 단위 테스트와 통합 테스트 검증.

### 2. 스냅샷 Diff 기반 매칭 탐지

- **문제**: 매칭 시작 시 방이 목록에서 사라져 조회 API만으로 매칭 중 유저 파악 불가.
- **해결**: 사라진 RANK_MATCH 방의 매칭 시작 판정과 방 재등장 및 TTL 만료 기반 상태 해제.
- **성과**: 매칭 중 유저의 Phantom Room 실시간 노출.

### 3. Redis 공유 캐시로 외부 RPCN 부하 제어

- **문제**: 동일 화면의 사용자별 반복 조회로 외부 RPCN 서버에 부하 전가 위험.
- **해결**: Redis 공유 캐시에 방 목록 10초, 리더보드 60초, 서버 목록 1시간 TTL 적용.
- **성과**: 외부 RPCN 요청 주기 제한과 다중 인스턴스의 캐시 공유.
