# Phase 8 실행계획 — 테스트 · 보안 점검 · 배포

> **상위 문서**: PLAN.md §8(보안·개인정보·규제), §9(인프라), §11(보완 체크리스트)
> **기간(현실안)**: 2주
> **완료 기준**: 운영 배포 + 비밀키 점검 + PIPA 동의 흐름
> **성격**: 출시 게이트. 특히 **얼굴 이미지 = 민감정보** 대응은 출시 전 필수.

---

## 0. 목표 한 줄

엔진 호출을 모킹한 테스트로 회귀를 막고, 시크릿·개인정보·규제를 점검한 뒤,
프론트/백엔드/워커를 운영 환경에 배포한다.

---

## 1. 선행 조건

- [ ] Phase 1~7 기능 완성(E2E 플로우 동작)
- [ ] 운영 인프라 계정 확보: Vercel / API 호스트(Render·Railway·VM) / 워커 호스트 / 관리형 PostgreSQL / Redis / S3·R2
- [ ] PIPA 동의·파기 정책 초안(§8) — 법적 확인 필요

---

## 2. 작업 단위

### 2.0 의존성 설치
```bash
pip install pytest pytest-asyncio pytest-cov  # 테스트
pip install python-dotenv  # 환경 변수
# 배포 도구는 각 플랫폼별로 다름 (Vercel CLI, Docker 등)
```

### 2.1 테스트 (§11 진행 중)
- [ ] **엔진 호출 모킹**: `AnalysisService.analyze()`를 모킹해 웹 계층만 테스트(무거운 모델 없이 CI 가능)
- [ ] 백엔드 단위/통합: auth, 업로드(검증·IDOR), enqueue, 결과 조회
- [ ] 워커 태스크: 성공/실패/멱등성(모킹 엔진)
- [ ] 프론트: 인증 플로우·업로드·결과 렌더(핵심 경로 E2E)
- [ ] 테스트 격리 위생: 전역 `sys.modules`/환경변수 오염 방지(과거 conftest 이슈 교훈)

### 테스트 전략 상세
- **백엔드**: pytest + pytest-asyncio, 엔진 mock 사용, 테스트 DB 분리
- **워커**: Celery 태스크 동기 실행 테스트 (`task.apply()`), 모킹 엔진
- **프론트**: Playwright 또는 Cypress E2E, 핵심 경로만 (가입→업로드→결과)
- **CI**: GitHub Actions, 엔진 모킹으로 무거운 의존성 없이 CI 통과

### 2.2 보안 점검 (§8 — 우선순위 상)
- [ ] **시크릿 점검(중요)**: 코드/compose/배포 산출물(zip·이미지)에 평문 자격증명 0건 — 전체 검색으로 확인
  > ⚠️ 과거 배포물에 라이브 자격증명 반복 적발 — 출시 전 반드시 0건 확인
- [ ] compose는 `${VAR}` 참조만, 실제 값은 `.env`(gitignore) 또는 시크릿 매니저
- [ ] `DEBUG=false`(운영), HTTPS 강제, CORS 화이트리스트(운영 도메인)
- [ ] Rate Limiting, 입력 검증(Pydantic), 업로드 타입/크기 검증
- [ ] 서명된 S3/R2 URL만 노출(직접 공개 금지), 인증 없는 작업 엔드포인트 0건(IDOR/Path traversal 점검)

### 2.3 PWA 검증 테스트 (Phase 5: 검증 테스트)
- [ ] **Lighthouse PWA 점수 90+ 달성**: Chrome DevTools → Lighthouse → Progressive Web App
- [ ] 다양 브라우저 호환성 테스트 (Chrome, Safari, Edge)
- [ ] 오프라인/온라인 전환 시나리오 테스트
- [ ] PWA 설치 경험 테스트 (beforeinstallprompt 이벤트)
- [ ] 푸시 알림 동작 테스트 (권한 요청, 수신, 클릭)
- [ ] 오프라인 캐싱 정책 검증 (정적 리소스, 결과 데이터)
- [ ] HTTPS 환경에서 PWA 설치 가능 확인

### 2.4 개인정보 / 규제 (§8 — 출시 전 결정)
- [ ] **PIPA 우선**: 얼굴 이미지 = 생체/민감정보 가능성 → 처리 흐름 정비
  - [ ] `consents` 테이블 활용: 얼굴 이미지(민감정보) 처리 **별도 명시 동의** 수집·기록
  - [ ] 가입/업로드 시 동의 UI + `consented_at` 저장
  - [ ] 암호화 저장, 보존기간(`retention_until`) + **자동 파기** 배치/잡
  - [ ] 계정 삭제(`DELETE /api/v1/user/account`) 시 관련 이미지/분석 파기 검증
- [ ] **의료기기/의료행위 회피**: 전 화면 문구를 "분석/측정/추정"으로 통일, "진단" 0건 점검
- [ ] 처리방침·이용약관 게시(§11 프로덕션 전)

### 2.5 배포 (§9)
- [ ] **프론트**: Vercel(Next.js) — 환경변수(`API_URL` 등) 설정
  ```bash
  # .env.production (프론트)
  NEXT_PUBLIC_API_URL=https://api.skyweb.com
  ```
- [ ] **API**: 단일 컨테이너(경량 — PyTorch 미포함)
  ```bash
  # .env (백엔드)
  DATABASE_URL=postgresql://...
  REDIS_URL=redis://...
  S3_BUCKET_NAME=...
  AWS_ACCESS_KEY_ID=...
  AWS_SECRET_ACCESS_KEY=...
  JWT_SECRET_KEY=...
  DEBUG=false
  ```
- [ ] **워커**: 메모리 여유 호스트(PyTorch+복원모델) — 사양 **실측 기반** 결정, 월 실비 추정
  ```bash
  # .env (워커)
  CELERY_BROKER_URL=redis://...
  CELERY_RESULT_BACKEND=redis://...
  SKINLENS_ENGINE_VERSION=1.0.0
  WORKER_CONCURRENCY=2
  ```
- [ ] **DB/Redis/스토리지**: 관리형 연결 + 백업 설정
- [ ] **CI/CD**: GitHub Actions(테스트 → 빌드 → 배포)
- [ ] 운영 모니터링/로깅 표준(에러 추적, per-execution 로그)

### 2.6 출시 점검
- [ ] 성공 지표 수치 확정(§1): 업로드→결과 P95 < N초, 실패율 < N%
- [ ] 스모크 테스트(운영): 가입→업로드→결과→히스토리 1회 관통
- [ ] 롤백 계획(간단) + 장애 시 연락/대응 메모

---

## 3. 산출물

1. 모킹 기반 테스트 스위트(CI 통과)
2. 보안 점검 결과(시크릿 0건·IDOR 0건 체크리스트)
3. PIPA 동의·파기 흐름(동의 UI + 보존/파기 잡)
4. 운영 배포(프론트/API/워커/DB/Redis/스토리지) + CI/CD
5. PWA 검증 테스트 결과 (Lighthouse 점수 90+, 브라우저 호환성, 오프라인 지원)

---

## 4. 완료 기준 (Definition of Done)

- [ ] 운영 환경에서 가입→업로드→결과→히스토리 E2E 성공
- [ ] 시크릿 평문 0건, 인증 없는 보호 엔드포인트 0건 확인
- [ ] 얼굴 이미지 처리 동의 흐름 동작 + 보존기간 만료 파기 동작
- [ ] 전 화면 "진단" 표현 0건
- [ ] CI에서 엔진 모킹 테스트가 무거운 모델 없이 통과
- [ ] Lighthouse PWA 점수 90+ 달성
- [ ] 다양 브라우저에서 PWA 설치/동작 확인
- [ ] 오프라인/온라인 전환 시나리오 테스트 통과

---

## 5. 리스크 / 주의

- **시크릿 누출(최우선)**: 반복된 전례. 배포물 전수 검색 없이는 출시 금지.
- **PIPA 미흡 출시 위험**: 민감정보를 동의·암호화·파기 체계 없이 다루면 법적 리스크.
- **워커 비용**: 워커 호스트가 비용의 핵심 변수. 보급형 저메모리 티어 OOM 주의 → 실측 후 사양 확정.
- **표현 규제**: "진단" 한 단어가 의료기기 규제 영역을 건드릴 수 있다 → 전수 점검.
- **테스트 신뢰성**: 전역 상태 오염으로 인한 거짓 통과/실패 방지(conftest 위생).

---

## 6. 다음 단계 연결 (v1 이후)

- v2: 소셜 로그인, refreshToken(무중단 갱신), 정교한 추천, SSE/WebSocket 고도화
- 부록 C(스케일링): Kubernetes·워커 수평 확장·Blue-Green/Canary·읽기 복제본 — **필요해질 때** 도입
