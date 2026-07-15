# Tag2Now — 포트폴리오

> **철권 태그 토너먼트 2 실시간 정보 대시보드**  
> 오픈소스 PS3 에뮬레이터(RPCS3)의 온라인 서버(RPCN)에 직접 접속하여 실시간 플레이어 현황·리더보드·통계를 제공하는 웹 서비스

| 항목 | 내용 |
|------|------|
| **역할** | 1인 풀스택 개발 (기획 · 설계 · 구현 · 배포) |
| **GitHub** | `github.com/[YOUR_ID]/tag2now` |
| **서비스** | `[YOUR_DOMAIN]` |

---

## 1. 프로젝트 배경

RPCS3 에뮬레이터로 TTT2를 즐기는 국내외 커뮤니티는 활성 유저 수, 대기 중인 매칭 상대, 랭킹 정보를 한눈에 볼 수 있는 도구가 없었다. RPCN 서버는 클로즈드 소스여서 공식 SDK가 존재하지 않는다. 직접 프로토콜을 분석하고 클라이언트를 구현하여 문제를 해결했다.

---

## 2. 기술 스택

### Backend (주력)
| 분류 | 기술 |
|------|------|
| 언어 / 프레임워크 | Python 3.10+, FastAPI 0.110 |
| ORM / DB 드라이버 | SQLAlchemy 2.0 (async), asyncpg |
| 데이터 검증 | Pydantic 2.0 |
| 캐시 | Redis 5.0 |
| 프로토콜 | TCP/TLS 바이너리, Protocol Buffers (gRPC tools) |
| DB | PostgreSQL, DynamoDB |
| 컨테이너 | Docker, Docker Compose, Nginx |
| CI/CD | GitHub Actions |
| Cloud | Amazon ECR, Amazon ECS (Fargate), AWS OIDC |

### Frontend
React 18 · TypeScript 5 · Vite · Tailwind CSS · Recharts (Vitest + Playwright)

---

## 3. 시스템 아키텍처

```
┌───────────────────────────────────────────────────────┐
│                    RPCN Server (PSN)                  │
│  rpcn.rpcs3.net:31313  (TCP/TLS, Binary + Protobuf)  │
└───────────────────────────┬───────────────────────────┘
                            │ RpcnClient (직접 구현)
                            ▼
┌───────────────────────────────────────────────────────┐
│                   FastAPI Application                 │
│                                                       │
│  ┌─────────────┐  ┌─────────────┐  ┌──────────────┐  │
│  │  Matching   │  │   History   │  │  Community   │  │
│  │  Module     │  │   Module    │  │  Module      │  │
│  │  /api/rooms │  │  /api/      │  │  /api/       │  │
│  │  /api/lb    │  │  history/*  │  │  community/* │  │
│  └──────┬──────┘  └──────┬──────┘  └──────┬───────┘  │
│         │  Event Bus     │                │           │
│         └────────────────┘                │           │
└──────────────┬────────────────────────────┼───────────┘
               │                            │
      ┌────────▼──────┐            ┌────────▼──────┐
      │     Redis     │            │  PostgreSQL   │
      │  (캐시 계층)  │            │  DynamoDB     │
      └───────────────┘            └───────────────┘
```

### 세 도메인 모듈

| 모듈 | 역할 |
|------|------|
| **Matching** | RPCN 서버에서 방/리더보드 실시간 수집, Redis 캐시, 매칭 탐지 |
| **History** | 스냅샷 → PostgreSQL 저장, 시간별/일별 통계 집계 |
| **Community** | 게시판 CRUD (PostgreSQL / DynamoDB 어댑터 교체 가능) |

---

## 4. 핵심 기술적 도전

### 4-1. 클로즈드 소스 바이너리 프로토콜 역공학 구현

**문제**  
RPCN은 RPCS3 프로젝트가 운영하는 PSN 호환 서버로, 공식 클라이언트 SDK가 존재하지 않는다. 서버와 통신하려면 RPCS3 C++ 소스코드의 프로토콜 문서를 분석하여 Python 클라이언트를 처음부터 직접 구현해야 했다.

**프로토콜 구조**

```
┌──────────────────────────────────────────────┐
│ 15-byte Little-Endian Header (<BHIQ)         │
├──────┬──────────┬────────────┬───────────────┤
│ u8   │ u16      │ u32        │ u64           │
│ type │ command  │ total_size │ packet_id     │
└──────┴──────────┴────────────┴───────────────┘
│ Payload                                      │
│  · Simple  → raw struct.pack LE integers    │
│  · Complex → u32 LE length + Protobuf bytes │
└──────────────────────────────────────────────┘
```

**구현 요점** (`src/rpcn_client/client.py`)

```python
def connect(self) -> int:
    ctx = ssl.create_default_context()
    ctx.check_hostname = False
    ctx.verify_mode = ssl.CERT_NONE  # RPCN은 자체 서명 인증서 사용

    raw = socket.create_connection((self.host, self.port), timeout=30)
    self._sock = ctx.wrap_socket(raw, server_hostname=self.host)

    hdr = self._recv_exact(HEADER_SIZE)
    pkt_type, _cmd, pkt_size, _pkt_id = struct.unpack(_HDR_FMT, hdr)
    # ...ServerInfo 패킷에서 프로토콜 버전 검증

def _recv_reply(self) -> tuple[int, bytes]:
    """서버 비동기 알림(PKT_NOTIF)을 무시하며 Reply를 대기."""
    while True:
        hdr = self._recv_exact(HEADER_SIZE)
        pkt_type, cmd, pkt_size, _pkt_id = struct.unpack(_HDR_FMT, hdr)
        payload = self._recv_exact(pkt_size - HEADER_SIZE)

        if pkt_type == PKT_NOTIF:  # 서버 푸시 알림은 스킵
            continue
        # ...
```

**성과**: 공식 SDK 없이 서버 목록, 방 검색, 리더보드, 스코어 조회 등 7개 커맨드 완전 구현

---

### 4-2. 스냅샷 Diff 기반 매칭 탐지 알고리즘 (Phantom Room)

**문제**  
TTT2 매칭 플로우: `방 생성(1인) → 상대 탐색 → 매칭 성사 → 방 소멸`. 매칭 대기 중인 플레이어의 방은 서버 목록에서 사라지므로, 사용자 입장에서는 활성 플레이어가 없는 것처럼 보였다.

**해결** (`src/matching/matchmaking_tracker.py`)

```python
def update_and_get_matchmaking(current_rooms: list[RoomInfoDTO]) -> list[RoomInfoDTO]:
    """이전 스냅샷과 비교해 사라진 RANK_MATCH 방의 플레이어를 매칭 중으로 판정."""
    prev_keys = set(_prev_rooms)
    curr_keys = set(current)

    # 사라진 RANK_MATCH 방 → 해당 유저를 matchmaking_players에 등록
    for room_id in prev_keys - curr_keys:
        prev = _prev_rooms[room_id]
        if prev.room_type == RoomType.PLAYER_MATCH:
            continue  # 랭크 매치만 탐지
        for user in prev.users:
            _matchmaking_players[user.npid] = _MatchmakingPlayer(...)
            publish(MatchmakingDetected(npid=user.npid, ...))

    # 다시 방에 나타난 유저 → 매칭 완료 또는 이탈로 해제
    for room in current.values():
        for user in room.users:
            if user.npid in _matchmaking_players:
                reason = "found_opponent" if room.is_gaming() else "rejoined_room"
                publish(MatchmakingResolved(npid=user.npid, reason=reason, ...))

    # TTL 초과 항목 만료 처리
    for npid in list(_matchmaking_players):
        if now - _matchmaking_players[npid].last_seen > ttl:
            del _matchmaking_players[npid]
            publish(MatchmakingResolved(npid=npid, reason="expired", ...))

    # Phantom Room 생성: 매칭 중인 플레이어를 가상의 방으로 UI에 노출
    return [RoomInfoDTO.phantom(...) for mp in _matchmaking_players.values()]
```

**성과**: 빈 방 목록 대신 "매칭 탐색 중" 플레이어를 실시간으로 시각화, 커뮤니티 체감 활성도 향상

---

### 4-3. 비동기 선언적 트랜잭션 데코레이터

**문제**  
SQLAlchemy 2.0 async 세션의 `begin/commit/rollback` 관리 코드가 서비스 레이어 전반에 반복되어 가독성과 실수 가능성이 높았다.

**해결** (`src/shared/database.py`)  
Spring의 `@Transactional` 패턴을 Python async 환경에 적용했다.

```python
def transactional(fn):
    """성공 시 commit, 예외 시 rollback을 보장하는 서비스 데코레이터."""
    @wraps(fn)
    async def wrapper(*args, **kwargs):
        async with async_session_factory() as session:
            async with session.begin():
                return await fn(*args, session=session, **kwargs)
    return wrapper

def read_only(fn):
    """읽기 전용 세션 — 트랜잭션 없이 세션만 제공."""
    @wraps(fn)
    async def wrapper(*args, **kwargs):
        async with async_session_factory() as session:
            return await fn(*args, session=session, **kwargs)
    return wrapper
```

**사용 예시** — 서비스 레이어에서 세션 관리 코드가 완전히 제거됨:

```python
@transactional
async def record_snapshot(rooms: list, *, session: AsyncSession) -> None:
    await repo.record_snapshot(session, rooms)

@read_only
async def get_player_stats(npid: str, *, session: AsyncSession) -> PlayerStats:
    return await repo.get_player_stats(session, npid)
```

---

### 4-4. 다중 데이터베이스 전략 & 계층적 캐싱

**문제**  
데이터마다 접근 패턴이 극명히 달랐다:
- 방 목록: 10초마다 갱신, 수 밀리초 이내 응답 필요
- 리더보드: 60초 주기, 조회 빈도 높음
- 플레이 통계: 90일 장기 보관, 사전 집계 필요
- 게시판: 유연한 스키마, 댓글 중첩

**해결 — DB 선택 기준**

| 데이터 | 저장소 | 이유 |
|--------|--------|------|
| 방 목록 / 리더보드 | Redis (TTL 캐시) | 고빈도 읽기, 만료 자동화 |
| 랭크 매치 스냅샷 | PostgreSQL | 시계열 집계, 복잡한 조인 |
| 시간별 통계 | PostgreSQL (사전 집계) | 90일 retention, UPSERT |
| 게시판 | DynamoDB / PostgreSQL (교체 가능) | 어댑터 패턴으로 추상화 |

**캐시 TTL 설계** (`src/shared/cache.py`, `src/shared/settings.py`):

```
servers   (서버 목록)   → 1시간
leaderboard (리더보드)  → 60초
rooms_all (전체 방)     → 10초
stats     (통계)        → 5분
phantom rooms           → 캐시 안 함 (매 스냅샷마다 재계산)
```

**PostgreSQL 사전 집계 — Upsert로 피크 값 유지** (`src/history/adapters/postgresql.py`):

```python
stmt = pg_insert(HourlyStatsRow).values(
    hour_key=hour_key,
    total_players=len(rooms) * 2,
    total_rooms=len(rooms),
)
stmt = stmt.on_conflict_do_update(
    index_elements=["hour_key"],
    set_={
        "total_players": func.greatest(
            HourlyStatsRow.total_players, stmt.excluded.total_players
        ),
    },
)
```

---

## 5. 이벤트 기반 설계

API 응답을 블로킹하지 않고 히스토리를 기록하기 위해 Fire-and-forget 이벤트 버스를 구현했다.

```
GET /api/rooms/all 호출
        │
        ▼
 RPCN에서 방 목록 수신
        │
        ├──── ActivitySnapshot 발행 ──→ history 모듈이 비동기 저장
        │
        └──── matchmaking_tracker.update() 호출
                  │
                  ├── MatchmakingDetected 발행
                  └── MatchmakingResolved 발행
```

이벤트 타입 (`src/matching/events.py`):
- `ActivitySnapshot` — 방 스냅샷 전달 → 히스토리 DB 저장
- `MatchmakingDetected` — 매칭 진입 감지
- `MatchmakingResolved` — 매칭 완료/이탈/만료

---

## 6. 인프라 & CI/CD 파이프라인

### 6-1. 전체 흐름

```
PR to master                         Tag push (v*)
      │                                    │
      ▼                                    ▼
 ┌─────────────────────┐       ┌───────────────────────────┐
 │   CI Workflow        │       │      CD Workflow           │
 │                     │       │                           │
 │  unit-tests         │       │  test                     │
 │  (pytest tests/unit)│       │  (unit + typecheck + E2E) │
 │         │           │       │          │                │
 │         ▼           │       │          ▼                │
 │  integration-tests  │       │  build & push to ECR      │
 │  (PostgreSQL+Redis  │       │  (OIDC keyless auth)      │
 │   +DynamoDB Local)  │       │          │                │
 └─────────────────────┘       │          ▼                │
                               │  ECS task def update      │
                               │  (이미지만 교체)          │
                               │          │                │
                               │          ▼                │
                               │  ECS service deploy       │
                               │  (wait-for-stability)     │
                               └───────────────────────────┘
```

---

### 6-2. CI — 통합 테스트 환경 자동화 (`ci.yml`)

PR이 master로 열리면 두 단계 테스트가 순차 실행된다.

```yaml
jobs:
  unit-tests:
    steps:
      - uses: actions/setup-python@v5
        with: { python-version: "3.12", cache: pip }
      - run: pip install -r requirements.txt && pip install -e .
      - run: pytest tests/unit/ -v

  integration-tests:
    needs: unit-tests          # unit 통과 후에만 실행
    steps:
      # PostgreSQL 16 + Redis 7 + DynamoDB Local 기동
      - run: docker compose -f compose.test.yml up -d --wait

      # protobuf 코드 생성 (빌드 아티팩트, 미커밋)
      - run: python -m grpc_tools.protoc -I. --python_out=src/rpcn_client np2_structs.proto

      - run: pytest tests/integration/ -v
        env:
          REDIS_URL: redis://localhost:6379/0
          DB_URL: postgresql://tag2now:tag2now@localhost:5432/tag2now
          RPCN_USER: ${{ secrets.RPCN_USER }}    # 실제 RPCN 서버 접속

      - if: always()
        run: docker compose -f compose.test.yml down
```

**설계 포인트**:
- DB를 Mock하지 않고 실제 PostgreSQL/Redis/DynamoDB Local에 쿼리 — 환경 불일치 버그 사전 차단
- `always()` 조건으로 테스트 실패 시에도 컨테이너 정리 보장

---

### 6-3. CD — ECR 빌드 & ECS 배포 (`deploy.yml`)

`v*` 태그를 푸시하면 테스트 → 빌드 → 배포가 자동 실행된다.

```yaml
on:
  push:
    tags: ['v*']        # 릴리스 태그 기반 트리거

jobs:
  build:
    permissions:
      id-token: write   # OIDC 토큰 발급에 필요
      contents: read
    outputs:
      image: ${{ steps.build-image.outputs.image }}
    steps:
      # IAM 액세스 키 없이 OIDC로 AWS 인증
      - uses: aws-actions/configure-aws-credentials@v6
        with:
          role-to-assume: ${{ secrets.AWS_ROLE_ARN }}
          aws-region: ${{ vars.AWS_REGION }}

      - uses: aws-actions/amazon-ecr-login@v2

      - name: Build and push Docker image
        env:
          ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}
          IMAGE_TAG: ${{ github.ref_name }}   # e.g. v1.2.3
        run: |
          # 버전 태그 + latest 이중 태그
          docker build \
            -t $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG \
            -t $ECR_REGISTRY/$ECR_REPOSITORY:latest .
          docker push $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG
          docker push $ECR_REGISTRY/$ECR_REPOSITORY:latest
          echo "image=$ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG" >> "$GITHUB_OUTPUT"

  deploy:
    needs: build
    steps:
      # 기존 ECS task definition에서 이미지 URI만 교체
      - uses: aws-actions/amazon-ecs-render-task-definition@v1
        id: task-def
        with:
          task-definition-family: ${{ vars.ECS_TASK_DEFINITION_FAMILY }}
          container-name: ${{ vars.ECS_TASK_CONTAINER_NAME }}
          image: ${{ needs.build.outputs.image }}

      # 롤링 배포 후 서비스 안정화까지 대기
      - uses: aws-actions/amazon-ecs-deploy-task-definition@v2
        with:
          task-definition: ${{ steps.task-def.outputs.task-definition }}
          cluster: ${{ vars.ECS_CLUSTER }}
          service: ${{ vars.ECS_SERVICE }}
          wait-for-service-stability: true   # 배포 실패 시 워크플로우 실패 처리
```

---

### 6-4. AWS OIDC — 키 없는 배포 보안

```
GitHub Actions Runner
       │
       │  OIDC 토큰 (JWT) 발급
       ▼
  GitHub OIDC Provider
       │
       │  AssumeRoleWithWebIdentity
       ▼
  AWS STS ──→ 임시 자격증명 (15분 TTL)
       │
       ▼
  ECR push / ECS deploy
```

IAM 액세스 키를 GitHub Secrets에 저장하지 않아 자격증명 유출 위험을 원천 차단한다.  
`id-token: write` 권한 하나로 동작하며, IAM Role의 신뢰 정책에서 특정 레포지토리·브랜치만 허용한다.

---

### 6-5. 프론트엔드 CD — E2E 게이트

배포 전 Playwright E2E 테스트를 의무 통과 조건으로 설정했다.

```yaml
jobs:
  test:
    steps:
      - run: npm test                          # Vitest 단위 테스트
      - run: npm run typecheck                 # TypeScript 타입 검사
      - run: npx playwright install --with-deps chromium
      - run: npx playwright test               # E2E (실제 브라우저)

      # 실패 시 리포트·스크린샷을 Artifact로 보존 (14일)
      - uses: actions/upload-artifact@v4
        if: ${{ !cancelled() }}
        with:
          name: playwright-report
          path: e2e/playwright-report/
          retention-days: 14

  build:
    needs: test    # E2E 통과해야 ECR 빌드 진행
    # ... (백엔드와 동일한 ECR → ECS 패턴)
```

---

### 6-6. Docker 멀티스테이지 빌드

```dockerfile
# Backend — protobuf 생성 포함
FROM python:3.12-slim AS base
RUN pip install -r requirements.txt
RUN python -m grpc_tools.protoc -I. --python_out=src/rpcn_client np2_structs.proto
CMD ["uvicorn", "app:app", "--host", "0.0.0.0"]

# Frontend — Node 빌드 → Nginx 서빙
FROM node:24-alpine AS build
RUN npm ci && npm run build

FROM nginx:alpine
COPY --from=build /app/dist /usr/share/nginx/html
COPY nginx.conf.template /etc/nginx/templates/
# envsubst로 BACKEND_URL 런타임 주입
```

### 6-7. Nginx 설정

```nginx
location /health { return 200 'ok'; }   # ECS 헬스체크 엔드포인트

location /api/ {
    proxy_pass http://${BACKEND_URL}/;  # 런타임 환경변수로 백엔드 URL 주입
}

location /assets/ {
    expires 1y;
    add_header Cache-Control "public, immutable";  # Vite 해시 파일명 → 영구 캐싱
}

location / {
    try_files $uri $uri/ /index.html;   # SPA fallback
    add_header Cache-Control "no-cache";
}
```

---

### 6-8. 비용 최적화 — ECS에서 Lightsail로 전환

ECS 배포 후 **CloudWatch**와 **AWS Billing**을 분석한 결과, 서비스 규모 대비 인프라가 과도하다고 판단했다.

**문제 인식**

| 항목 | 내용 |
|------|------|
| CloudWatch CPU | Fargate 태스크 평균 CPU 사용률이 지속적으로 낮음 — 소규모 트래픽 확인 |
| AWS Billing | ECS Fargate(BE+FE) + ElastiCache Redis + ALB 합산 비용이 트래픽 대비 과도 |
| Fargate 과금 구조 | 최소 vCPU/메모리 단위로 24시간 과금 — 요청이 없어도 비용 발생 |
| ElastiCache Redis | 캐시 용도만인데 단독 인스턴스 비용 발생 |

**전환 결정**

```
[Before] AWS ECS (Fargate)
┌────────────────────────────────────────────────┐
│  ECR                                           │
│  ECS Fargate (Backend)  ←→  ElastiCache Redis │
│  ECS Fargate (Frontend)                        │
│  ALB (Application Load Balancer)               │
│  DynamoDB / RDS                                │
└────────────────────────────────────────────────┘
→ 관리 포인트 多, 높은 고정 비용

[After] AWS Lightsail (단일 VPS)
┌─────────────────────────────────┐
│  Docker Compose                 │
│  ├── nginx  (FE + 역방향 프록시)│
│  ├── backend (FastAPI/Uvicorn)  │
│  ├── redis                      │
│  └── postgresql                 │
└─────────────────────────────────┘
→ 단순한 구성, 고정 월정액, 예측 가능한 비용
```

**트레이드오프 인식**

| 항목 | ECS | Lightsail |
|------|-----|-----------|
| 가용성 | 멀티 AZ 가능, 오토스케일링 | 단일 서버 (SPOF) |
| 비용 | 리소스 사용량 비례 (소규모엔 고정비 有) | 고정 월정액 |
| 관리 복잡도 | 태스크 정의, 서비스, ALB 등 | Docker Compose 하나 |
| 적합 규모 | 트래픽 변동 큰 서비스 | 트래픽 예측 가능한 소규모 서비스 |

소규모 게임 커뮤니티 서비스에서 단일 장애점은 감수 가능한 트레이드오프로 판단하여 전환했다.

---

## 7. 주요 API 엔드포인트

| Method | Path | 설명 |
|--------|------|------|
| `GET` | `/api/rooms` | 활성 방 목록 (캐시 10s) |
| `GET` | `/api/rooms/all` | 전체 방 + Phantom Room (매칭 탐지 포함) |
| `GET` | `/api/leaderboard` | 상위 N명 리더보드 (캐시 60s) |
| `GET` | `/api/players/{npid}` | 플레이어 상세 (온라인 상태 + 통계) |
| `GET` | `/api/history/hourly` | 시간대별 활성 유저 추이 |
| `GET` | `/api/history/daily` | 일별 피크/평균 요약 |
| `GET` | `/api/history/weekly-top` | 주간 최다 플레이어 |
| `GET/POST` | `/api/community/posts` | 게시판 CRUD |

---

## 8. 배운 점 & 성장

| 경험 | 구체적 내용 |
|------|-------------|
| **프로토콜 분석** | 공개된 C++ 소스를 읽고 Python 클라이언트를 처음부터 역설계. struct, ssl, socket 저수준 API 직접 활용 |
| **비동기 아키텍처** | FastAPI + asyncio + asyncpg 환경에서 이벤트 기반 설계로 응답 지연 없이 히스토리 수집 |
| **DB 설계 판단** | 데이터 특성(빈도, 스키마 유연성, 보존 기간)에 따라 Redis / PostgreSQL / DynamoDB를 선별 적용 |
| **알고리즘 설계** | 스냅샷 Diff + TTL 만료로 관찰 불가능한 게임 상태를 추론하는 알고리즘 직접 고안 |
| **클린 아키텍처** | Ports & Adapters 패턴으로 DB 구현체를 교체 가능하게 설계 (Community: DynamoDB ↔ PostgreSQL) |
| **CI/CD 자동화** | GitHub Actions로 PR → 통합 테스트, 태그 → ECR 빌드 + ECS 배포까지 완전 자동화 |
| **보안 설계** | AWS OIDC로 IAM 액세스 키 없이 배포 구현 — 자격증명을 Secrets에 저장하지 않는 keyless 방식 |
| **배포 안정성** | `wait-for-service-stability` + E2E 게이트로 배포 실패를 워크플로우 레벨에서 즉시 감지 |
| **비용 최적화** | CloudWatch + AWS Billing으로 오버스펙 인식 → Lightsail 단일 서버로 전환, 서비스 규모에 맞는 인프라 선택 판단 |
