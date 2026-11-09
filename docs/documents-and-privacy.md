# 서류 생성과 개인정보

## 서류 생성

`ai/app/nodes/doc_builder.py` — **LLM 을 쓰지 않는다.** 값 생성은 전부 결정적 매핑이다.

```
profile + mappings/*.yaml → Jinja2 (templates/*.html) → WeasyPrint → PDF
```

- 렌더에는 마스킹하지 않은 평문 프로필을 쓴다 (서버 내부에서만 생성).
  마스킹은 **응답을 만들 때만** 적용되고 state 원본은 항상 평문이다.
  그래서 무엇을 얼마나 들고 있는지가 중요하다 — 아래 "개인정보" 절 참고.
- OCR 신뢰도 0.95 미만 필드는 서류와 화면 양쪽에 '확인요망'으로 표시된다.
- WeasyPrint 는 Pango/cairo 를 dlopen 한다. 시스템 라이브러리가 없으면 PDF 생성이
  조용히 실패하고 HTML 만 남는다 (`ai/Dockerfile` 참고).

## 개인정보

무엇을 들고 있고 언제 버리는지.

| 데이터 | 어디에 | 언제 사라지나 |
|---|---|---|
| 신분증 원본 사진 | S3 `members/{id}/uploads/` | **OCR 직후 즉시 삭제** (`FileService.discardOriginal`) |
| OCR 원문 | — | **저장하지 않는다.** 추출 함수 밖으로 나가지 않는다 |
| 프로필 · 대화 | 체크포인터 (Postgres) | 보관기간 스크립트로 정리 (아래) |
| 생성 서류 PDF | S3 `members/{id}/generated-documents/` | 남는다 — 서류함은 회원 자산이다 |
| 임시 렌더 결과 | `ai/output/*.html`, `*.pdf` | 아직 정리하지 않는다 |

- **응답에 예외 메시지를 싣지 않는다.** OCR·프로필 처리 중 난 예외에는 신분증에서
  읽은 값이 섞일 수 있다. 클라이언트에는 예외 **타입만** 나가고, 원인은 서버
  로그의 traceback 에만 남는다.
- 외국인등록번호는 응답에서 마스킹된다(`990101-*******`). state 원본은 평문이므로
  저장 매체 암호화가 따로 필요하다 — `deploy/aws-hardening.md`.

```bash
uv run python scripts/purge_stale_sessions.py                # 30일 이상 방치된 세션
uv run python scripts/purge_raw_texts.py                     # 옛 체크포인트의 OCR 원문 (1회)
```

둘 다 기본이 dry-run 이다. `--apply` 를 붙여야 실제로 지운다.

세션 보관기간은 **마지막 활동 기준 30일**이다. 생성 기준이 아니라 활동 기준인
이유는, 90일 기한을 추적하는 앱에서 나이만으로 지우면 계속 쓰는 사람의 D-day 가
매번 리셋되기 때문이다.

AWS 쪽에서 해야 하는 것(버킷 기본 암호화, 퍼블릭 차단, EBS 암호화)은
[deploy/aws-hardening.md](../deploy/aws-hardening.md) 에 모아 두었다.
