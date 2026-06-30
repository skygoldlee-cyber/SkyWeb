# Phase 4 실행계획 — Next.js 인증/레이아웃 + API 연동

> **상위 문서**: PHASE0_PLAN.md Day 15-21, PLAN.md §3, §6
> **기간(현실안)**: 3–4주
> **완료 기준**: 로그인 ~ 업로드 플로우 **E2E** 동작
> **성격**: Phase 0에서 익힌 패턴을 **실제 백엔드(Phase 1~3)에 연결**. 프론트 본격 구현 시작.
>
> **🔄 v2 개정(PWA 리뷰 반영)**: PWA 도구를 `next-pwa`(유지보수 중단) → **Serwist**(Next.js 공식 권장)로 교체. 오프라인 캐싱이 민감정보(얼굴 이미지·분석 결과)와 충돌하지 않도록 `/api`·서명 이미지를 **NetworkOnly**(캐시 금지)로 고정. 설치 UX에 iOS 수동 안내를 추가. 상세는 `SkyWeb_PWA_리뷰_및_보완.md` 참조.

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
npm install @serwist/next serwist  # PWA (next-pwa 후속, Next.js 공식 권장)
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

### 2.2 PWA 기본 설정 (Serwist 기반)
- [ ] `next.config.ts`에 Serwist 설정(개발 중 비활성, 온라인 복귀 강제 새로고침 금지)
  ```ts
  import withSerwistInit from "@serwist/next";
  const withSerwist = withSerwistInit({
    swSrc: "app/sw.ts",
    swDest: "public/sw.js",
    disable: process.env.NODE_ENV === "development",
    reloadOnOnline: false,
  });
  export default withSerwist({ reactStrictMode: true });
  ```
- [ ] **빌드 스크립트 주의**: Turbopack 기본 빌드 버전에서는 Serwist가 Webpack을 요구하므로 `"build": "next build --webpack"`로 지정
- [ ] `app/sw.ts` 서비스워커 — **캐싱 정책에 보안 반영**(아래 표)
  ```ts
  import { defaultCache } from "@serwist/next/worker";
  import { NetworkOnly, Serwist } from "serwist";
  const runtimeCaching = [
    // 민감 API(분석 결과·히스토리·사용자 데이터): 캐시 금지
    { matcher: ({ url, sameOrigin }) => sameOrigin && url.pathname.startsWith("/api/"),
      handler: new NetworkOnly() },
    // 서명된 사용자 이미지(원본·히트맵): 캐시 금지(만료·접근통제·생체정보)
    { matcher: ({ url }) => /\.(s3|r2)\./.test(url.hostname) || url.pathname.includes("/signed/"),
      handler: new NetworkOnly() },
    ...defaultCache, // 그 외 정적 자산(앱 셸·아이콘·폰트)만 캐시
  ];
  const serwist = new Serwist({
    precacheEntries: self.__SW_MANIFEST,
    skipWaiting: false, clientsClaim: true, navigationPreload: true, runtimeCaching,
    fallbacks: { entries: [{ url: "/~offline", matcher: ({ request }) => request.mode === "navigate" }] },
  });
  serwist.addEventListeners();
  ```
- [ ] `app/~offline/page.tsx` — 오프라인 안내 페이지("분석에는 인터넷 연결이 필요합니다"). **분석 결과를 오프라인 저장·표시하지 않음**
- [ ] `app/manifest.ts`(또는 `public/manifest.json`) — `id`/`scope`/`orientation` + 192·512 + **maskable** 아이콘 + `screenshots`, 브랜드 테마색
- [ ] `app/layout.tsx` — `metadata.manifest`, `appleWebApp`, `<link rel="apple-touch-icon">`. **`themeColor`는 `metadata`가 아니라 `viewport` export로**(Next 15)
  ```tsx
  export const viewport = { themeColor: "#0E7C7B" };
  ```
- [ ] HTTPS 환경 확인 (PWA 필수 — localhost는 보안 출처로 취급됨)

**경로별 캐싱 정책 (보안 핵심)**

| 대상 | 전략 | 이유 |
|---|---|---|
| 앱 셸·정적 JS/CSS/아이콘/폰트 | precache + `defaultCache` | 로딩 속도·오프라인 셸 |
| `/api/**` (분석 결과·히스토리) | **NetworkOnly** | 민감정보 캐시 금지(PIPA) |
| 서명 S3/R2 이미지(원본·히트맵) | **NetworkOnly** | 만료 URL·접근통제·생체정보 |
| 네비게이션 실패 | `/~offline` 폴백 | 분석은 오프라인 불가, 안내만 |

### 2.3 쿠키 기반 인증 (BFF) — Day 15-18
- [ ] `app/api/auth/login/route.ts` — FastAPI `/api/v1/auth/login` 호출 → `access_token`을 **httpOnly 쿠키**로 `Set-Cookie`
  - `httpOnly: true`, `secure`(prod), `sameSite: "lax"`, `path:"/"`, `maxAge: 60*30`
- [ ] `app/api/auth/register/route.ts` — FastAPI `/api/v1/auth/register` 프록시
- [ ] `app/api/auth/logout/route.ts` — 쿠키 삭제
- [ ] `AuthContext` — 현재 사용자 상태만 보관(토큰은 JS가 안 만짐)
- [ ] `middleware.ts` — 보호 라우트 가드(`matcher: ["/analyses/:path*", "/profile"]`)

### 2.4 레이아웃 / 공통
- [ ] `app/layout.tsx` — `<html lang="ko">`, 네비게이션, `Providers`(TanStack Query) 래핑
- [ ] 로그인 상태별 네비게이션(로그인/로그아웃 토글)
- [ ] `app/providers.tsx` — `QueryClientProvider`
- [ ] `loading.tsx` / `error.tsx` 공통 패턴

### 2.5 인증 화면
- [ ] `/auth/register`, `/auth/login` — react-hook-form + Zod(`LoginSchema`, `RegisterSchema`)
- [ ] 로그인 성공 → 보호 영역 리다이렉트, 실패 → 에러 표시
- [ ] 401 처리: 보호 fetch가 401이면 로그인으로 리다이렉트(`retry:false`)

### 2.6 이미지 업로드 플로우 (핵심 E2E)
- [ ] `app/api/analyses/route.ts`(BFF POST) — 쿠키 읽어 FastAPI `POST /api/v1/analyses`에 `Authorization` 전달, multipart 전달
- [ ] 업로드 UI: 드래그앤드롭 / 파일 선택 / 미리보기 / 다중 이미지(다중 뷰) — PLAN.md §7
- [ ] `useMutation`으로 업로드 → 성공 시 `analysis_id` 수신 → 결과 페이지로 이동(또는 대기 화면)
- [ ] 클라이언트 측 1차 검증(타입/크기)으로 UX 개선(서버 검증은 그대로 유지)

### 2.7 데이터 연동 기반
- [ ] `lib/api.ts` — BFF fetch 래퍼(상대경로 `/api/...`)
- [ ] Zod 스키마: `User`, `Analysis`(`status: queued|processing|done|failed`)
- [ ] Server Component에서 `cookies()` 읽어 초기 데이터(필요한 화면) 패칭

### 2.8 PWA 설치 UX (Android prompt + iOS 수동 안내)
- [ ] `InstallPrompt` 컴포넌트 — `beforeinstallprompt` 처리(Android/데스크톱 Chromium)
- [ ] **iOS 분기**: iOS Safari에는 `beforeinstallprompt`가 없음 → "공유 → 홈 화면에 추가" **수동 안내** 노출
- [ ] 설치 여부 감지는 `window.matchMedia('(display-mode: standalone)')` + `appinstalled` 이벤트 사용(`localStorage`는 "배너 다시 보지 않기" 기억 용도로만)
- [ ] 설치 버튼/안내 UI (네비게이션 또는 설정 화면)
- [ ] 이미 설치(standalone) 상태면 비노출

---

## 3. 산출물

1. 쿠키 인증(BFF) + middleware 가드가 동작하는 Next.js 앱
2. 회원가입/로그인/로그아웃 화면
3. 이미지 업로드 화면 + BFF 업로드 라우트
4. 공통 레이아웃/Providers/에러·로딩
5. PWA 기본 설정 (Serwist SW, manifest, viewport, 오프라인 폴백 — 민감정보 NetworkOnly)
6. PWA 설치 UX 컴포넌트 (Android prompt + iOS 수동 안내)

---

## 4. 완료 기준 (Definition of Done)

- [ ] **E2E**: (브라우저) 로그인 → 보호 영역 진입 → 이미지 업로드 → `analysis_id` 수신까지 끊김 없이 동작
- [ ] 토큰이 httpOnly 쿠키에만 존재(JS `document.cookie`로 안 보임) 확인
- [ ] 미인증 상태로 보호 라우트 접근 시 로그인 리다이렉트
- [ ] 업로드된 이미지가 백엔드에서 `queued` 레코드로 생성됨(Phase 2 연동 확인)
- [ ] PWA manifest가 정상 로드되고 DevTools Application 패널에서 설치 가능으로 인식
- [ ] 설치 UX: Android `beforeinstallprompt` 동작 + iOS 수동 안내 노출
- [ ] **오프라인 시 `/api`·서명 이미지가 캐시에서 나오지 않음**(Cache Storage 검사) — 앱 셸+`/~offline`만 동작
- [ ] HTTPS 환경에서 PWA 설치 가능 확인

### 테스트 전략
- Playwright 또는 Cypress로 E2E 테스트 (로그인 → 업로드 플로우)
- 쿠키 설정 확인 (httpOnly, secure, sameSite)
- middleware 가드 테스트 (미인증 리다이렉트)
- Zod 스키마 검증 테스트 (백엔드 응답 계약)
- PWA manifest 로드 / DevTools 설치 가능성 확인
- 설치 UX 동작 테스트 (Android prompt, iOS 안내 분기)
- 오프라인 캐시 보안 테스트: 로그아웃/오프라인 시 `/api`·이미지 캐시 미적재 확인

---

## 5. 리스크 / 주의

- **localStorage 금지**: 토큰은 절대 localStorage/JS 접근 영역에 두지 않는다(XSS·민감정보).
- **CORS 회피**: 브라우저는 같은 출처의 `/api/*`(BFF)만 호출, 실제 백엔드 URL은 서버에서 숨김.
- **Next 15+ 비동기 API**: `cookies()`/`params`는 `await`. 동기 취급 시 타입 에러.
- **계약 어긋남**: 백엔드 응답 키가 바뀌면 Zod `parse`에서 잡힌다 → 에러를 무시하지 말 것.
- **일정 리스크**: 가장 긴 단계(3–4주). 업로드 E2E를 먼저 관통시키고 폴리싱은 후순위.
- **PWA HTTPS 필수**: PWA는 HTTPS(또는 localhost)에서만 동작하므로 개발 시에도 보안 출처 필요
- **오프라인 캐싱 ↔ 민감정보(중요)**: 얼굴 이미지·히트맵·분석 결과를 Cache Storage에 남기면 PIPA 원칙 위반. `/api`·서명 이미지는 반드시 NetworkOnly, 로그아웃·계정 삭제 시 캐시 purge(§Phase 8 연동).
- **SW 업데이트 stale**: `skipWaiting` 자동 적용 시 Next 해시 청크 불일치로 흰 화면 위험 → `skipWaiting:false` + 사용자 안내 reload, `reloadOnOnline:false`.
- **Turbopack/Webpack**: Serwist는 Webpack 빌드 필요 → `next build --webpack`.

---

## 6. 다음 단계 연결

- Phase 5가 업로드 직후의 **결과 대기/표시 화면**을 완성한다(여기선 `analysis_id` 수신까지).
- 인증/레이아웃/Query 패턴은 Phase 5~7 모든 화면의 공통 토대로 재사용.
