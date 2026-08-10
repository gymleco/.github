# 보안 체크리스트

기획서 §17 을 실행 가능한 형태로 옮긴 것. 배포 전에 하나씩 직접 확인한다.
"설정했다"가 아니라 **"확인했다"** 가 기준이다.

---

## 0. gitleaks 설치 (Phase 0 — 지금)

이 PC 에는 winget 도 scoop 도 없으므로 아래 중 하나를 쓴다.

### 방법 A — 바이너리 직접 설치 (권장)

1. <https://github.com/gitleaks/gitleaks/releases/latest> 에서
   `gitleaks_<버전>_windows_x64.zip` 을 받는다.
2. 압축을 풀어 `gitleaks.exe` 를 PATH 에 있는 폴더로 옮긴다.
   예: `C:\Users\<사용자>\bin\` 을 만들고 사용자 환경변수 PATH 에 추가.
3. 확인:

```bash
gitleaks version
```

### 방법 B — Docker 로 실행 (설치 없이)

Docker 가 이미 있으므로 훅을 이렇게 실행할 수 있다.

```bash
GITLEAKS_DOCKER=1 git commit -m "..."
```

매번 붙이기 번거로우면 셸 프로파일에 `export GITLEAKS_DOCKER=1` 을 넣는다.
첫 실행 시 `zricethezav/gitleaks` 이미지를 내려받는다.

### 훅 활성화

```bash
git config core.hooksPath .githooks
```

### 동작 확인 (반드시 해볼 것)

가짜 키를 하나 넣고 커밋이 실제로 막히는지 본다.
설정만 하고 확인하지 않으면 훅이 도는지 알 수 없다.

저장소 안에서 실행한다 (스캔은 스테이징된 변경만 본다).

```bash
printf 'AWS_SECRET_ACCESS_KEY=wJalrXUtnFEMIK7MDENGbPxRfiCYEXAMPLEKEY\n' > leak-test.txt
git add leak-test.txt
git commit -m "hook test"    # ← 차단되어야 정상
```

커밋이 통과했다면 훅이 돌지 않고 있는 것이다. `core.hooksPath` 설정과
`.githooks/pre-commit` 의 줄바꿈(LF 여야 한다)을 확인한다.

정리:

```bash
git reset leak-test.txt && rm leak-test.txt
```

---

## 1. 네트워크

- [ ] 보안그룹: 443 만 전체 개방, 80 은 리다이렉트용
- [ ] SSH(22) 는 내 IP `/32` 만 — 또는 **SSM Session Manager 로 22 를 완전히 닫음**
- [ ] DB(5432) · 앱(8080) 외부 차단
- [ ] `PermitRootLogin no`, `PasswordAuthentication no`
- [ ] fail2ban 설치 및 **밴 기록이 실제로 쌓이는지** 확인
- [ ] Elastic IP 연결

## 2. 전송

- [ ] HTTPS 적용, HTTP → HTTPS 301
- [ ] 인증서 자동 갱신 — `certbot renew --dry-run` 통과
- [ ] TLS 1.2 이상만 허용, `server_tokens off`
- [ ] **보안 헤더를 Vercel 쪽(`next.config.ts` / `middleware.ts`)에 설정**
      ← nginx 헤더는 API 응답에만 걸린다. 방문자 HTML 에는 적용되지 않는다.
- [ ] CSP 를 `Report-Only` 로 먼저 관찰 → 위반 없음 확인 후 적용
- [ ] CSP nonce 방식 적용 (App Router 는 인라인 스크립트를 쓴다)
- [ ] securityheaders.com 에서 **실제 공개 도메인**을 찍어 A 등급

## 3. 인증

- [ ] BCrypt 저장, 최소 12자, 흔한 비밀번호 차단
- [ ] 로그인 시도 제한 — **IP 기준 1차**
      (계정 전면 잠금만 두면 공격자가 대표님을 상시 차단할 수 있다)
- [ ] JWT 를 HttpOnly + Secure + SameSite=Strict 쿠키로 발급
- [ ] 쿠키 `Domain` 이 사이트/API 공통 부모 도메인으로 설정됨
- [ ] **CSRF 토큰 활성화** (`CookieCsrfTokenRepository`)
      ← 쿠키 인증이면 필수. SameSite 하나에만 의존하지 않는다.
- [ ] Refresh Token DB 저장 + 로그아웃 시 무효화
- [ ] 새 IP 로그인 시 대표님 이메일 알림
- [ ] (권장) TOTP 2FA

## 4. 애플리케이션

- [ ] CMS 에디터에 `<script>alert(1)</script>` 를 실제로 입력해보고
      **저장 후 공개 페이지에서 실행되지 않는지** 눈으로 확인
- [ ] 업로드: 확장자 화이트리스트 + MIME + **매직 넘버** 검증
- [ ] 업로드 이미지 **서버 재인코딩** 후 저장
- [ ] 파일명 UUID 재생성, 용량 제한
- [ ] 이미지는 S3 + CloudFront 서빙 (앱 서버가 파일을 서빙하지 않음)
- [ ] 모든 관리 API 에 `@PreAuthorize` — **엔드포인트를 세면서** 확인
- [ ] 동적 정렬 컬럼은 허용목록 검증 (`ORDER BY ${sort}` 금지)
- [ ] CORS 명시적 화이트리스트, 와일드카드 없음
- [ ] 문의 폼 rate limit + honeypot

## 5. 데이터

- [ ] 앱 DB 계정에 DROP/CREATE 권한 없음
- [ ] 전화번호 컬럼 AES-256 암호화 (랜덤 IV)
- [ ] 검색용 블라인드 인덱스(HMAC) 별도 컬럼 — 결정적 암호화 사용 안 함
- [ ] **CSV 내보내기 시 감사 로그 기록** ← 컬럼 암호화를 무력화하는 통로다
- [ ] 매일 자동 백업 + 별도 위치 보관
- [ ] **복구 테스트 1회 실제 수행** (복구해본 적 없는 백업은 백업이 아니다)
- [ ] `.env` gitignore, gitleaks 훅 동작 확인
- [ ] 운영 시크릿은 SSM Parameter Store 또는 파일 권한 600

## 6. 법적 (개인정보보호법)

- [ ] 개인정보처리방침 페이지 게시 (수집항목·목적·보유기간·파기절차·보호책임자)
- [ ] 문의 폼 수집·이용 동의 체크박스 — 기본 체크 해제 상태
- [ ] 마케팅 활용 동의는 **별도 항목으로 분리**
- [ ] `purge_at` 기반 자동 파기 배치 동작 확인
- [ ] 유출 신고 기준을 클라이언트와 공유
      — 외부의 불법적 접근에 의한 유출은 **건수와 무관하게** 신고 대상이며
        기한은 72시간. "1천명 이상"만 해당한다는 인식은 위험하다.
        (최종 문구는 개인정보보호위원회 안내자료로 확인)
- [ ] 개인정보 책임 주체를 계약서에 명시

## 7. 운영

- [ ] `unattended-upgrades` 활성화
- [ ] UptimeRobot 5분 간격 모니터링
- [ ] **AWS Billing Alert** — 요금 급증은 서버가 악용되고 있다는 신호
- [ ] 디스크 80% 알림
- [ ] 관리자 로그인/변경 이력 로깅 (`admin_audit_log`)
- [ ] 로그에 비밀번호·토큰·개인정보가 남지 않는지 확인
- [ ] 인수인계 문서 + 5분 사용 영상

## 8. 성능 · SEO

- [ ] Lighthouse 모바일 성능·접근성 90+
- [ ] 첫 화면 2.5초 이내
- [ ] `prefers-reduced-motion` 대응
- [ ] JS 비활성 상태에서도 텍스트가 DOM 에 존재
- [ ] 모바일 대체 연출 — 중급 안드로이드 실기기 테스트
- [ ] 메타·OG, sitemap.xml, robots.txt (`/admin` 차단)
