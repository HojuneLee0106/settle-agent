# 개발 가이드

## 환경 변수

**ai** (`ai/.env.example`)

| 키 | 없으면 |
|---|---|
| `ANTHROPIC_API_KEY` | LLM 기능이 전부 꺼진다. 질의응답도, 응답·오류의 다국어 변환도 안 되고 화면이 항상 한국어로 나온다 |
| `ANTHROPIC_MODEL` | `claude-sonnet-4-5` |
| `DATABASE_URL` | 벡터 검색이 죽고 BM25 단독으로 동작한다. 세션도 휘발된다 |
| `CLOVA_INVOKE_URL` / `CLOVA_SECRET_KEY` | 신분증 OCR 불가 |
| `RESEARCH_ENABLED` | 기본 `1`. `0` 이면 도구 루프를 끄고 단발 RAG 로 되돌린다 |

**backend** (`backend/server/.env.example`)

`DB_URL` · `DB_USERNAME` · `DB_PASSWORD` · `JWT_SECRET` · `JWT_ACCESS_TOKEN_TTL_SECONDS`
· `AWS_REGION` · `AWS_S3_BUCKET` · `AI_BASE_URL` · `AI_CONNECT_TIMEOUT_SECONDS` · `AI_READ_TIMEOUT_SECONDS`

**frontend** — `VITE_API_BASE_URL` (기본 `/api`), `VITE_USE_MOCK`

## 테스트

```bash
cd ai && uv run pytest
```

```bash
cd backend/server && ./gradlew test
```

검색이 정상인지 확인:

```bash
cd ai && DATABASE_URL=postgresql://settle:settle@localhost:5432/settle uv run python eval/retrieval_check.py
```

정답 문서를 하나로 특정하기는 어려우므로, 무너지면 반드시 깨지는 것만 본다 —
내 자격 문서가 남의 자격 문서에 밀리지 않는가.

## 디렉터리

```
ai/
  app/agent/       LangGraph 상태 기계 (graph · service · state)
  app/nodes/       planner · profiler · doc_builder · qa · researcher
  app/extractors/  외국인등록증 · 여권 OCR (CLOVA)
  app/tools/       llm · embed · rag_store · progress
  rules/           체류자격 매트릭스 · 근거법령 · 코퍼스
  mappings/        서식 필드 매핑
  templates/       서식 HTML
  scripts/         코퍼스 구축 · 임베딩 · 적재 · 보관기간 정리 · 자료 최신성 점검
backend/server/    Spring Boot — domain: auth · member · file · document · agent · action · profile · card · application
frontend/frontend-react/
                   React + Vite (App.jsx 단일 스텝 라우팅)
deploy/            Caddyfile · 배포 스크립트 · IAM·CORS 정책 · AWS 보안 설정 안내
mock-institution/  기관 API 시뮬레이터
seed/              샘플 신분증 이미지 · 프로필
```
