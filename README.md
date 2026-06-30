# SkyWeb - AI 피부분석 웹서버

> 얼굴 이미지를 **분석·측정**하여 피부 상태 점수와 맞춤형 솔루션을 제공하는 AI 기반 웹 서비스
>
> ⚠️ **표현 주의**: 본 서비스는 "진단(diagnosis)"이 아니라 **분석/측정/추정** 정보를 제공합니다.
> 질환 진단·치료를 암시하는 표현은 사용하지 않습니다(의료기기·의료행위 규제 회피 — PLAN.md §8).

## 프로젝트 개요

SkyWeb은 기존의 AI 피부분석 엔진(SkinLens)을 웹 서비스로 **래핑(wrapping)** 하여 사용자가 쉽게 접근할 수 있도록 하는 프로젝트입니다. 새로운 AI 모델을 학습/개발하는 것이 아니라, 이미 완성된 분석 파이프라인(`AnalysisService`)을 웹 API·비동기 워커로 감싸 호출·운영하는 것이 핵심입니다. 사용자가 얼굴 이미지를 업로드하면 백엔드 워커가 기존 엔진을 호출해 피부 상태를 분석하고, 결과를 시각화하여 제공합니다.

### 핵심 목표
- 기존 AI 엔진(SkinLens)을 웹 서비스로 래핑 (모델 신규 학습 없음)
- 1인(솔로) 개발자가 빠르게 만들고 오래 유지보수 가능한 구조
- 사용자 친화적인 UI/UX 제공
- 개인정보 보호 및 규제 준수 (PIPA)

## 기술 스택

### 프론트엔드
- **Next.js 15+** (App Router)
- **TypeScript**
- **TailwindCSS** + **shadcn/ui** + **Lucide React**
- **TanStack Query** (서버 상태/캐싱) · React Context (전역 UI 상태)
- **Zod** (런타임 검증 — Pydantic의 TS판)
- **PWA** (Serwist) — 앱 설치 경험·빠른 재방문, 오프라인 앱 셸 (민감정보·분석 결과는 비캐시)

### 백엔드
- **Python 3.10+** / **FastAPI**
- **SQLAlchemy 2.x** (ORM) + **Alembic** (마이그레이션)
- **Pydantic v2** (검증/DTO)
- **python-jose** (JWT) · **passlib[bcrypt]** (비밀번호 해시)
- **Celery** (비동기 작업 큐) — *단계적 도입: 극초기 `BackgroundTasks` → 워커 격리/재시도 필요 시 Celery 승격*

### 데이터 / 인프라
- **PostgreSQL** (주 데이터베이스 — Supabase/Neon 등 관리형)
- **Redis** (Celery 메시지 브로커 + 선택적 캐싱) — *서버 세션 저장소 아님(인증은 stateless JWT)*
- **S3 / Cloudflare R2** (원본 이미지·히트맵 저장, 서명된 URL로만 노출)
- **Docker Compose** (개발 환경) · **GitHub Actions** (CI/CD)

### AI/ML (기존 SkinLens 엔진 — 워커 이미지에만 격리 적재)
- **PyTorch** + 복원 모델(CodeFormer / RestoreFormer++)
- **OpenCV** (CV 분석 파이프라인)

> 참고: 무거운 ML 의존성은 **워커 컨테이너에만** 둔다. FastAPI API 컨테이너는 이를 포함하지 않아 가볍게 유지한다.
> 모델 학습 단계가 없으므로 증강(augmentation) 등 학습용 의존성은 본 프로젝트 범위가 아니다.

## 주요 기능 (MVP / v1)

### 1. 사용자 인증
- 이메일 회원가입/로그인 (소셜 로그인은 v2)
- JWT 기반 인증 (httpOnly 쿠키 — BFF 패턴)

### 2. 피부분석
- 얼굴 이미지 업로드 (드래그앤드롭, 카메라 촬영, 미리보기, 다중 뷰)
- 기존 AI 엔진을 통한 비동기 분석 (10개 직교 카테고리 + 18개 리포트 항목)
- 분석 결과 시각화 (히트맵, 점수 그래프)
- 분석 히스토리 및 기간별 변화 추이/비교

### 3. 맞춤형 솔루션 추천
- v1: 점수 구간 기반 규칙 추천 (정교한 추천은 v2)

### 4. 관리자 기능 (최소)
- SQLAdmin 기반 사용자/분석 결과 조회·통계

> **v2 이후**: 소셜 로그인, 비밀번호 재설정(메일 인프라 필요), refreshToken 무중단 갱신, 정교한 추천/개인화

## 시스템 아키텍처

```
┌─────────────┐
│   브라우저   │
└──────┬──────┘
       │ HTTPS
       ▼
┌──────────────────────┐
│  Next.js (Vercel)    │  프론트엔드 + BFF(httpOnly 쿠키)
└──────┬───────────────┘
       │ REST API (JSON)
       ▼
┌──────────────────────┐        ┌──────────────┐
│   FastAPI 백엔드      │───────▶│  S3 / R2     │  원본·히트맵
│   인증/업로드/조회    │ 이미지 └──────────────┘
└──────┬───────────────┘
       │ enqueue
       ▼
┌──────────────┐        ┌──────────────────────┐
│ Redis (큐)   │───────▶│  Celery Worker       │  분석 래퍼
└──────────────┘        │  └ SkinLens 엔진 호출 │  ← 신규 모델 없음
                        └──────┬───────────────┘
                               │ 결과 저장
                               ▼
                        ┌──────────────────────┐
                        │  PostgreSQL          │
                        │  users/analyses/...  │
                        └──────────────────────┘
```

상세 흐름·DB 스키마·API 설계는 [PLAN.md](./PLAN.md) §3~§6 참조.

## 개발 단계

| Phase | 내용 | 기간 |
|---|---|---|
| 0 | TypeScript/React/Next.js 학습 | 2-3주 |
| 1 | FastAPI 골격 + 인증 + Alembic | 2주 |
| 2 | 업로드 + S3/R2 + 작업 큐 | 1-2주 |
| 3 | SkinLens 엔진 래핑 (워커) | 2주 |
| 4 | Next.js 인증/레이아웃 + API 연동 | 3-4주 |
| 5 | 결과 시각화 (리포트/히트맵) | 2주 |
| 6 | 히스토리/추이/비교 | 1-2주 |
| 7 | 규칙 기반 추천 + 관리자 | 1-2주 |
| 8 | 테스트·보안 점검·배포 | 2주 |

총 **16-21주** 예상 (솔로 개발자 기준, 기존 프로젝트 유지보수 병행 전제)

## 문서

- **[PLAN.md](./docs/plan/PLAN.md)** - 상세 프로젝트 계획 (기술 스택, 아키텍처, API 설계, DB 스키마, 보안/규제)
- **[PHASE0_PLAN.md](./docs/plan/PHASE0_PLAN.md)** - TypeScript/React/Next.js 학습 (Python 개발자용)
- **[PHASE1_PLAN.md](./docs/plan/PHASE1_PLAN.md)** - FastAPI 골격 + 인증(JWT) + Alembic
- **[PHASE2_PLAN.md](./docs/plan/PHASE2_PLAN.md)** - 업로드 + S3/R2 + 작업 큐 enqueue
- **[PHASE3_PLAN.md](./docs/plan/PHASE3_PLAN.md)** - SkinLens 엔진 래핑 (워커)
- **[PHASE4_PLAN.md](./docs/plan/PHASE4_PLAN.md)** - Next.js 인증/레이아웃 + API 연동
- **[PHASE5_PLAN.md](./docs/plan/PHASE5_PLAN.md)** - 결과 시각화 (리포트/히트맵/그래프)
- **[PHASE6_PLAN.md](./docs/plan/PHASE6_PLAN.md)** - 히스토리/추이/비교
- **[PHASE7_PLAN.md](./docs/plan/PHASE7_PLAN.md)** - 규칙 기반 추천 + 최소 관리자
- **[PHASE8_PLAN.md](./docs/plan/PHASE8_PLAN.md)** - 테스트·보안 점검·배포

## 보안 및 규제 준수

### 보안
- JWT 기반 인증 (httpOnly 쿠키 — JS 접근 불가, XSS에 강함)
- HTTPS 강제, CORS 화이트리스트, Rate Limiting
- 입력 검증(Pydantic), 업로드 파일 타입/크기 검증
- 서명된 S3/R2 URL만 노출 (직접 공개 금지), IDOR/Path traversal 방지
- **시크릿 평문 금지**: 코드·compose·배포 산출물에 자격증명 인라인 금지, `.env`/시크릿 매니저 사용, `DEBUG=false`(운영)

### 개인정보 보호 (PIPA 우선)
- 얼굴 이미지는 생체/민감정보로 해석될 여지 → **별도 명시 동의** 수집·기록
- 암호화 저장, 보존기간(`retention_until`) + 자동 파기
- 계정 삭제 시 관련 이미지/분석 파기
- 이용약관 및 개인정보처리방침 고지

### 의료기기/의료행위 회피
- 전 화면 문구를 **"분석/측정/추정"**으로 통일, "진단" 미사용
- 질환 진단·치료를 암시하는 표현 배제

## 개발 환경

### 사전 요구사항
- Node.js v20+
- Python 3.10+
- Docker Desktop (WSL2 백엔드)
- PostgreSQL 15+
- Redis 7+

### Windows 개발 환경
- **WSL2 + Docker Desktop** 권장
- **VS Code Remote WSL** 사용
- 프로젝트 위치: `\\wsl$\Ubuntu\home\user\projects\...` (`C:\` 드라이브 비권장)

상세 설정은 [PLAN.md 부록 A](./PLAN.md) 참조

## PWA (Progressive Web App)

> **🔄 v2 개정(PWA 리뷰 반영)**: 도구 `next-pwa`(유지보수 중단) → **Serwist**(Next.js 공식 권장). 오프라인 범위를 앱 셸+안내로 한정하고 민감정보(분석 결과·얼굴 이미지)는 `NetworkOnly`로 비캐시. "Lighthouse PWA 점수"는 폐지(v12)되어 DevTools Application 검증으로 대체. 푸시는 v2로 분리. 상세: `SkyWeb_PWA_리뷰_및_보완.md`

### 필요성

SkyWeb에서 PWA의 실제 가치는 다음과 같습니다. (분석은 본질적으로 온라인+워커가 필요하므로 "오프라인 분석"은 제공하지 않습니다.)

- **앱 설치 경험**: 네이티브 앱과 유사한 설치/실행 경험 (앱스토어 승인 없이)
- **빠른 재방문**: 홈스크린 아이콘 + 앱 셸 캐싱으로 재진입·로딩 개선
- **오프라인 앱 셸**: 네트워크가 끊겨도 레이아웃과 안내 페이지는 표시 (분석 결과·이미지는 표시하지 않음)
- **(v2) 푸시 알림**: 분석 완료/추천 업데이트 재참여 — Web Push 인프라 필요, iOS 제약(아래) 때문에 v2로 분리

> ⚠️ **오프라인 범위 주의**: 얼굴 이미지·히트맵·분석 결과는 PIPA상 민감정보이며, 브라우저 캐시는 암호화되지 않고 기기에 영속한다. 따라서 이들을 **오프라인 저장·조회 대상으로 삼지 않는다**. 오프라인은 "앱 셸 + 안내 페이지"로만 한정한다.

### 단계별 구현·검증 (개정)

**Phase 1: 기본 PWA 설정**
- manifest 생성 (`id`/`scope`/`orientation` + 192·512 + **maskable** 아이콘 + screenshots, 브랜드 테마색)
- Serwist 서비스 워커 등록 (Next.js 공식 권장 도구)
- HTTPS 환경 확인 (localhost는 보안 출처로 취급)

**Phase 2: 오프라인(앱 셸 한정)**
- 정적 자산(앱 셸·아이콘·폰트) 캐싱
- `/~offline` 폴백 페이지 — "분석에는 인터넷 연결이 필요합니다" 안내
- **민감 경로 비캐시**: `/api/**`·서명 이미지(S3/R2)는 `NetworkOnly`(캐시 금지)
- 네트워크 상태 감지 및 UI 반영

**Phase 3: 설치 가능성**
- `beforeinstallprompt`(Android/Chromium) 처리 + 설치 버튼
- **iOS 수동 안내**: iOS Safari에는 `beforeinstallprompt`가 없으므로 "공유 → 홈 화면에 추가" 안내
- 설치 감지: `matchMedia('(display-mode: standalone)')` + `appinstalled` 이벤트

**Phase 4: 세션·보안 정리**
- 로그아웃·계정 삭제 시 Cache Storage·IndexedDB 정리(`Clear-Site-Data`) — 민감정보 캐시 잔존 방지

**Phase 5: 검증 (Lighthouse PWA 점수 폐지 반영)**
- DevTools **Application 패널**로 Manifest/Service Worker/설치 가능성 확인
- 브라우저 호환성(Chrome/Safari/Edge) + 오프라인/온라인 전환 시나리오
- **보안 회귀**: 로그아웃 후 캐시에 사용자 데이터·이미지 잔존 0건
- 성능은 일반 Lighthouse **Performance** 점수로 별도 관리 — *Lighthouse는 v12.0(2024-05)에서 PWA 카테고리를 제거함*

### 설치 가이드 (Serwist)

#### 1. 의존성 설치

```bash
npm install @serwist/next serwist   # next-pwa(유지보수 중단) 후속, Next.js 공식 권장
```

#### 2. next.config.ts 설정

```ts
import type { NextConfig } from "next";
import withSerwistInit from "@serwist/next";

const withSerwist = withSerwistInit({
  swSrc: "app/sw.ts",
  swDest: "public/sw.js",
  disable: process.env.NODE_ENV === "development", // 개발 중 SW 비활성(캐시 지옥 방지)
  reloadOnOnline: false,                            // 온라인 복귀 시 강제 새로고침 금지
});

const nextConfig: NextConfig = { reactStrictMode: true };
export default withSerwist(nextConfig);
```

> Turbopack을 기본 빌드로 쓰는 버전에서는 Serwist가 Webpack을 요구하므로 `package.json`에서 `"build": "next build --webpack"`로 지정한다.

#### 3. 서비스 워커 (`app/sw.ts`) — 캐싱 정책에 보안 반영

```ts
import { defaultCache } from "@serwist/next/worker";
import { NetworkOnly, Serwist } from "serwist";

const runtimeCaching = [
  // 민감 API(분석 결과·히스토리): 캐시 금지
  { matcher: ({ url, sameOrigin }) => sameOrigin && url.pathname.startsWith("/api/"),
    handler: new NetworkOnly() },
  // 서명된 사용자 이미지(원본·히트맵): 캐시 금지(만료·접근통제·생체정보)
  { matcher: ({ url }) => /\.(s3|r2)\./.test(url.hostname) || url.pathname.includes("/signed/"),
    handler: new NetworkOnly() },
  ...defaultCache, // 그 외 정적 자산만 캐시
];

const serwist = new Serwist({
  precacheEntries: self.__SW_MANIFEST,
  skipWaiting: false, clientsClaim: true, navigationPreload: true, runtimeCaching,
  fallbacks: { entries: [{ url: "/~offline", matcher: ({ request }) => request.mode === "navigate" }] },
});
serwist.addEventListeners();
```

#### 4. manifest (`app/manifest.ts` 또는 `public/manifest.json`)

```jsonc
{
  "id": "/",
  "name": "SkyWeb — AI 피부분석",
  "short_name": "SkyWeb",
  "description": "AI 기반 피부 상태 분석 서비스 (진단 아님)",
  "start_url": "/", "scope": "/", "display": "standalone", "orientation": "portrait",
  "background_color": "#FFFFFF", "theme_color": "#0E7C7B",
  "categories": ["health", "lifestyle"],
  "icons": [
    { "src": "/icons/icon-192.png", "sizes": "192x192", "type": "image/png" },
    { "src": "/icons/icon-512.png", "sizes": "512x512", "type": "image/png" },
    { "src": "/icons/maskable-192.png", "sizes": "192x192", "type": "image/png", "purpose": "maskable" },
    { "src": "/icons/maskable-512.png", "sizes": "512x512", "type": "image/png", "purpose": "maskable" }
  ]
}
```

> iOS는 manifest 아이콘을 무시하는 경우가 있어 `app/layout.tsx`에 `<link rel="apple-touch-icon" href="/icons/apple-touch-icon.png">`를 별도로 둔다.

#### 5. 메타데이터 / viewport (Next 15)

```tsx
// app/layout.tsx
import type { Metadata, Viewport } from "next";

export const metadata: Metadata = {
  applicationName: "SkyWeb",
  manifest: "/manifest.webmanifest",
  appleWebApp: { capable: true, statusBarStyle: "default", title: "SkyWeb" },
};

// themeColor는 metadata가 아니라 viewport로 (Next 15)
export const viewport: Viewport = { themeColor: "#0E7C7B" };
```

#### 6. 설치 UX (Android prompt + iOS 수동 안내)

```tsx
"use client";
import { useEffect, useState } from "react";

export default function InstallPrompt() {
  const [deferred, setDeferred] = useState<any>(null);
  const [isIOS, setIsIOS] = useState(false);
  const [installed, setInstalled] = useState(false);

  useEffect(() => {
    setInstalled(window.matchMedia("(display-mode: standalone)").matches);
    setIsIOS(/iphone|ipad|ipod/i.test(navigator.userAgent));
    const onPrompt = (e: Event) => { e.preventDefault(); setDeferred(e); };
    const onInstalled = () => setInstalled(true);
    window.addEventListener("beforeinstallprompt", onPrompt);
    window.addEventListener("appinstalled", onInstalled);
    return () => {
      window.removeEventListener("beforeinstallprompt", onPrompt);
      window.removeEventListener("appinstalled", onInstalled);
    };
  }, []);

  if (installed) return null;
  if (isIOS) return <p>설치하려면 공유 버튼 → &quot;홈 화면에 추가&quot;를 눌러주세요.</p>;
  if (!deferred) return null;
  return (
    <button onClick={async () => { deferred.prompt(); await deferred.userChoice; setDeferred(null); }}>
      앱 설치
    </button>
  );
}
```

#### 7. 검증

```bash
npm run dev          # 개발: SW 비활성(disable)
npm run build        # 프로덕션 빌드 (필요 시 next build --webpack)
npm start
```

- DevTools → **Application** 패널에서 Manifest·Service Worker·설치 가능성 확인 (Lighthouse PWA 점수는 폐지됨)
- 오프라인 차단 시 `/api`·이미지가 캐시에서 나오지 않는지 Cache Storage 검사
- 로그아웃 후 캐시·IndexedDB에 사용자 데이터 잔존 0건 확인

## 시작하기

### 1. 리포지토리 클론
```bash
git clone https://github.com/skygoldlee-cyber/SkyWeb.git
cd SkyWeb
```

### 2. 환경 변수 설정
```bash
cp .env.example .env   # 실제 값 채우기 (자격증명 평문 커밋 금지)
```

### 3. Docker 컨테이너 시작
```bash
docker-compose up -d
```

### 4. 데이터베이스 마이그레이션
```bash
alembic upgrade head
```

### 5. 개발 서버 실행
```bash
# 백엔드
uvicorn app.main:app --reload --host 0.0.0.0 --port 8000

# 프론트엔드
npm run dev
```

## 라이선스

MIT License

## 연락처

- GitHub: https://github.com/skygoldlee-cyber/SkyWeb
