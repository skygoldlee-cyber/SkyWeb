# Phase 4 실행계획 — Next.js 인증/레이아웃 + API 연동

> **상위 문서**: PHASE0_PLAN.md Day 15-21, PLAN.md §3, §6
> **기간(현실안)**: 3–4주
> **완료 기준**: 로그인 ~ 업로드 플로우 **E2E** 동작
> **성격**: Phase 0에서 익힌 패턴을 **실제 백엔드(Phase 1~3)에 연결**. 프론트 본격 구현 시작.

---

## 0. 목표 한 줄

httpOnly 쿠키 인증(BFF)을 갖춘 Next.js App Router 앱을 세우고, 로그인→레이아웃→이미지 업로드까지
실제 FastAPI 백엔드와 끝까지 연결한다.

---

## 1. 선행 조건

- [ ] Phase 0 완료(인증+목록 SPA 직접 구현 경험)
- [ ] Phase 1 완료(`/api/auth/*` 동작) + Phase 2 완료(`POST /api/analyses`)
- [ ] 백엔드 응답 계약 고정: 로그인 `{access_token, user}`, 업로드 `{analysis_id, status}`

---

## 2. 작업 단위

### 2.0 의존성 설치
```bash
npm install @tanstack/react-query react-hook-form @hookform/resolvers zod
npx shadcn@latest init
```

### 2.1 프로젝트 셋업
- [ ] `npx create-next-app@latest --typescript --tailwind --app`
- [ ] `npx shadcn@latest init` + 필요한 컴포넌트(`button card input form`) 추가
- [ ] `.env.local`: `API_URL`(서버 전용), `NEXT_PUBLIC_API_URL`(필요 시)
  ```bash
  API_URL=http://localhost:8000
  NEXT_PUBLIC_API_URL=http://localhost:8000
  ```
- [ ] 디렉토리: `app/`, `app/api/`(BFF), `components/`, `lib/`(zod 스키마·fetch 래퍼)

### 2.2 쿠키 기반 인증 (BFF) — Day 15-18
- [ ] `app/api/auth/login/route.ts` — FastAPI `/api/v1/auth/login` 호출 → `access_token`을 **httpOnly 쿠키**로 `Set-Cookie`
  - `httpOnly: true`, `secure`(prod), `sameSite: "lax"`, `path:"/"`, `maxAge: 60*30`
- [ ] `app/api/auth/register/route.ts` — FastAPI `/api/v1/auth/register` 프록시
- [ ] `app/api/auth/logout/route.ts` — 쿠키 삭제
- [ ] `AuthContext` — 현재 사용자 상태만 보관(토큰은 JS가 안 만짐)
- [ ] `middleware.ts` — 보호 라우트 가드(`matcher: ["/analyses/:path*", "/profile"]`)

### 2.3 레이아웃 / 공통
- [ ] `app/layout.tsx` — `<html lang="ko">`, 네비게이션, `Providers`(TanStack Query) 래핑
- [ ] 로그인 상태별 네비게이션(로그인/로그아웃 토글)
- [ ] `app/providers.tsx` — `QueryClientProvider`
- [ ] `loading.tsx` / `error.tsx` 공통 패턴

### 2.4 인증 화면
- [ ] `/auth/register`, `/auth/login` — react-hook-form + Zod(`LoginSchema`, `RegisterSchema`)
- [ ] 로그인 성공 → 보호 영역 리다이렉트, 실패 → 에러 표시
- [ ] 401 처리: 보호 fetch가 401이면 로그인으로 리다이렉트(`retry:false`)

### 2.5 이미지 업로드 플로우 (핵심 E2E)
- [ ] `app/api/analyses/route.ts`(BFF POST) — 쿠키 읽어 FastAPI `POST /api/v1/analyses`에 `Authorization` 전달, multipart 전달
- [ ] 업로드 UI: 드래그앤드롭 / 파일 선택 / 미리보기 / 다중 이미지(다중 뷰) — PLAN.md §7
- [ ] `useMutation`으로 업로드 → 성공 시 `analysis_id` 수신 → 결과 페이지로 이동(또는 대기 화면)
- [ ] 클라이언트 측 1차 검증(타입/크기)으로 UX 개선(서버 검증은 그대로 유지)

### 2.6 데이터 연동 기반
- [ ] `lib/api.ts` — BFF fetch 래퍼(상대경로 `/api/...`)
- [ ] Zod 스키마: `User`, `Analysis`(`status: queued|processing|done|failed`)
- [ ] Server Component에서 `cookies()` 읽어 초기 데이터(필요한 화면) 패칭

---

## 3. 산출물

1. 쿠키 인증(BFF) + middleware 가드가 동작하는 Next.js 앱
2. 회원가입/로그인/로그아웃 화면
3. 이미지 업로드 화면 + BFF 업로드 라우트
4. 공통 레이아웃/Providers/에러·로딩

---

## 4. 완료 기준 (Definition of Done)

- [ ] **E2E**: (브라우저) 로그인 → 보호 영역 진입 → 이미지 업로드 → `analysis_id` 수신까지 끊김 없이 동작
- [ ] 토큰이 httpOnly 쿠키에만 존재(JS `document.cookie`로 안 보임) 확인
- [ ] 미인증 상태로 보호 라우트 접근 시 로그인 리다이렉트
- [ ] 업로드된 이미지가 백엔드에서 `queued` 레코드로 생성됨(Phase 2 연동 확인)

### 테스트 전략
- Playwright 또는 Cypress로 E2E 테스트 (로그인 → 업로드 플로우)
- 쿠키 설정 확인 (httpOnly, secure, sameSite)
- middleware 가드 테스트 (미인증 리다이렉트)
- Zod 스키마 검증 테스트 (백엔드 응답 계약)

---

## 5. 리스크 / 주의

- **localStorage 금지**: 토큰은 절대 localStorage/JS 접근 영역에 두지 않는다(XSS·민감정보).
- **CORS 회피**: 브라우저는 같은 출처의 `/api/*`(BFF)만 호출, 실제 백엔드 URL은 서버에서 숨김.
- **Next 15+ 비동기 API**: `cookies()`/`params`는 `await`. 동기 취급 시 타입 에러.
- **계약 어긋남**: 백엔드 응답 키가 바뀌면 Zod `parse`에서 잡힌다 → 에러를 무시하지 말 것.
- **일정 리스크**: 가장 긴 단계(3–4주). 업로드 E2E를 먼저 관통시키고 폴리싱은 후순위.

---

## 6. 다음 단계 연결

- Phase 5가 업로드 직후의 **결과 대기/표시 화면**을 완성한다(여기선 `analysis_id` 수신까지).
- 인증/레이아웃/Query 패턴은 Phase 5~7 모든 화면의 공통 토대로 재사용.
