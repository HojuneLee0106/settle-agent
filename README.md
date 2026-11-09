# Settle Agent

국내 체류 외국인의 금융·행정 정착을 돕는 AI 에이전트.

신분증을 찍어 올리면 OCR 로 프로필을 만들고, 체류자격 룰로 **지금 무엇을 할 수 있고
무엇이 막혀 있는지**를 계산해 과제 목록으로 보여준다. 법령·안내매뉴얼을 검색해 근거와 함께
답하고, 통합신청서 같은 서식을 채워 PDF 로 만들어 준다. 되돌릴 수 없는 행동에는 승인 게이트가 걸린다.

```
외국인등록증·여권 촬영 → 프로필 확인 → 체류자격 판정 → 과제 그래프
                                              ↓
                            질의응답(법령·매뉴얼 RAG) · 서류 생성 → 승인 → 보관함
```

## 실행 방법

### 전체 스택 (Docker)

```bash
cp ai/.env.example .env.local
```

값을 채운 뒤(`ANTHROPIC_API_KEY`, `CLOVA_*` 등):

```bash
docker compose -f compose.local.yml up -d --build
```

- 앱 http://localhost:8081
- Swagger http://localhost:8081/swagger-ui.html
- 헬스체크 http://localhost:8081/health/ai · http://localhost:8081/health/backend

### 개별 실행 (개발)

```bash
docker compose up -d db
```

```bash
cd ai && cp .env.example .env && uv sync && uv run uvicorn app.main:app --reload --port 8000
```

```bash
cd backend/server && cp .env.example .env && ./gradlew bootRun
```

```bash
cd frontend/frontend-react && npm install && npm run dev
```

API 스펙: AI http://localhost:8000/docs · Backend http://localhost:8080/swagger-ui.html

`JWT_SECRET` 은 32바이트 이상이어야 한다. 백엔드가 아직 없어도 프론트는 목데이터로 돈다 —
`VITE_USE_MOCK=true npm run dev`.

## 아키텍처

```
                     ┌───────────────┐
브라우저 ──────────► │ Caddy (:8081) │  경로 라우팅
                     └───────┬───────┘
                  ┌──────────┼──────────┐
                  ▼          ▼          ▼
            frontend      backend    /health/*
          React + Vite  Spring Boot
            (nginx)     Java 21 · :8080
                            │  RestClient
                            ▼
                      ai · FastAPI + LangGraph · :8000
                            │
                            ▼
                      PostgreSQL + pgvector
```

| 서비스 | 스택 | 역할 | 포트 |
|---|---|---|---|
| `frontend/` | React 18 · Vite 5 · React Router | 모바일 프레임 단일 페이지. 촬영 → 확인 → 과제 → 서류 흐름 | 5173 (dev) / 3000 (nginx) |
| `backend/` | Java 21 · Spring Boot 3 · JPA | 회원·JWT 인증, S3 presigned 업로드, 생성 서류 영속화, AI 서비스 중계 | 8080 |
| `ai/` | Python 3.12 · FastAPI · LangGraph | 에이전트 상태 기계, OCR 추출, 룰 엔진, RAG 질의응답, 서식 렌더 | 8000 |
| `db` | pgvector/pgvector:pg16 | 회원·서류 테이블, LangGraph 체크포인트, `rag.chunk` 벡터 | 5432 |
| `deploy/` | Caddy 2 | 리버스 프록시 (`/api/*` → backend, 그 외 → frontend) | 8081 |
| `mock-institution/` | Python | 기관 API 시뮬레이터 (스켈레톤) | — |

프론트는 개발 시 Vite 프록시로 `/api/v1/*` 를 Spring 에, 나머지 `/api/*` 를 AI 에 직접 보낸다.
운영에서는 Caddy 가 `/api/*` 를 전부 backend 로 보내고 backend 가 AI 를 중계한다.

## 문서

| 문서 | 내용 |
|---|---|
| [에이전트 설계](docs/agent.md) | 상태 기계 · 위험 등급 · 룰 · 선행조건과 완료 |
| [검색과 근거 자료](docs/rag.md) | RAG 코퍼스 · 하이브리드 검색 · 자료 최신성 |
| [서류 생성과 개인정보](docs/documents-and-privacy.md) | PDF 렌더 · 데이터 보관과 삭제 |
| [API](docs/api.md) | AI · Backend 주요 엔드포인트 |
| [개발 가이드](docs/development.md) | 환경 변수 · 테스트 · 디렉터리 구조 |
| [배포 · CI/CD](docs/deployment.md) | CI · CD · 롤백 · 인프라 |
