# Phase 1 실행계획 — FastAPI 골격 + 인증(JWT) + Alembic

> **상위 문서**: PLAN.md §2(백엔드), §5(users), §6(인증 API), §8(시크릿)
> **기간(현실안)**: 2주
> **완료 기준**: `/api/auth/*` 통과, Alembic 마이그레이션 동작
> **성격**: 모든 후속 백엔드(업로드·워커·결과)가 올라설 **토대** 구축.

---

## 0. 목표 한 줄

SkinLens 엔진을 **별도 언어 브리지 없이 직접 import**할 수 있는 Python(FastAPI) 백엔드 골격과
JWT 인증을 세우고, 스키마 변경을 Alembic으로 버전 관리한다.

---

## 1. 선행 조건

- [ ] Python 3.10+ 환경 (WSL2 권장 — 부록 A)
- [ ] PostgreSQL 접근(로컬 docker 또는 관리형 Supabase/Neon 개발 인스턴스)
- [ ] `.env`/`.gitignore` 규칙 합의 (시크릿 평문 금지 — §8)

---

## 2. 작업 단위

### 2.0 의존성 설치
```bash
pip install fastapi uvicorn[standard] sqlalchemy alembic pydantic-settings passlib[bcrypt] python-jose[cryptography] psycopg2-binary
```

### 2.1 프로젝트 골격
- [ ] 디렉토리 구조 확정 (예시)
  ```
  backend/
    app/
      main.py            # FastAPI 앱 + 라우터 등록
      core/config.py     # 설정(Pydantic Settings, env 참조)
      core/security.py   # JWT 발급/검증, 비밀번호 해시
      db/session.py      # SQLAlchemy 2.x 엔진/세션
      db/base.py         # 모델 베이스
      models/user.py     # users 테이블 ORM
      schemas/auth.py    # Pydantic v2 요청/응답 DTO
      api/auth.py        # /api/auth/* 라우터
      repositories/      # DB 접근 계층
    alembic/             # 마이그레이션
    .env                 # (gitignore) 평문 자격증명 금지 대상
  ```
- [ ] `app.main`에 `/api/v1` 버전 프리픽스 도입(권장 — §6 보완)
- [ ] 헬스체크 엔드포인트 `GET /healthz`

### 2.2 설정 / 시크릿 (§8 — 우선순위 상)
- [ ] `Pydantic Settings`로 환경변수 로드 (`DATABASE_URL`, `JWT_SECRET_KEY`, `JWT_ALGORITHM`, `ACCESS_TOKEN_EXPIRE_MINUTES`)
- [ ] `DEBUG` 기본 `false`, 운영에서 절대 `true` 금지
- [ ] **시크릿 평문 금지**: 코드/compose/배포 산출물에 자격증명 인라인 금지
  > ⚠️ 과거 SkinLens/SkyPredictor 배포 zip에 라이브 자격증명 포함이 반복 적발됨 — 같은 실수 반복 금지

### 2.3 DB & ORM (users)
- [ ] SQLAlchemy 2.x 스타일 모델 정의 (`users` — PLAN.md §5)
  - `id UUID PK`, `email UNIQUE NOT NULL`, `password_hash`, `name`, `created_at`, `updated_at`
- [ ] 세션 의존성(`get_db`) 작성
- [ ] (선택) `consents` 테이블 스텁만 미리 정의 — PIPA는 Phase 8에서 흐름 완성

### 2.4 Alembic
- [ ] `alembic init` + `env.py`에 모델 메타데이터/`DATABASE_URL` 연결
- [ ] 첫 마이그레이션 자동생성 (`alembic revision --autogenerate`) → `users` 테이블
- [ ] `alembic upgrade head` / `downgrade` 왕복 동작 확인

### 2.5 인증 (JWT)
- [ ] 비밀번호 해시: `passlib[bcrypt]`
- [ ] 토큰: `python-jose`, access token 만료 = **30분**(PHASE0_PLAN.md 쿠키 maxAge와 일치)
- [ ] 엔드포인트 구현 (§6)
  - `POST /api/v1/auth/register` — 이메일 중복 검사 → 해시 저장
  - `POST /api/v1/auth/login` — 검증 → `{ access_token, user }` 반환
  - `POST /api/v1/auth/logout` — (서버 무상태면 클라/쿠키 폐기 안내; v2에서 블랙리스트)
  - `GET /api/v1/auth/me` — Bearer 토큰 → 현재 사용자
- [ ] 보호 의존성(`get_current_user`)으로 인증 가드

### 2.6 표준화(시작 시점에 박아두면 이득)
- [ ] 표준 에러 응답 스키마 + 예외 핸들러
- [ ] 요청/응답 예시를 OpenAPI(docstring/`examples`)에 명시 (§6 보완)
- [ ] CORS 화이트리스트(개발: localhost:3000)

---

## 3. 산출물

1. 실행 가능한 FastAPI 앱 (`uvicorn app.main:app --reload`)
2. `users` 테이블 + 동작하는 Alembic 마이그레이션
3. `/api/v1/auth/register|login|logout|me` 4종
4. `.env.example`(키 목록만, 값 없음)
   ```bash
   DATABASE_URL=postgresql://user:password@localhost:5432/skyweb
   JWT_SECRET_KEY=your-secret-key-here
   JWT_ALGORITHM=HS256
   ACCESS_TOKEN_EXPIRE_MINUTES=30
   DEBUG=false
   ```

---

## 4. 완료 기준 (Definition of Done)

- [ ] `register → login → me` 플로우가 실제 DB로 통과 (수동 또는 pytest)
- [ ] `alembic upgrade head`로 빈 DB에서 스키마 재현 가능
- [ ] OpenAPI 문서(`/docs`)에서 4개 엔드포인트 확인
- [ ] 코드/compose 어디에도 평문 시크릿 없음(검색으로 확인)
- [ ] (선택) 기본 인증 흐름 pytest 작성

### 테스트 전략
- conftest.py에 테스트 DB 환경 픽스처
- 테스트용 DATABASE_URL 분리
- 테스트 후 트랜잭션 롤백

---

## 5. 리스크 / 주의

- **IDOR 방지**: 인증 없는 보호 엔드포인트 금지(§8). `get_current_user` 일관 적용.
- **토큰 만료 정합**: 30분 만료를 프론트 쿠키 maxAge(PHASE0_PLAN.md Day 15-18)와 맞춤. v2에서 refresh 패턴.
- **마이그레이션 위생**: autogenerate 결과는 항상 사람이 검수(특히 타입/인덱스).
- **테스트 격리**: 과거 `EnvInjector.restore()`가 `JWT_SECRET_KEY`를 삭제해 무관 테스트가 깨진 전례 → conftest 환경 픽스처를 깔끔하게.

---

## 6. 다음 단계 연결

- Phase 2(업로드)는 이 앱에 `/api/v1/analyses` 라우터와 `analyses` 모델을 추가한다.
- Phase 4(프론트)의 BFF 로그인 라우트가 여기 `/api/v1/auth/login`을 호출한다 → **응답 형태(`{access_token, user}`) 계약 고정**.

---

## 7. PWA 통합 고려사항 (백엔드 관점)

Phase 1은 백엔드 단계이지만, PWA 통합을 위해 다음 사항을 고려합니다.

### PWA와 백엔드 통합 포인트
- **HTTPS 필수**: PWA는 HTTPS 환경에서만 동작하므로 백엔드는 HTTPS 강제 설정 필요
- **CORS 설정**: PWA가 다른 도메인에서 실행될 경우를 대비한 CORS 화이트리스트 구성
- **푸시 알림 준비**: Phase 4 이후 푸시 알림 구현을 위한 백엔드 엔드포인트 설계 여지
- **오프라인 지원**: PWA의 오프라인 캐싱과 백엔드 API의 일관성 유지

### 검증 체크리스트 (백엔드)
- [ ] HTTPS 강제 설정 (`SECURE_SSL_REDIRECT`, `HSTS`)
- [ ] CORS 화이트리스트 설정 (개발: localhost:3000, 운영: 도메인)
- [ ] API 응답 캐싱 헤더 고려 (PWA Service Worker 캐싱과 정합)
- [ ] 푸시 알림용 엔드포인트 설계 (Phase 4 이후 구현)

> 실제 PWA 구현은 Phase 4(프론트)에서 시작됩니다.
