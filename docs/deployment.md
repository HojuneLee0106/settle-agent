# 배포 · CI/CD


- **CI** (`.github/workflows/ci-cd.yml`) — `develop`·`master` 대상 PR 에서 실행.
  변경된 서비스만 빌드하고, AI 는 `pytest` 를 돌린 뒤 컨테이너를 띄워 `/health`
  스모크 테스트까지 한다. `CodeRabbit` 이 같은 PR 에 리뷰를 단다(`.coderabbit.yaml`).
- **브랜치 정책** — `master` 로는 동일 저장소의 `develop` 브랜치만 PR 을 보낼 수 있다.
  기본 작업 브랜치는 `develop`, `master` 는 릴리스 브랜치다.
- **CD** (`cd-production.yml`) — `master` push 시 이미지 3개를 GHCR 에 올리고,
  SSM 으로 EC2 에 `compose.selfhost.yml`·`Caddyfile`·`deploy/remote-deploy.sh` 를
  실어 보내 pull·기동시킨다. 인스턴스에서 빌드하지 않는다.
  배포 검증은 인스턴스 내부와 **공인 DNS 양쪽**에서 한다 — 내부 `--resolve` 만으로는
  외부 접속을 증명하지 못한다.
- **롤백** — `workflow_dispatch` 에 이전 커밋 SHA 를 `image_tag` 로 넣으면
  빌드를 건너뛰고 그 이미지로 되돌린다.

배포 인프라는 EC2 한 대다. Postgres(pgvector)도 같은 박스의 컨테이너로 돌고,
매니지드 의존은 S3 하나뿐이다. 자세한 것은 `compose.selfhost.yml` 주석 참고.

Postgres 가 EC2 위의 컨테이너이므로 저장 암호화는 RDS 설정이 아니라 **EBS 볼륨
암호화**다. 기존 볼륨에는 제자리 적용이 안 되고 스냅샷 교체가 필요하다 —
절차와 남은 항목은 [deploy/aws-hardening.md](../deploy/aws-hardening.md) 에 있다.
