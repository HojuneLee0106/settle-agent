# 주요 엔드포인트

**AI** (`ai/app/main.py`)

| 메서드 | 경로 | |
|---|---|---|
| GET | `/health` | `persistent`, `rag`, `sources`(근거 자료 판) 포함 |
| POST | `/api/session` | 세션 생성·이어받기·리셋 |
| POST | `/api/profile/extract-upload` | 신분증 업로드 → OCR → 프로필 |
| POST | `/api/profile/confirm` | 확인 화면에서 고친 값 반영 |
| POST | `/api/chat` · `/api/chat/stream` | 대화 (스트림은 SSE 로 진행 단계 전송) |
| POST | `/api/actions/{id}/start` · `/preview` · `/approve` | 액션 시작 · 서류 생성 · 승인 |
| GET | `/api/state` · `/api/ledger` | 새로고침 복원 · 처리 이력 |
| GET | `/api/documents/{id}/preview` · `/api/documents/{id}.pdf` | 생성 서류 |

**Backend**

| 메서드 | 경로 | |
|---|---|---|
| POST | `/api/v1/auth/signup` · `/api/v1/auth/login` | JWT 발급 |
| POST | `/api/v1/uploads` | S3 presigned PUT URL |
| POST | `/api/v1/documents/extractions` | `uploadId` 로 OCR 중계 |
| POST | `/api/session` · `/api/chat` · `/api/chat/stream` · `/api/actions/{id}/start` | AI 중계 |
| POST | `/api/actions/{id}/preview` · `/api/actions/{id}/approve` | 서류 생성 후 영속화 |
| GET | `/api/ledger` · `/api/documents/{id}/preview` · `/api/documents/{id}/download` | |

모든 에이전트 응답은 `{ schema_version, reply, ui, state }` 봉투다.
오류는 `{ detail: { error, message, details } }`.
계약은 `ai/app/api/schemas.py` 에 있고 **필드 추가만 허용, 삭제·개명 금지**다.
