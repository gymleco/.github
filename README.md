# .github

GYMLECO KOREA 조직의 공용 저장소입니다.

- **PR·이슈 템플릿** — 조직 내 모든 저장소에 자동 상속됩니다
- **설계 문서** — 두 코드 저장소가 공유하는 단일 출처
- **조직 프로필** — `profile/README.md` 가 조직 페이지에 표시됩니다

---

## 저장소 구성

| 저장소 | 내용 |
|---|---|
| [`FE`](../../../FE) | 공개 사이트(Next.js) + 관리자 CMS(Vite) |
| [`BE`](../../../BE) | API 서버 (Spring Boot 4 / Java 21) |
| `.github` | 이 저장소 — 템플릿과 설계 문서 |

---

## 문서

| 문서 | 내용 |
|---|---|
| [작업 계획](docs/01-project-plan.md) | 일정, 단계별 산출물, 리스크 |
| [인프라 구조](docs/02-infrastructure.md) | 배포 구성, 요청 경로, 비용 |
| [DB 구조](docs/03-database.md) | ERD, 개인정보 처리 방식, 트랜잭션 방침 |
| [화면 설계](docs/04-screens.md) | 사이트맵, 스크롤 구조, 문의 폼, 관리자 |
| [API 명세](docs/05-api-spec.md) | 엔드포인트, 인증 흐름, 개인정보 취급 |
| [미결 사항](docs/OPEN-DECISIONS.md) | 구조를 정하기 전에 확인 |
| [보안 체크리스트](docs/SECURITY-CHECKLIST.md) | 배포 전, 보안 코드를 건드릴 때 |
| [클라이언트 확인 항목](docs/PHASE0-CLIENT-QUESTIONS.md) | 대표님께 그대로 전달 가능 |

---

## 상속되는 것과 안 되는 것

조직 `.github` 저장소가 다른 저장소에 **자동으로 제공하는 것**:

- `PULL_REQUEST_TEMPLATE.md`
- `ISSUE_TEMPLATE/`
- `CONTRIBUTING.md`, `CODE_OF_CONDUCT.md`, `SECURITY.md`

**상속되지 않는 것** — 각 저장소에 따로 둬야 합니다:

- `.github/workflows/` (CI)
- `.github/dependabot.yml`
- `.gitleaks.toml`, `.githooks/`

이 파일들은 두 저장소에 복제돼 있습니다. **한쪽만 고치면 어긋납니다** —
보안 설정을 바꿀 때는 양쪽을 함께 수정하세요.
# .github
