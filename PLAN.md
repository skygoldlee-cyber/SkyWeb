# 피부분석 웹서버 개발 계획 (정리본)

> 이 문서는 **기존 SkinLens 분석 엔진을 웹 서비스로 제공**하기 위한 계획서입니다.
> 새로운 AI 모델을 학습/개발하는 프로젝트가 아니라, 이미 완성도 높은 분석 파이프라인(`AnalysisService`)을
> 웹 API·비동기 워커로 감싸서(wrapping) 사용자에게 노출하는 것이 핵심입니다.

---

## 1. 프로젝트 개요

CÔTELEAF 피부분석 서비스의 웹 버전. 사용자가 얼굴 이미지를 업로드하면 기존 SkinLens 엔진이
피부 상태를 **분석·측정**하고, 점수 리포트·히트맵·맞춤 추천을 제공한다.

> **표현 주의**: "진단(diagnosis)"이 아니라 **"분석/측정/추정"**으로 일관되게 표기한다.
> "진단"은 의료기기·의료행위 규제(식약처) 영역을 건드릴 수 있다. (→ §8 규제)

### 작성자 / 운영 형태
- **솔로 개발자(1인)** 프로젝트. Python 주력, JavaScript 입문 단계.
- 따라서 모든 기술 선택 기준은 **혼자서 빠르게 만들고 오래 유지보수 가능한가**이다.
  채용 인력 풀·팀 협업 같은 기준은 적용하지 않는다.

### MVP 범위 (v1)
- 이메일 회원가입/로그인 (소셜 로그인은 v2)
- 단일/다중 이미지 업로드 → 비동기 분석 → 결과 리포트 + 히트맵
- 분석 히스토리 조회 및 기간별 변화 추이
- 간단한 규칙 기반 제품 추천 (정교한 추천 알고리즘은 v2)
- 최소 관리자 조회 (SQLAdmin 등)

> **성공 지표(예시 — 실제 수치로 확정 필요)**: 업로드→결과 평균 처리시간 P95 < N초,
> 분석 실패율 < N%, 주 N건 분석.

---

## 2. 기술 스택 (결정 + 포기한 것)

비교 표 전체는 **부록 B**로 분리했다. 본문에는 결정과 근거만 둔다.

### 프론트엔드 — Next.js + **TypeScript**
- **스택**: Next.js (App Router) / **TypeScript** / TailwindCSS / shadcn/ui / Lucide React / `next/image`
- **상태관리**: 서버 상태는 TanStack Query, 전역 UI 상태는 React Context (필요 시 Zustand)
- **근거**: React 생태계·이미지 최적화 내장·향후 React Native 확장 여지.
- **TypeScript를 선택한 이유 (이전 안의 JS 권장을 뒤집음)**:
  Python을 주력으로 쓰고 이미 **type hints + Pydantic**에 익숙하므로 TS 타입 시스템은 오히려 친숙한 영역이다.
  shadcn/ui·Next.js 생태계가 TS를 기본 전제로 하므로 JS로 가면 예제·문서와 어긋나 학습 부담이 **더** 커진다.
  컴파일 타임 타입 에러는 입문자의 디버깅 부담을 줄여준다.
- **포기한 것**: Svelte/Solid의 더 작은 번들·빠른 렌더링. v1 규모에서 체감 이득이 작고 생태계·자료가 얇아 1인 유지보수에 불리.

### 백엔드 — FastAPI (Python)
- **스택**: Python 3.10+ / FastAPI / SQLAlchemy 2.x + Alembic / Pydantic v2 / python-jose(JWT) / passlib(bcrypt)
- **근거**: **기존 SkinLens 엔진이 Python이다.** 백엔드를 Python으로 두면 분석 엔진을
  별도 언어 브리지 없이 **직접 import해서 호출**할 수 있다. 이게 이 프로젝트에서 가장 중요한 결정.
- **FastAPI vs Django**: 비동기 API 서버·자동 문서화(OpenAPI)·AI 통합 친화성 때문에 FastAPI.
- **포기한 것**: Django의 내장 Admin/ORM/인증. 관리자 화면은 초기엔 **SQLAdmin** 같은 경량 도구로 대체한다.

### 비동기 처리 — Redis + Celery (단, 단계적 도입)
- 분석은 CV 파이프라인 + 복원 모델(CodeFormer/RestoreFormer++)까지 포함해 **수 초~수십 초** 걸릴 수 있으므로
  요청-응답 안에서 동기 처리하면 안 된다. → **작업 큐 기반 비동기**.
- **단계적 도입**: v1 극초기에는 FastAPI `BackgroundTasks`로 시작해도 되지만, 워커 격리·재시도·동시성 제어가
  필요해지는 시점에 **Celery + Redis**로 승격한다. (아래 아키텍처는 Celery 기준)
- **제외**: Celery Beat(스케줄 작업 없음), Kubernetes, 멀티리전, Canary/Blue-Green —
  전부 v1 과설계. → **부록 C(향후 스케일링)**로 이동.

---

## 3. 시스템 아키텍처

```
┌──────────────────────────────────────────────────────────┐
│  사용자 브라우저 (Chrome / Safari / Mobile)              │
└───────────────────────────┬──────────────────────────────┘
                            │ HTTPS
                            ▼
┌──────────────────────────────────────────────────────────┐
│  Next.js 프론트엔드 (Vercel)                              │
│  인증 / 업로드 UI / 결과 시각화 / 히스토리 대시보드       │
└───────────────────────────┬──────────────────────────────┘
                            │ REST API (JSON)
                            ▼
┌──────────────────────────────────────────────────────────┐
│  FastAPI 백엔드                                            │
│  - /api/auth/*  /api/analyses/*  /api/user/*               │
│  - 인증(JWT) / 업로드 처리 / 작업 enqueue / 결과 조회      │
│  - SQLAlchemy ORM (users, analyses, recommendations)       │
└──────┬─────────────────────────────┬─────────────────────┘
       │ 이미지                       │ enqueue
       ▼                              ▼
┌──────────────┐            ┌──────────────────────────────┐
│ S3 / R2      │            │ Redis (작업 큐)              │
│ 원본·히트맵  │            └───────────────┬──────────────┘
└──────────────┘                            │
                                            ▼
                       ┌────────────────────────────────────┐
                       │ Celery Worker (분석 래퍼)          │
                       │  1. S3에서 이미지 로드             │
                       │  2. SkinLens AnalysisService 호출  │ ← 신규 모델 없음
                       │  3. 점수/리포트/히트맵 수신        │
                       │  4. 히트맵 S3 업로드, 결과 DB 저장 │
                       │  (엔진 = 기존 SkinLens 패키지)     │
                       └───────────────┬────────────────────┘
                                       │ 결과 저장
                                       ▼
                       ┌────────────────────────────────────┐
                       │ PostgreSQL (Supabase/Neon)         │
                       │ users / analyses / recommendations │
                       └────────────────────────────────────┘
```

### 통신 흐름
1. 사용자 → Next.js → FastAPI 로 이미지 업로드
2. FastAPI → S3 저장, `analyses` 레코드를 `status=queued`로 생성, Redis에 작업 enqueue, **작업 ID 즉시 반환**
3. Celery Worker가 작업 수신 → 이미지 로드 → **SkinLens 엔진 호출** → 히트맵 S3 업로드 → 결과 DB 저장(`status=done`)
4. 프론트는 폴링 또는 SSE/WebSocket으로 `status`를 확인하고 완료 시 결과 표시

---

## 4. AI 분석 계층 — 기존 엔진 래핑 (핵심)

> 이전 안은 "PyTorch/TensorFlow로 CNN·ViT 모델을 학습/추론"으로 적혀 있었으나, 이는 잘못된 전제다.
> 분석 로직은 **이미 SkinLens에 존재**한다. 웹 프로젝트가 할 일은 모델 개발이 아니라 **호출·운영**이다.

### 4.1 워커가 하는 일
워커는 얇은 어댑터다. 다음 외에는 아무 분석 로직도 갖지 않는다.

```python
# worker/tasks.py (개념 코드)
from celery import shared_task
from skinlens.services import AnalysisService   # 기존 엔진의 정식 진입점
from app.storage import download_from_s3, upload_heatmaps_to_s3
from app.repositories import save_analysis_result, mark_failed

@shared_task(bind=True, max_retries=2)
def run_analysis(self, analysis_id: str, image_keys: list[str], options: dict):
    try:
        local_paths = [download_from_s3(k) for k in image_keys]

        # ── 신규 모델 없음. 기존 파이프라인을 그대로 호출 ──
        result = AnalysisService.analyze(
            image_paths=local_paths,
            options=options,            # 다중 뷰/복원 on-off 등
        )
        # result: 10개 직교 카테고리 점수 + 18개 리포트 항목 + 히트맵 + 엔진/스키마 버전

        heatmap_keys = upload_heatmaps_to_s3(analysis_id, result.heatmaps)
        save_analysis_result(analysis_id, result, heatmap_keys)
    except Exception as e:
        mark_failed(analysis_id, error=str(e))
        raise
```

### 4.2 엔진 연결(seam) 설계
- **의존성으로 설치**: SkinLens를 워커 이미지에 패키지로 설치한다.
  - 사내 패키지(예: 프라이빗 인덱스) 또는 editable 설치(`pip install -e ../skinlens`) 또는 Docker 멀티스테이지에서 `src/` vendoring.
- **안정적 인터페이스(DTO) 고정**: 웹 계층은 엔진 내부 구조에 의존하지 않고 `AnalysisService.analyze()`의
  **입력/출력 계약**에만 의존한다. 출력은 Pydantic DTO로 받아 엔진이 내부적으로 진화해도 웹이 깨지지 않게 한다.
- **버전 기록**: 분석마다 `engine_version`, `score_schema_version`을 함께 저장한다.
  (재현성, 그리고 MDC95 기반 변화 검출/리포트 비교 로직과 정합을 맞추기 위함.)
- **무거운 의존성 격리**: PyTorch·OpenCV·복원 모델 가중치는 **워커 이미지에만** 둔다.
  FastAPI API 컨테이너는 이들을 포함하지 않아 가볍게 유지한다.

### 4.3 호스팅 함의
- 워커는 PyTorch + 복원 모델을 메모리에 올리므로 **메모리·CPU(또는 GPU) 여유가 있는 호스트**가 필요하다.
  Railway/Render 보급형(저메모리) 티어로는 부족하기 쉽다. → 워커 호스트 사양은 실제 모델 메모리 사용량을 측정해 결정.
- 모델 가중치 파일은 이미지에 굽거나 S3에서 시작 시 다운로드하는 방식 중 택1. (콜드스타트 vs 이미지 크기 트레이드오프)

---

## 5. 데이터베이스 스키마

> 변경점: 점수를 **고정 컬럼으로 하드코딩하지 않는다.** 기존 엔진은 10개 직교 카테고리 +
> 18개 리포트 항목을 산출하므로, 핵심 점수 몇 개만 컬럼으로 두고 상세는 JSONB로 둔다.
> 점수 타입은 `INTEGER`가 아니라 **NUMERIC**(소수 정밀도 보존).

### users
```sql
id            UUID PRIMARY KEY
email         VARCHAR UNIQUE NOT NULL
password_hash VARCHAR NOT NULL
name          VARCHAR
created_at    TIMESTAMPTZ DEFAULT now()
updated_at    TIMESTAMPTZ DEFAULT now()
```

### analyses
```sql
id                   UUID PRIMARY KEY
user_id              UUID REFERENCES users(id)
status               VARCHAR NOT NULL          -- queued | processing | done | failed
image_keys           JSONB   NOT NULL          -- 원본 이미지 S3 키(다중 뷰 지원)
heatmap_keys         JSONB                      -- 히트맵 S3 키
skin_type            VARCHAR                    -- 건성/지성/복합성/중성
overall_score        NUMERIC(5,2)
category_scores      JSONB                      -- 10개 직교 카테고리 점수
report_items         JSONB                      -- 18개 리포트 항목
engine_version       VARCHAR NOT NULL           -- 재현성/비교용
score_schema_version VARCHAR NOT NULL
error                TEXT                        -- 실패 시 사유
created_at           TIMESTAMPTZ DEFAULT now()
completed_at         TIMESTAMPTZ

-- 인덱스: (user_id, created_at DESC), (status)
```

### recommendations
```sql
id           UUID PRIMARY KEY
analysis_id  UUID REFERENCES analyses(id)
product_name VARCHAR
category      VARCHAR
reason        TEXT
created_at   TIMESTAMPTZ DEFAULT now()
```

### 개인정보/동의 (PIPA 대응 — §8 참조)
```sql
-- consents (또는 users에 컬럼으로)
user_id            UUID
consent_type       VARCHAR     -- 얼굴 이미지(민감정보) 처리 동의 등
consented_at       TIMESTAMPTZ
retention_until    DATE        -- 보존 만료 → 자동 파기 대상
```

---

## 6. API 설계 (FastAPI)

> 이전 문서에서 중복 기재되어 있던 API 목록을 하나로 통합. RESTful 자원명으로 정리(`/analyze` → `/analyses`).

### 인증
```
POST   /api/auth/register      회원가입
POST   /api/auth/login         로그인 (JWT 발급)
POST   /api/auth/logout        로그아웃
GET    /api/auth/me            현재 사용자
```

### 분석
```
POST   /api/analyses           업로드 + 분석 작업 생성 → { analysis_id, status: queued } 즉시 반환
GET    /api/analyses/{id}      단건 상태/결과 조회 (status로 진행상황 표현)
GET    /api/analyses           히스토리 목록 (페이지네이션)
GET    /api/analyses/{id}/heatmap   히트맵 URL(서명된 URL)
```

### 사용자
```
GET    /api/user/profile       프로필 조회
PUT    /api/user/profile       프로필 수정
DELETE /api/user/account       계정 삭제 (관련 이미지/분석 파기 포함)
```

> **보완(시작 전 권장)**: 각 엔드포인트 요청/응답 예시, 표준 에러 코드, Rate Limiting 정책, API 버전(`/api/v1`) 명시.

---

## 7. 핵심 기능

1. **인증** — 이메일 가입/로그인, JWT, 프로필 관리 (소셜 로그인 v2)
2. **이미지 업로드** — 드래그앤드롭, 카메라 촬영, 미리보기, 다중 이미지(다중 뷰)
3. **분석** — 기존 엔진의 10 카테고리/18 항목 점수 + 히트맵 (모델 신규 개발 없음)
4. **추천** — v1은 점수 구간 기반 규칙 추천, v2에서 고도화
5. **히스토리** — 과거 결과 조회, 기간별 변화 추이 그래프, 비교
6. **관리자(최소)** — SQLAdmin 기반 조회/통계

---

## 8. 보안 · 개인정보 · 규제 (시작 전 정리 필요 — 우선순위 상)

> 이전 문서에서 "낮은 우선순위"로 미뤄졌지만, 얼굴 이미지 서비스에서는 **출시 전 결정 사항**이다.

- **PIPA(개인정보보호법) 우선**: 한국 서비스이므로 GDPR보다 PIPA가 1순위. **얼굴 이미지는 생체정보/민감정보**로
  해석될 여지가 크다 → 별도 명시 동의, 암호화 저장, 보존기간·자동 파기, 처리방침 고지가 법적 요구사항이 될 수 있다.
- **의료기기/의료행위 회피**: 문구를 "분석/측정/추정"으로 통일하고 "진단"은 쓰지 않는다.
  질환 진단·치료를 암시하는 표현 배제.
- **시크릿 관리(중요)**: 어떤 자격증명도 코드·compose·배포 산출물에 평문으로 넣지 않는다.
  - compose는 `${VAR}` 참조만 사용, 실제 값은 `.env`(.gitignore) 또는 시크릿 매니저.
  - `DEBUG`는 기본 false, 운영에서 절대 true 금지.
- **일반 보안**: HTTPS 강제, CORS 화이트리스트, Rate Limiting, 입력 검증(Pydantic),
  업로드 파일 타입/크기 검증, 서명된 S3 URL(직접 공개 금지), 인증 없는 작업 엔드포인트 금지(IDOR/Path traversal 주의).

---

## 9. 인프라 (v1)

> K8s/멀티리전/Canary는 부록 C로 분리. v1은 단순하게.

- **프론트**: Vercel (Next.js)
- **API**: 단일 컨테이너 (Render/Railway 또는 작은 VM + Docker Compose)
- **워커**: **메모리 여유 있는 별도 호스트** (PyTorch+복원 모델 적재) — 사양은 측정 후 결정
- **DB**: 관리형 PostgreSQL (Supabase/Neon)
- **Redis**: 관리형 또는 컨테이너
- **스토리지**: S3 또는 Cloudflare R2 (+ CDN)
- **CI/CD**: GitHub Actions
- **비용**: 워커 호스트가 비용의 핵심 변수다. **시작 전 월 실비를 실수치로 추정**한다(보급형 티어 메모리 한계 주의).

### docker-compose (개발용, 시크릿은 모두 env 참조)
```yaml
services:
  api:
    build: ./backend
    command: uvicorn app.main:app --host 0.0.0.0 --port 8000 --reload
    ports: ["8000:8000"]
    env_file: [.env]            # 평문 자격증명 인라인 금지
    depends_on: [postgres, redis]
    volumes: ["./backend:/app"]
    networks: [app-net]

  worker:
    build: ./worker             # PyTorch/OpenCV/복원모델 + SkinLens 엔진 포함
    command: celery -A app.tasks worker --loglevel=info
    env_file: [.env]
    depends_on: [postgres, redis]
    volumes: ["./worker:/app", "./models:/models"]
    networks: [app-net]

  postgres:
    image: postgres:15-alpine
    environment:
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
      POSTGRES_DB: ${POSTGRES_DB}
    volumes: ["postgres-data:/var/lib/postgresql/data"]
    networks: [app-net]

  redis:
    image: redis:7-alpine
    volumes: ["redis-data:/data"]
    networks: [app-net]

  # 모니터링은 선택. 필요 시 profiles로 켠다.
  flower:
    build: ./worker
    command: celery -A app.tasks flower --port=5555
    ports: ["5555:5555"]
    env_file: [.env]
    depends_on: [redis]
    networks: [app-net]
    profiles: ["monitoring"]

networks: { app-net: { driver: bridge } }
volumes: { postgres-data: , redis-data: }
```

---

## 10. 개발 단계 (솔로 · 일정 현실화)

> 이전 일정은 매우 빠듯했다. 프론트 학습·구현 구간을 늘리고, AI 단계를 "엔진 래핑"으로 재정의했다.
> SkyPredictor·SkinLens 본체 유지보수와 병행한다는 전제이므로 버퍼를 둔다.

| Phase | 내용 | 기간(현실안) | 완료 기준 |
|---|---|---|---|
| 0 | TypeScript/React/Next 학습 | 2–3주 | 간단한 인증+목록 SPA 직접 구현 |
| 1 | FastAPI 골격 + 인증(JWT) + Alembic | 2주 | `/auth/*` 통과, 마이그레이션 동작 |
| 2 | 업로드 + S3/R2 + 작업 큐 enqueue | 1–2주 | 업로드 후 `status=queued` 레코드 생성 |
| 3 | **SkinLens 엔진 래핑(워커)** | 2주 | 워커가 엔진 호출→점수/히트맵 DB 저장, `engine_version` 기록 |
| 4 | Next.js 인증/레이아웃 + API 연동 | 3–4주 | 로그인~업로드 플로우 E2E |
| 5 | 결과 시각화(리포트/히트맵/그래프) | 2주 | 단건 결과 화면 완성 |
| 6 | 히스토리/추이/비교 | 1–2주 | 기간별 추이 그래프 |
| 7 | 규칙 기반 추천 + 최소 관리자 | 1–2주 | 점수 구간 추천 노출 |
| 8 | 테스트·보안 점검·배포 | 2주 | 운영 배포, 비밀키 점검, PIPA 동의 흐름 |

---

## 11. 남은 보완 항목 (체크리스트)

**시작 전(상)**: 성공 지표 수치 확정 · 엔진 DTO 계약 확정 · API 요청/응답 예시 · PIPA 동의/파기 흐름 · 워커 호스트 사양·비용 실측
**진행 중(중)**: 테스트 전략(엔진 호출 모킹 포함) · 에러/로깅 표준 · Rate Limiting · 성능 모니터링
**프로덕션 전(하)**: 백업/복구(RTO/RPO) · 이용약관/처리방침 · UX 와이어프레임 · 스케일링(부록 C)

---

# 부록 A. Windows 개발 환경 (WSL2)

> 본문에서 분리. 1인 개발 시 참고용. 핵심만 요약.

**권장 구성**: WSL2(Ubuntu) + Docker Desktop(WSL2 엔진) + VS Code Remote WSL.
프로젝트는 **WSL2 파일시스템**(`\\wsl$\Ubuntu\home\user\projects\...`)에 두고 `C:\`에 두지 않는다(바인드 마운트 성능 저하 회피).

**자주 겪는 이슈**
- **라인엔딩(CRLF/LF)**: `git config --global core.autocrlf input` + `.gitattributes`에 `* text=auto eol=lf`.
- **포트 충돌**: 호스트 포트만 바꿔 매핑(`"8001:8000"` 등).
- **리소스**: Docker Desktop > Resources에서 메모리 4GB+ / CPU 4+.
- **경로**: Python에서 `pathlib.Path(__file__).resolve()`로 크로스플랫폼 경로 처리.
- **PyTorch**: GPU 있으면 WSL2+CUDA(NVIDIA WSL 드라이버) 또는 Windows+CUDA, 없으면 CPU 휠.

**.dockerignore**: `__pycache__ *.pyc .venv .git node_modules *.log`

**비권장**: PowerShell/CMD에서 직접 Docker·Python 개발, `C:\` 드라이브 프로젝트 저장.

---

# 부록 B. 프레임워크 비교 (요약)

선택 결과만 알면 본문으로 충분하다. 아래는 후보 검토 기록.

| 프론트 후보 | 한줄 평 | v1 채택 여부 |
|---|---|---|
| **Next.js** | React 생태계 + 이미지 최적화 + SSR | **채택** |
| React(CRA/Vite) | 메타프레임워크 이점 없음 | 미채택 |
| Svelte/Solid | 빠르고 가볍지만 생태계·자료 얇음(1인 유지보수 불리) | 미채택 |
| Vue/Nuxt | 무난하나 RN 확장 고려 시 React 우위 | 미채택 |
| Angular | SPA v1엔 과설계 | 미채택 |

| 백엔드 후보 | 한줄 평 | v1 채택 여부 |
|---|---|---|
| **FastAPI** | 비동기·자동문서화·**엔진과 동일 언어(Python)** | **채택** |
| Django | Admin/ORM 내장 강점이나 무겁고 비동기 약함 | 미채택(Admin은 SQLAdmin로 대체) |
| Node.js | 프론트와 언어 일치하나 Python 엔진과 단절·CPU 작업 불리 | 미채택 |
| Go/Rust | 고성능이나 ML/엔진과 언어 불일치·개발속도 손해 | 미채택 |

---

# 부록 C. 향후 스케일링 (v1 제외)

필요해질 때 도입. v1에서는 하지 않는다.
- 컨테이너 오케스트레이션(Kubernetes), 워커 수평 확장(`replicas`)
- 배포 전략(Blue-Green / Canary), 멀티리전
- Celery Beat(주기 작업이 생길 때), 정교한 추천/모델 A-B 테스트
- CDN 고도화, 읽기 복제본, 캐시 계층 분리
