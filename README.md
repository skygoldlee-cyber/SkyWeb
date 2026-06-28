# SkyWeb - AI 피부분석 웹서버

> 얼굴 이미지를 분석하여 피부 상태를 진단하고 맞춤형 솔루션을 추천하는 AI 기반 웹 서비스

## 프로젝트 개요

SkyWeb은 기존의 AI 피부분석 엔진(SkinLens)을 웹 서비스로 래핑하여 사용자가 쉽게 접근할 수 있도록 하는 프로젝트입니다. 사용자가 얼굴 이미지를 업로드하면 백엔드에서 AI 엔진을 통해 피부 상태를 분석하고, 결과를 시각화하여 제공합니다.

### 핵심 목표
- 기존 AI 엔진을 웹 서비스로 래핑
- 사용자 친화적인 UI/UX 제공
- 개인정보 보호 및 규제 준수 (PIPA)
- 확장 가능한 아키텍처 설계

## 기술 스택

### 프론트엔드
- **Next.js 15+** (App Router)
- **TypeScript**
- **TailwindCSS**
- **shadcn/ui**
- **TanStack Query** (데이터 캐싱)
- **Zod** (런타임 검증)

### 백엔드
- **FastAPI** (Python)
- **SQLAlchemy** (ORM)
- **Alembic** (마이그레이션)
- **Celery** (비동기 작업 큐)
- **Redis** (캐싱 + 메시지 브로커)

### 데이터베이스
- **PostgreSQL** (주 데이터베이스)
- **Redis** (캐싱 + 세션)

### 인프라
- **Docker Compose** (개발 환경)
- **Docker** (컨테이너화)
- **WSL2** (Windows 개발 환경)

### AI/ML
- **PyTorch** (기존 엔진)
- **TensorFlow** (기존 엔진)
- **OpenCV** (이미지 처리)
- **albumentations** (이미지 증강)

## 주요 기능

### 1. 사용자 인증
- 이메일 회원가입/로그인
- JWT 기반 인증 (httpOnly 쿠키)
- 비밀번호 재설정

### 2. 피부분석
- 얼굴 이미지 업로드
- AI 엔진을 통한 자동 분석
- 분석 결과 시각화 (히트맵, 그래프)
- 분석 히스토리 관리

### 3. 맞춤형 솔루션 추천
- 분석 결과 기반 추천
- 규칙 기반 추천 시스템

### 4. 관리자 기능
- 사용자 관리
- 분석 결과 관리
- 시스템 모니터링

## 시스템 아키텍처

```
┌─────────────┐
│   브라우저   │
└──────┬──────┘
       │ HTTP/HTTPS
       ▼
┌─────────────┐
│  Next.js    │ (프론트엔드 + BFF)
│  (App Router)│
└──────┬──────┘
       │ REST API
       ▼
┌─────────────┐
│   FastAPI   │ (백엔드)
└──────┬──────┘
       │ Celery
       ▼
┌─────────────┐
│  Celery     │ (AI 분석 워커)
│  Worker     │
└──────┬──────┘
       │
       ▼
┌─────────────┐
│ SkinLens    │ (AI 엔진)
│  Engine     │
└─────────────┘
```

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

총 **15-20주** 예상 (솔로 개발자 기준)

## 문서

- **[PLAN.md](./PLAN.md)** - 상세 프로젝트 계획 (기술 스택, 아키텍처, API 설계, 데이터베이스 스키마)
- **[PHASE0_PLAN.md](./PHASE0_PLAN.md)** - Phase 0 실행계획: TypeScript/React/Next.js 학습 가이드 (Python 개발자용)
- **[PHASE1_PLAN.md](./PHASE1_PLAN.md)** - Phase 1 실행계획: FastAPI 골격 + 인증(JWT) + Alembic

## 보안 및 규제 준수

### 보안
- JWT 기반 인증 (httpOnly 쿠키)
- HTTPS 통신
- 환경 변수 관리
- SQL Injection 방지
- XSS 방지

### 개인정보 보호
- PIPA (개인정보보호법) 준수
- 이용약관 및 개인정보처리방침
- 동의/파기 흐름
- 데이터 암호화

### 의료기기 회피
- 진단 목적이 아닌 참고 정보 제공
- 의료기기법 적용 범위 외

## 개발 환경

### 사전 요구사항
- Node.js v20+
- Python 3.11+
- Docker Desktop (WSL2 백엔드)
- PostgreSQL 15+
- Redis 7+

### Windows 개발 환경
- **WSL2 + Docker Desktop** 권장
- **VS Code Remote WSL** 사용
- 프로젝트 위치: `\\wsl$\Ubuntu\home\user\projects\...`

상세 설정은 [PLAN.md 부록 A](./PLAN.md#부록-a-windows-개발-환경-wsl2) 참조

## 시작하기

### 1. 리포지토리 클론
```bash
git clone https://github.com/skygoldlee-cyber/SkyWeb.git
cd SkyWeb
```

### 2. 개발 환경 설정
```bash
# Python 가상환경
python -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate
pip install -r requirements.txt

# Node.js 의존성
npm install
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
