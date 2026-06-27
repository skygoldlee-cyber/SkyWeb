# 피부분석 웹서버 개발 계획

## 프로젝트 개요
AI 기반 피부분석 웹서버로 사용자의 피부 이미지를 분석하여 피부 상태를 진단하고 맞춤형 솔루션을 제공하는 웹 애플리케이션

## 기술 스택

### 프론트엔드 옵션 분석

#### React.js (현재 선택)
- **장점**:
  - 거대한 생태계와 커뮤니티
  - shadcn/ui 등 풍부한 컴포넌트 라이브러리
  - React Native로 모바일 확장 용이
  - 방대한 학습 자료 및 튜토리얼
  - 대규모 프로젝트 검증됨
- **단점**:
  - 번들 크기 상대적으로 큼
  - Virtual DOM 오버헤드
  - 복잡한 상태관리 필요
  - 학습 곡선 존재

#### Svelte (대안 고려)
- **장점**:
  - 컴파일 타임 최적화로 작은 번들 크기
  - Virtual DOM 없이 직접 DOM 조작 (빠른 렌더링)
  - 직관적인 문법, 적은 보일러플레이트
  - 내장 상태관리 (stores, reactive statements)
  - 이미지 처리 같은 고성능 작업에 유리
  - SvelteKit으로 풀스택 개발 가능
- **단점**:
  - 생태계가 React보다 작음
  - 컴포넌트 라이브러리 제한적 (shadcn-svelte 존재하지만 제한적)
  - 모바일 확장 (Svelte Native) 미성숙
  - 채용 인력 풀 작음
  - 장기적 유지보수 관련 우려

#### Vue.js (대안 고려)
- **장점**:
  - 학습 곡선 낮음, 직관적인 문법
  - 템플릿 기반으로 HTML 친화적
  - Progressive Framework (점진적 적용 가능)
  - Nuxt.js로 풀스택 개발 가능
  - 성능 좋음, 번들 크기 적절
  - 한국 커뮤니티 활발
- **단점**:
  - React보다 생태계 작음
  - TypeScript 지원이 Vue 3에서 개선되었지만 아직 미흡
  - 대규모 프로젝트에서 구조 관리 복잡
  - 모바일 확장 (NativeScript-Vue) 미성숙

#### Angular (대안 고려)
- **장점**:
  - 완전한 프레임워크 (모든 기능 내장)
  - TypeScript 기본 지원
  - 대규모 엔터프라이즈 프로젝트에 적합
  - 강력한 의존성 주입 시스템
  - 구조화된 개발 패턴
  - Ionic으로 모바일 확장 용이
- **단점**:
  - 학습 곡선 가파름
  - 번들 크기 큼
  - 과도한 보일러플레이트
  - 작은 프로젝트에 과도한 구조
  - 커뮤니티 성장 둔화

#### Solid.js (대안 고려)
- **장점**:
  - React와 유사한 JSX 문법
  - Fine-grained reactivity (매우 빠른 렌더링)
  - 작은 번들 크기
  - Virtual DOM 없음
  - 최신 Reactivity 기술
- **단점**:
  - 매우 작은 생태계
  - 컴포넌트 라이브러리 거의 없음
  - 학습 자료 부족
  - 채용 인력 풀 매우 작음
  - 장기적 안정성 미검증

#### Next.js (React 기반 메타프레임워크)
- **장점**:
  - React 생태계 그대로 활용
  - SSR/SSG 지원 (SEO 최적화)
  - API Routes로 백엔드 통합
  - 이미지 최적화 내장
  - Vercel 배포 최적화
  - 풍부한 미들웨어 및 기능
- **단점**:
  - React 단점 그대로 상속
  - 학습 곡선 추가 (Next.js 특유)
  - 구성 복잡성 증가
  - 서버 사이드 렌더링 오버헤드

#### 피부분석 프로젝트에 적합한 선택
- **Svelte 추천 이유**:
  - 이미지 처리/분석 결과 시각화에 높은 성능 필요
  - SPA 특성상 빠른 초기 로딩 중요
  - 복잡한 상태관리 필요성 낮음
  - SvelteKit으로 백엔드 통합 간소화 가능
- **React 유지 이유**:
  - 향후 모바일 앱 확장 계획 (React Native)
  - 팀의 React 경험 활용
  - 빠른 개발 속도 (컴포넌트 라이브러리)
- **Next.js 추천 이유**:
  - React 생태계 + 이미지 최적화 내장
  - API Routes로 백엔드 간소화 가능
  - SEO 고려시 유리
  - Vercel 배포 최적화
- **Vue.js 추천 이유**:
  - 학습 용이, 빠른 개발 가능
  - 한국 커뮤니티 활발
  - Nuxt.js로 풀스택 개발 가능
- **Solid.js 추천 이유**:
  - 최고 성능 (이미지 처리에 적합)
  - React 유사 문법으로 학습 용이
  - 매우 작은 번들 크기
- **Angular 비추천 이유**:
  - 과도한 구조, 학습 곡선 가파름
  - SPA 프로젝트에 과엔지니어링

### 프론트엔드 (최종 결정: Next.js - JavaScript 초고자용)
- **프레임워크**: Next.js (App Router)
- **언어**: JavaScript (TypeScript 선택적, 추천하지 않음)
- **스타일링**: TailwindCSS
- **컴포넌트**: shadcn/ui
- **아이콘**: Lucide React
- **상태관리**: React Context (Zustand 학습 부담 회피)
- **이미지**: next/image
- **API**: Django REST API와 통합

### 백엔드 (웹서버) 옵션 분석

#### Node.js (현재 선택)
- **장점**:
  - 프론트엔드와 동일 언어 (JavaScript/TypeScript)
  - Next.js API Routes로 통합 간소화
  - 비동기 I/O에 최적화 (높은 동시성)
  - npm 생태계 풍부
  - Vercel 배포 최적화
  - JSON 처리 네이티브
- **단점**:
  - CPU 집약적 작업에 부적합
  - 이미지 처리 시 제한적
  - Python 분석 서비스와 별도 통신 필요
  - 타입 시스템 (TypeScript) 학습 필요

#### Python (대안 고려)
- **장점**:
  - 분석 서비스와 동일 언어 (코드 공유 가능)
  - AI/ML 라이브러리 풍부 (직접 분석 가능)
  - 이미지 처리 강점 (OpenCV, PIL)
  - 데이터 처리 용이 (Pandas, NumPy)
  - 직관적인 문법, 빠른 개발
  - Django/FastAPI로 풀스택 가능
- **단점**:
  - GIL로 인해 동시성 제한
  - 비동기 처리 복잡 (asyncio)
  - Node.js보다 느린 HTTP 처리
  - 프론트엔드와 언어 불일치
  - 메모리 사용량 상대적으로 큼
  - Vercel 배포 제한적

#### Go (대안 고려)
- **장점**:
  - 고성능, 낮은 메모리 사용
  - 강력한 동시성 (Goroutines)
  - 컴파일 언어로 빠른 실행
  - 간단한 배포 (단일 바이너리)
  - 표준 라이브러리 풍부
- **단점**:
  - 생태계가 Node.js/Python보다 작음
  - 프론트엔드/ML과 언어 불일치
  - 학습 곡선 존재
  - 웹 프레임워크 제한적

#### Rust (대안 고려)
- **장점**:
  - 최고 성능, 메모리 안전
  - zero-cost abstractions
  - 강력한 타입 시스템
  - Actix-web으로 고성능 웹서버
- **단점**:
  - 학습 곡선 매우 가파름
  - 생태계 작음
  - 컴파일 시간 긺
  - 개발 속도 느림

#### 백엔드 선택 추천 (JavaScript 초고자 고려)
- **Python 추천 이유**:
  - JavaScript 학습 부담 최소화
  - Python 경험 활용
  - Django/FastAPI로 빠른 백엔드 개발
  - 분석 서비스와 통합 간소화
  - 풍부한 Python 생태계 활용
- **Node.js 비추천 이유**:
  - JavaScript/TypeScript 학습 곡선
  - 비동기 프로그래밍 개념 학습 필요
  - npm 생태계 익숙해지는 시간 소요
  - Python 경험 활용 불가

### 백엔드 (웹서버) - Python 프레임워크 비교

#### Django (현재 선택)
- **장점**:
  - 풀스택 프레임워크 (ORM, 인증, 관리자 패널 내장)
  - Django REST Framework으로 REST API 쉽게 구현
  - 보안 기능 내장 (CSRF, XSS 방지)
  - 방대한 생태계와 패키지
  - 관리자 패널로 데이터 관리 용이
  - 배포 경험 풍부
- **단점**:
  - 무거운 구조, 불필요한 기능 포함
  - 학습 곡선 존재
  - 동기 처리 기본 (비동기 지원 제한적)
  - 모놀리식 구조로 경향
  - 마이크로서비스에 부적합

#### FastAPI (대안 고려)
- **장점**:
  - 현대적, 빠른 성능 (Starlette 기반)
  - 비동기 처리 기본 지원 (async/await)
  - 자동 API 문서화 (Swagger UI, ReDoc)
  - 타입 힌트 기반 (Pydantic)
  - 적은 코드로 API 구현
  - 마이크로서비스에 적합
  - AI/ML 통합에 유리
- **단점**:
  - Django보다 생태계 작음
  - ORM 내장 없음 (SQLAlchemy 필요)
  - 인증/관리자 패널 직접 구현 필요
  - 배포 경험 상대적으로 부족
  - 풀스택 기능 부족

#### Django vs FastAPI 선택 추천
- **Django 추천 이유**:
  - 빠른 개발 속도 (내장 기능 많음)
  - 관리자 패널로 데이터 관리 편리
  - 보안 기능 내장
  - 팀이 Django에 익숙할 때
  - 풀스택 웹 애플리케이션에 적합
- **FastAPI 추천 이유**:
  - 고성능 API 서버 필요시
  - 비동기 처리 중요할 때 (AI 분석)
  - 마이크로서비스 아키텍처시
  - 자동 API 문서화 필요시
  - AI/ML 서비스에 적합
  - 최신 Python 기술 선호시

#### 피부분석 프로젝트에 적합한 선택
- **FastAPI 추천 이유**:
  - AI 분석 비동기 처리에 적합
  - 이미지 처리 성능 중요
  - API 서버 역할에 최적화
  - Celery와 통합 용이
  - 자동 문서화로 프론트엔드 연동 편리
- **Django 추천 이유**:
  - 관리자 패널 필요시
  - 빠른 프로토타이핑 필요시
  - 팀이 Django에 익숙할 때

### 백엔드 (웹서버) - 최종 결정: FastAPI
- **런타임**: Python 3.10+
- **프레임워크**: FastAPI
- **이미지 처리**: Pillow, OpenCV
- **인증**: FastAPI Security, JWT
- **데이터베이스**: SQLAlchemy + Alembic
- **ORM**: SQLAlchemy
- **태스크 큐**: Celery + Redis
- **API 문서**: Swagger UI (자동 생성)

### Python 분석 서비스 (FastAPI에 통합)
- **런타임**: Python 3.10+
- **프레임워크**: FastAPI (백엔드와 통합)
- **AI/ML**: PyTorch, TensorFlow, OpenCV
- **이미지 처리**: PIL, OpenCV, albumentations
- **모델**: 피부분석 딥러닝 모델 (CNN, Vision Transformer)
- **통합**: FastAPI Background Tasks 또는 Celery

### 데이터베이스
- **주 DB**: PostgreSQL (사용자 정보, 분석 결과)
- **캐싱**: Redis (세션 관리)

### 인프라
- **프론트엔드 배포**: Vercel
- **백엔드 배포**: Docker Compose (개발), Kubernetes/EC2 (프로덕션)
- **데이터베이스**: PostgreSQL (Docker 컨테이너)
- **캐싱**: Redis (Docker 컨테이너)
- **파일 스토리지**: AWS S3, Cloudflare R2, 또는 로컬 볼륨
- **CI/CD**: GitHub Actions
- **컨테이너**: Docker Compose (전체 스택)
- **태스크 큐**: Redis + Celery (비동기 분석 처리)

## 시스템 아키텍처 (Python 백엔드 통합)

### 전체 아키텍처 다이아그램
```
┌─────────────────────────────────────────────────────────────────┐
│                         사용자 브라우저                          │
│                    (Chrome, Safari, Mobile)                     │
└────────────────────────────┬────────────────────────────────────┘
                             │ HTTPS
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│                      Next.js 프론트엔드                          │
│                         (Vercel)                                │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │  React UI Components                                      │  │
│  │  - 인증 페이지                                            │  │
│  │  - 이미지 업로드 UI                                       │  │
│  │  - 분석 결과 시각화                                        │  │
│  │  - 히스토리 대시보드                                      │  │
│  └──────────────────────────────────────────────────────────┘  │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │  next/image (이미지 최적화)                               │  │
│  │  TailwindCSS + shadcn/ui                                 │  │
│  └──────────────────────────────────────────────────────────┘  │
└────────────────────────────┬────────────────────────────────────┘
                             │ REST API (JSON)
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│                       FastAPI 백엔드                            │
│                      (Railway/Render)                           │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │  API 레이어                                                │  │
│  │  - /api/auth/* (인증)                                     │  │
│  │  - /api/analyze/* (분석)                                   │  │
│  │  - /api/user/* (사용자)                                   │  │
│  │  Swagger UI (자동 문서화)                                 │  │
│  └──────────────────────────────────────────────────────────┘  │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │  비즈니스 로직                                             │  │
│  │  - 인증/인가 (JWT)                                        │  │
│  │  - 이미지 업로드 처리                                     │  │
│  │  - 사용자 관리                                            │  │
│  │  - 분석 결과 조회                                         │  │
│  └──────────────────────────────────────────────────────────┘  │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │  SQLAlchemy ORM                                           │  │
│  │  - User 모델                                              │  │
│  │  - Analysis 모델                                          │  │
│  │  - Recommendation 모델                                    │  │
│  └──────────────────────────────────────────────────────────┘  │
└──────┬──────────────────────────┬───────────────────────────────┘
       │                          │
       │                          │ Celery Tasks
       │                          ▼
       │              ┌───────────────────────────────────────┐
       │              │         Redis 태스크 큐               │
       │              │      (분석 작업 대기열)               │
       │              └───────────────┬───────────────────────┘
       │                              │
       │                              ▼
       │              ┌───────────────────────────────────────┐
       │              │      Celery Worker (AI 분석 엔진)     │
       │              │  ┌─────────────────────────────────┐  │
       │              │  │  Python AI/ML 라이브러리        │  │
       │              │  │  - PyTorch / TensorFlow         │  │
       │              │  │  - OpenCV                       │  │
       │              │  │  - PIL / Pillow                │  │
       │              │  │  - albumentations               │  │
       │              │  └─────────────────────────────────┘  │
       │              │  ┌─────────────────────────────────┐  │
       │              │  │  피부분석 딥러닝 모델           │  │
       │              │  │  - CNN (주름 탐지)              │  │
       │              │  │  - Vision Transformer           │  │
       │              │  │  - 분류 모델 (피부타입)         │  │
       │              │  │  - 세그멘테이션 (히트맵)        │  │
       │              │  └─────────────────────────────────┘  │
       │              │  ┌─────────────────────────────────┐  │
       │              │  │  분석 파이프라인               │  │
       │              │  │  1. 이미지 로드                │  │
       │              │  │  2. 전처리 (리사이징, 정규화)   │  │
       │              │  │  3. 모델 추론                  │  │
       │              │  │  4. 후처리 (점수 계산)          │  │
       │              │  │  5. 히트맵 생성                │  │
       │              │  │  6. 결과 저장                  │  │
       │              │  └─────────────────────────────────┘  │
       │              └───────────────┬───────────────────────┘
       │                              │
       │                              │ 분석 결과
       ▼                              ▼
┌──────────────────┐      ┌───────────────────────────────────────┐
│  PostgreSQL      │◄─────│         SQLAlchemy ORM                 │
│  (Supabase/Neon) │      │      (결과 DB 저장)                   │
│  ┌────────────┐  │      └───────────────────────────────────────┘
│  │ users      │  │
│  │ analyses   │◄─┤
│  │ recommend  │  │
│  └────────────┘  │
└──────────────────┘

       │
       │ 이미지 업로드/다운로드
       ▼
┌─────────────────────────────────────────────────────────────────┐
│              파일 스토리지 (S3 / Cloudflare R2)                  │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │  원본 이미지 저장                                          │  │
│  │  - 사용자 업로드 이미지                                    │  │
│  │  - 분석 히트맵 이미지                                      │  │
│  └──────────────────────────────────────────────────────────┘  │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │  CDN 배포                                                │  │
│  │  - 빠른 이미지 전송                                       │  │
│  └──────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
```

### 아키텍처 상세 설명

#### 1. 프론트엔드 계층 (Next.js)
- **역할**: 사용자 인터페이스, 정적 파일 제공
- **기술**: React, TailwindCSS, shadcn/ui
- **배포**: Vercel
- **특징**: 이미지 최적화, SSR/SSG 지원

#### 2. API 계층 (FastAPI)
- **역할**: REST API 제공, 비즈니스 로직 처리
- **기술**: FastAPI, SQLAlchemy, Pydantic
- **배포**: Railway/Render
- **특징**: 비동기 처리, 자동 API 문서화

#### 3. AI 분석 계층 (Celery Worker)
- **역할**: 피부분석 AI 모델 추론
- **기술**: PyTorch/TensorFlow, OpenCV, Celery
- **배포**: Railway/Render (별도 Worker)
- **특징**: 비동기 분석 처리, 확장 가능

#### 4. 데이터 계층
- **PostgreSQL**: 사용자, 분석 결과 저장
- **Redis**: 태스크 큐, 세션 캐싱
- **S3/R2**: 이미지 파일 저장

#### 5. 통신 흐름
1. 사용자 → Next.js (UI 상호작용)
2. Next.js → FastAPI (REST API 호출)
3. FastAPI → S3 (이미지 저장)
4. FastAPI → Redis (분석 태스크 큐에 추가)
5. Celery Worker → Redis (태스크 수신)
6. Celery Worker → AI 모델 (분석 수행)
7. Celery Worker → PostgreSQL (결과 저장)
8. FastAPI → Next.js (결과 전송)
9. Next.js → 사용자 (결과 표시)

### 서비스 통합 방식

#### FastAPI REST API 통합
- **통신**: HTTP REST API
- **프로토콜**: JSON
- **장점**: 단일 Python 백엔드, 고성능 비동기, 자동 문서화
- **구현**:
  - Next.js → FastAPI
  - 이미지 업로드 → FastAPI → S3
  - 분석 요청 → FastAPI Background Tasks/Celery → AI 모델
  - 분석 결과 → FastAPI → Next.js

### 데이터 흐름
1. **이미지 업로드**: 사용자 → Next.js → FastAPI → S3
2. **분석 요청**: FastAPI → Celery 태스크 큐
3. **AI 추론**: Celery Worker → 모델 로드 → 분석 수행
4. **결과 저장**: Celery → PostgreSQL
5. **결과 전송**: FastAPI → Next.js → 사용자

## Docker 컨테이너화 아키텍처

### Docker Compose 구성
```yaml
version: '3.8'

services:
  # FastAPI 백엔드
  fastapi:
    build: ./backend
    ports:
      - "8000:8000"
    environment:
      - DATABASE_URL=postgresql://user:password@postgres:5432/skyweb
      - REDIS_URL=redis://redis:6379/0
      - AWS_ACCESS_KEY_ID=${AWS_ACCESS_KEY_ID}
      - AWS_SECRET_ACCESS_KEY=${AWS_SECRET_ACCESS_KEY}
      - S3_BUCKET=${S3_BUCKET}
    depends_on:
      - postgres
      - redis
    volumes:
      - ./backend:/app
      - ./uploads:/uploads
    networks:
      - skyweb-network

  # Celery Worker (AI 분석 엔진)
  celery-worker:
    build: ./backend
    command: celery -A app.tasks worker --loglevel=info
    environment:
      - DATABASE_URL=postgresql://user:password@postgres:5432/skyweb
      - REDIS_URL=redis://redis:6379/0
      - AWS_ACCESS_KEY_ID=${AWS_ACCESS_KEY_ID}
      - AWS_SECRET_ACCESS_KEY=${AWS_SECRET_ACCESS_KEY}
      - S3_BUCKET=${S3_BUCKET}
    depends_on:
      - postgres
      - redis
    volumes:
      - ./backend:/app
      - ./models:/models  # AI 모델 파일
    networks:
      - skyweb-network

  # Celery Beat (스케줄러)
  celery-beat:
    build: ./backend
    command: celery -A app.tasks beat --loglevel=info
    environment:
      - REDIS_URL=redis://redis:6379/0
    depends_on:
      - redis
    volumes:
      - ./backend:/app
    networks:
      - skyweb-network

  # PostgreSQL 데이터베이스
  postgres:
    image: postgres:15-alpine
    environment:
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=password
      - POSTGRES_DB=skyweb
    volumes:
      - postgres-data:/var/lib/postgresql/data
    ports:
      - "5432:5432"
    networks:
      - skyweb-network

  # Redis 캐시/태스크 큐
  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    volumes:
      - redis-data:/data
    networks:
      - skyweb-network

  # Flower (Celery 모니터링)
  flower:
    build: ./backend
    command: celery -A app.tasks flower --port=5555
    ports:
      - "5555:5555"
    environment:
      - REDIS_URL=redis://redis:6379/0
    depends_on:
      - redis
    networks:
      - skyweb-network

networks:
  skyweb-network:
    driver: bridge

volumes:
  postgres-data:
  redis-data:
```

### Dockerfile 구성

#### Backend Dockerfile (FastAPI + Celery)
```dockerfile
FROM python:3.10-slim

WORKDIR /app

# 시스템 의존성
RUN apt-get update && apt-get install -y \
    gcc \
    g++ \
    libpq-dev \
    && rm -rf /var/lib/apt/lists/*

# Python 의존성 복사
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# 애플리케이션 복사
COPY . .

# 모델 파일 디렉토리
RUN mkdir -p /models

# 환경 변수
ENV PYTHONUNBUFFERED=1

# 기본 명령 (FastAPI)
CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

#### requirements.txt
```
fastapi==0.104.1
uvicorn[standard]==0.24.0
sqlalchemy==2.0.23
alembic==1.12.1
psycopg2-binary==2.9.9
redis==5.0.1
celery==5.3.4
flower==2.0.1
pydantic==2.5.0
pydantic-settings==2.1.0
python-jose[cryptography]==3.3.0
passlib[bcrypt]==1.7.4
python-multipart==0.0.6
boto3==1.29.0
pillow==10.1.0
opencv-python==4.8.1.78
torch==2.1.0
torchvision==0.16.0
albumentations==1.3.1
```

### Docker 네트워크 구성

#### 네트워크 아키텍처
```
┌─────────────────────────────────────────────────────────┐
│              Docker Network: skyweb-network             │
│                                                         │
│  ┌──────────────┐  ┌──────────────┐  ┌─────────────┐ │
│  │   fastapi    │  │ celery-worker│  │   postgres  │ │
│  │   :8000      │  │              │  │   :5432     │ │
│  └──────┬───────┘  └──────┬───────┘  └──────┬──────┘ │
│         │                 │                 │         │
│         └─────────────────┼─────────────────┘         │
│                           │                           │
│                    ┌──────┴──────┐                    │
│                    │    redis    │                    │
│                    │    :6379    │                    │
│                    └─────────────┘                    │
│                                                         │
│  ┌──────────────┐                                      │
│  │   flower     │ (모니터링)                          │
│  │   :5555      │                                      │
│  └──────────────┘                                      │
└─────────────────────────────────────────────────────────┘
         │
         │ 호스트 포트 매핑
         ▼
┌─────────────────────────────────────────────────────────┐
│                    호스트 머신                            │
│  - localhost:8000 → FastAPI                             │
│  - localhost:5432 → PostgreSQL                          │
│  - localhost:6379 → Redis                                │
│  - localhost:5555 → Flower (Celery 모니터링)             │
└─────────────────────────────────────────────────────────┘
```

### 볼륨 마운트 구성

#### 데이터 영속성
- **postgres-data**: PostgreSQL 데이터 영속성
- **redis-data**: Redis 데이터 영속성
- **./backend:/app**: 개발 시 코드 실시간 반영 (핫 리로드)
- **./models:/models**: AI 모델 파일 공유
- **./uploads:/uploads**: 업로드 이미지 임시 저장

### 환경 변수 관리

#### .env 파일
```bash
# 데이터베이스
DATABASE_URL=postgresql://user:password@postgres:5432/skyweb
POSTGRES_USER=user
POSTGRES_PASSWORD=password
POSTGRES_DB=skyweb

# Redis
REDIS_URL=redis://redis:6379/0

# AWS S3
AWS_ACCESS_KEY_ID=your_access_key
AWS_SECRET_ACCESS_KEY=your_secret_key
S3_BUCKET=skyweb-uploads
AWS_REGION=us-east-1

# JWT
SECRET_KEY=your_secret_key
ALGORITHM=HS256
ACCESS_TOKEN_EXPIRE_MINUTES=30

# FastAPI
API_HOST=0.0.0.0
API_PORT=8000
DEBUG=True
```

### 컨테이너 시작/중지 명령어

#### 개발 환경
```bash
# 모든 서비스 시작
docker-compose up -d

# 로그 확인
docker-compose logs -f fastapi
docker-compose logs -f celery-worker

# 서비스 중지
docker-compose down

# 볼륨 포함 삭제
docker-compose down -v

# 특정 서비스 재시작
docker-compose restart celery-worker

# 컨테이너 접속
docker-compose exec fastapi bash
docker-compose exec celery-worker bash
```

#### 프로덕션 환경 (Kubernetes)
```bash
# Kubernetes 배포
kubectl apply -f k8s/

# Pod 상태 확인
kubectl get pods

# 로그 확인
kubectl logs -f deployment/fastapi
kubectl logs -f deployment/celery-worker

# 스케일링
kubectl scale deployment celery-worker --replicas=3
```

### Windows 개발 환경 고려사항

#### 필수 요구사항
- **Docker Desktop for Windows**: 최신 버전 설치
- **WSL2 (Windows Subsystem for Linux 2)**: 권장 백엔드
- **Windows 10/11**: WSL2 지원 버전

#### 설치 및 설정
```powershell
# WSL2 설치
wsl --install

# Docker Desktop 다운로드
# https://www.docker.com/products/docker-desktop

# Docker Desktop 설정
# Settings > General > Use WSL 2 based engine: 체크
# Settings > Resources > WSL Integration: 사용할 WSL 배포판 선택
```

#### 잠재적 문제점 및 해결책

##### 1. 파일 시스템 성능 저하
**문제**: Windows 파일 시스템과 Linux 컨테이너 간 bind mount 시 성능 저하

**해결책**:
- 프로젝트를 WSL2 파일 시스템에 저장 (`\\wsl$\Ubuntu\home\user\projects\`)
- Docker Desktop에서 WSL2 백엔드 사용
- `.dockerignore`로 불필요한 파일 제외

```dockerignore
# .dockerignore
__pycache__
*.pyc
.venv
.git
node_modules
*.log
```

##### 2. 라인 엔딩 문제 (CRLF vs LF)
**문제**: Windows (CRLF)와 Linux (LF) 간 라인 엔딩 불일치

**해결책**:
```bash
# Git 설정
git config --global core.autocrlf true

# .gitattributes
* text=auto eol=lf
*.py text eol=lf
*.sh text eol=lf
*.yml text eol=lf
*.yaml text eol=lf
```

##### 3. 포트 충돌
**문제**: Windows에서 사용 중인 포트와 컨테이너 포트 충돌

**해결책**:
```yaml
# docker-compose.yml 수정
services:
  fastapi:
    ports:
      - "8001:8000"  # 호스트 포트 변경
  postgres:
    ports:
      - "5433:5432"  # 호스트 포트 변경
  redis:
    ports:
      - "6380:6379"  # 호스트 포트 변경
```

##### 4. 리소스 사용량
**문제**: Docker Desktop이 메모리/CPU를 많이 사용

**해결책**:
- Docker Desktop > Settings > Resources
  - Memory: 4GB 이상 권장
  - CPUs: 4 이상 권장
  - Disk image size: 적절히 설정

##### 5. 경로 문제
**문제**: Windows 경로 (`C:\`) vs Linux 경로 (`/app/`)

**해결책**:
- Docker Compose가 자동 경로 변환
- 절대 경로 사용 피하고 상대 경로 사용
```yaml
volumes:
  - ./backend:/app  # 상대 경로 사용
  - ./models:/models
```

#### 권장 Windows 개발 워크플로우

##### 1. 프로젝트 구조 (WSL2)
```
\\wsl$\Ubuntu\home\user\projects\SkyWeb\
├── backend/
├── frontend/
├── docker-compose.yml
├── .env
└── .gitignore
```

##### 2. VS Code 설정
```json
// .vscode/settings.json
{
  "terminal.integrated.defaultProfile.windows": "WSL",
  "files.eol": "\n",
  "python.defaultInterpreterPath": "./backend/.venv/bin/python"
}
```

##### 3. Docker Compose 실행 (WSL2 터미널)
```bash
# WSL2 터미널에서 실행
cd ~/projects/SkyWeb
docker-compose up -d
```

##### 4. 개발 시 핫 리로드
```yaml
# docker-compose.yml
services:
  fastapi:
    command: uvicorn app.main:app --host 0.0.0.0 --port 8000 --reload
    volumes:
      - ./backend:/app
```

#### Windows 전용 Docker Compose 설정
```yaml
# docker-compose.windows.yml (선택적)
version: '3.8'

services:
  fastapi:
    ports:
      - "8001:8000"  # Windows 포트 충돌 방지
    volumes:
      - /mnt/c/projects/SkyWeb/backend:/app  # Windows 경로 마운트
```

#### 트러블슈팅

##### 컨테이너 시작 실패
```bash
# 로그 확인
docker-compose logs fastapi

# 컨테이너 재시작
docker-compose restart fastapi

# 캐시 삭제 후 재빌드
docker-compose down -v
docker-compose build --no-cache
docker-compose up -d
```

##### WSL2 네트워크 문제
```powershell
# WSL 재시작
wsl --shutdown
wsl

# DNS 문제 해결
# /etc/resolv.conf에 nameserver 8.8.8.8 추가
```

##### 파일 권한 문제
```bash
# WSL2에서 파일 권한 수정
chmod -R 755 ./backend
```

#### 성능 최적화 팁
1. 프로젝트를 WSL2 파일 시스템에 저장
2. `.dockerignore`로 불필요한 파일 제외
3. Docker Desktop 리소스 할당 증가
4. 핫 리로드 대신 필요시만 재빌드
5. 멀티스레드 빌드 사용 (`DOCKER_BUILDKIT=1`)

## Windows 전체 프로젝트 진행 고려사항

### 개발 환경 설정

#### 1. Python 개발 환경
**가상 환경 설정 (WSL2 권장)**:
```bash
# WSL2 터미널에서
cd ~/projects/SkyWeb/backend
python -m venv .venv
source .venv/bin/activate

# Windows PowerShell에서 (비추천)
cd C:\Project\SkyWeb\backend
python -m venv .venv
.venv\Scripts\activate
```

**패키지 설치**:
```bash
# WSL2에서 (권장)
pip install -r requirements.txt

# Windows에서 (CUDA 문제 가능)
pip install -r requirements.txt
```

#### 2. Node.js/Next.js 개발 환경
**설치**:
```powershell
# Windows에서 Node.js 설치
# https://nodejs.org/

# WSL2에서도 사용 가능 (권장)
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt-get install -y nodejs
```

**패키지 설치**:
```bash
# WSL2 터미널에서
cd ~/projects/SkyWeb/frontend
npm install

# Windows PowerShell에서
cd C:\Project\SkyWeb\frontend
npm install
```

#### 3. AI/ML 라이브러리 (PyTorch)
**Windows 고려사항**:
- CUDA 지원 GPU가 있는 경우 Windows에서 PyTorch 설치 용이
- WSL2에서도 CUDA 지원 (NVIDIA WSL2 드라이버 필요)
- CPU만 사용하는 경우 WSL2에서 충분

**PyTorch 설치**:
```bash
# Windows (CUDA 지원)
pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu118

# WSL2 (CUDA 지원)
pip install torch torchvision torchaudio

# CPU만 사용
pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cpu
```

### 터미널 환경

#### 권장: WSL2 터미널 사용
- Linux 명령어 사용 가능
- Docker와 호환성 우수
- 파일 시스템 성능 우수
- 대부분의 개발 도구 지원

#### 비권장: PowerShell/CMD
- Docker 성능 저하
- 경로 문제 발생
- Linux 명령어 미지원
- AI/ML 라이브러리 호환성 문제

### 파일 시스템 및 경로

#### 프로젝트 위치 권장
```
권장: \\wsl$\Ubuntu\home\user\projects\SkyWeb\
비권장: C:\Project\SkyWeb\
```

#### 경로 문제 해결
```python
# Python 코드에서 경로 처리
from pathlib import Path

# 절대 경로 사용 (크로스 플랫폼)
BASE_DIR = Path(__file__).resolve().parent.parent
MODELS_DIR = BASE_DIR / "models"
UPLOADS_DIR = BASE_DIR / "uploads"
```

### 개발 도구

#### VS Code 설정
```json
// .vscode/settings.json
{
  "terminal.integrated.defaultProfile.windows": "WSL",
  "files.eol": "\n",
  "python.defaultInterpreterPath": "./backend/.venv/bin/python",
  "python.linting.enabled": true,
  "python.formatting.provider": "black",
  "editor.formatOnSave": true
}
```

#### Git 설정
```bash
# WSL2에서
git config --global core.autocrlf input
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"

# .gitattributes
* text=auto eol=lf
*.py text eol=lf
*.js text eol=lf
*.json text eol=lf
*.yml text eol=lf
*.yaml text eol=lf
```

### 환경 변수 관리

#### .env 파일 (WSL2)
```bash
# backend/.env
DATABASE_URL=postgresql://user:password@postgres:5432/skyweb
REDIS_URL=redis://redis:6379/0
AWS_ACCESS_KEY_ID=your_key
AWS_SECRET_ACCESS_KEY=your_secret
S3_BUCKET=skyweb-uploads
SECRET_KEY=your_secret_key
```

#### Windows 환경 변수 (비권장)
```powershell
# PowerShell
$env:DATABASE_URL="postgresql://user:password@localhost:5432/skyweb"
$env:REDIS_URL="redis://localhost:6379/0"
```

### 테스트 및 디버깅

#### pytest 설정 (WSL2)
```bash
# backend/pytest.ini
[pytest]
testpaths = tests
python_files = test_*.py
python_classes = Test*
python_functions = test_*
```

#### 디버깅
```python
# VS Code launch.json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Python: FastAPI",
      "type": "python",
      "request": "launch",
      "module": "uvicorn",
      "args": ["app.main:app", "--host", "0.0.0.0", "--port", "8000", "--reload"],
      "cwd": "${workspaceFolder}/backend",
      "envFile": "${workspaceFolder}/backend/.env"
    }
  ]
}
```

### 성능 고려사항

#### AI 모델 로딩
- **WSL2 + CUDA**: 최고 성능
- **Windows + CUDA**: 우수 성능
- **WSL2 + CPU**: 양호 성능
- **Windows + CPU**: 낮은 성능

#### 이미지 처리
- OpenCV: WSL2에서 성능 우수
- Pillow: 큰 차이 없음
- GPU 가속: Windows 또는 WSL2 + CUDA

### 배포 고려사항

#### 로컬 테스트 (Docker Compose)
```bash
# WSL2에서 실행
cd ~/projects/SkyWeb
docker-compose up -d
```

#### 프로덕션 배포
- **Vercel**: 프론트엔드 (Windows에서 개발 가능)
- **Railway/Render**: 백엔드 (Docker 이미지 빌드)
- **AWS EC2**: Docker 컨테이너 배포

### 일반적인 Windows 개발 팁

1. **WSL2 사용**: 대부분의 작업을 WSL2에서 수행
2. **VS Code Remote WSL**: WSL2 파일 시스템에 직접 접근
3. **Git Bash 사용**: Windows에서 Linux 명령어 필요시
4. **파일 시스템 주의**: Windows와 WSL2 간 파일 복사 최소화
5. **환경 변수 통일**: .env 파일 사용
6. **라인 엔딩 일치**: LF 사용 (Git 설정)
7. **포트 관리**: Windows 사용 포트 확인

### Windows 전용 문제 해결

#### Python 패키지 설치 실패
```bash
# Microsoft C++ Build Tools 설치 필요
# https://visualstudio.microsoft.com/visual-cpp-build-tools/

# 또는 pre-built wheel 사용
pip install --only-binary :all: <package-name>
```

#### Node.js 권한 문제
```powershell
# PowerShell 관리자 권한으로 실행
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
```

#### Docker 권한 문제
```powershell
# Docker Desktop 재시작
# 사용자 그룹에 docker-users 추가
```

### 요약

**권장 Windows 개발 환경**:
- WSL2 (Ubuntu) + Docker Desktop
- VS Code Remote WSL 확장
- 프로젝트를 WSL2 파일 시스템에 저장
- .env 파일로 환경 변수 관리
- Git으로 라인 엔딩 통일

**비권장**:
- Windows 네이티브 Python 개발
- PowerShell에서 Docker 사용
- 프로젝트를 C:\ 드라이브에 저장
- 수동 환경 변수 설정

### FastAPI API 설계
```
POST /api/auth/register - 회원가입
POST /api/auth/login - 로그인
POST /api/auth/logout - 로그아웃
GET /api/auth/me - 현재 사용자 정보

POST /api/analyze/upload - 이미지 업로드
POST /api/analyze/process - AI 분석 요청
GET /api/analyze/result/{id} - 분석 결과 조회
GET /api/analyze/history - 분석 히스토리

GET /api/user/profile - 프로필 조회
PUT /api/user/profile - 프로필 수정
DELETE /api/user/account - 계정 삭제
```

## 핵심 기능

### 1. 사용자 인증
- 회원가입/로그인 (이메일, 소셜 로그인)
- JWT 기반 세션 관리
- 프로필 관리

### 2. 이미지 업로드
- 드래그 앤 드롭 업로드
- 카메라 촬영 지원
- 이미지 미리보기
- 다중 이미지 업로드

### 3. AI 피부분석
- **분석 항목**:
  - 피부타입 (건성, 지성, 복합성, 중성)
  - 주름/미세선 탐지
  - 색소침착/기미 분석
  - 모공 크기 분석
  - 트러블/여드름 탐지
  - 수분도 추정
  - 탄력도 측정
- **분석 결과**: 점수 기반 리포트, 시각적 히트맵

### 4. 맞춤형 솔루션 추천
- 피부 상태 기반 제품 추천
- 스킨케어 루틴 제안
- 라이프스타일 조언

### 5. 히스토리 관리
- 과거 분석 결과 조회
- 피부 상태 변화 추적 (그래프)
- 비교 기능

### 6. 관리자 기능
- 사용자 관리
- 분석 통계 대시보드
- 추천 제품 관리

## API 설계 (FastAPI)

### 인증 API
- `POST /api/auth/register` - 회원가입
- `POST /api/auth/login` - 로그인
- `POST /api/auth/logout` - 로그아웃
- `GET /api/auth/me` - 현재 사용자 정보

### 분석 API
- `POST /api/analyze/upload` - 이미지 업로드
- `POST /api/analyze/process` - AI 분석 요청
- `GET /api/analyze/result/{id}` - 분석 결과 조회
- `GET /api/analyze/history` - 분석 히스토리

### 사용자 API
- `GET /api/user/profile` - 프로필 조회
- `PUT /api/user/profile` - 프로필 수정
- `DELETE /api/user/account` - 계정 삭제

## 데이터베이스 스키마

### users 테이블
```sql
- id (UUID, PK)
- email (VARCHAR, UNIQUE)
- password_hash (VARCHAR)
- name (VARCHAR)
- created_at (TIMESTAMP)
- updated_at (TIMESTAMP)
```

### analyses 테이블
```sql
- id (UUID, PK)
- user_id (UUID, FK)
- image_url (VARCHAR)
- skin_type (VARCHAR)
- wrinkle_score (INTEGER)
- pigmentation_score (INTEGER)
- pore_score (INTEGER)
- acne_score (INTEGER)
- moisture_score (INTEGER)
- elasticity_score (INTEGER)
- overall_score (INTEGER)
- analysis_data (JSONB)
- created_at (TIMESTAMP)
```

### recommendations 테이블
```sql
- id (UUID, PK)
- analysis_id (UUID, FK)
- product_name (VARCHAR)
- category (VARCHAR)
- reason (TEXT)
- created_at (TIMESTAMP)
```

## 개발 단계 (JavaScript 초고자 고려)

### Phase 0: JavaScript 학습 (1-2주)
- [ ] JavaScript 기본 문법 학습
- [ ] React 기본 개념 학습 (컴포넌트, State, Props)
- [ ] Next.js 기본 구조 이해
- [ ] 간단한 React 프로젝트 실습

### Phase 1: FastAPI 백엔드 구축 (2주)
- [ ] FastAPI 프로젝트 초기화 및 설정
- [ ] 데이터베이스 스키마 설계 (SQLAlchemy Models)
- [ ] Alembic 마이그레이션 설정
- [ ] 인증 시스템 구현 (JWT)
- [ ] 기본 API 엔드포인트 구현

### Phase 2: 이미지 업로드 및 처리 (1주)
- [ ] FastAPI 이미지 업로드 구현
- [ ] S3/Cloudflare R2 연동
- [ ] 이미지 전처리 파이프라인 구축
- [ ] Celery 태스크 큐 설정

### Phase 3: AI 분석 통합 (2주)
- [ ] AI 모델 로드 및 테스트
- [ ] Celery Worker로 분석 태스크 구현
- [ ] 분석 결과 저장 로직
- [ ] 분석 API 완성

### Phase 4: Next.js 프론트엔드 개발 (2주)
- [ ] Next.js 프로젝트 설정
- [ ] TailwindCSS 및 shadcn/ui 구성
- [ ] 레이아웃 및 네비게이션 구현
- [ ] 인증 페이지 구현
- [ ] FastAPI와 연동 (Swagger UI 참조)

### Phase 5: 이미지 업로드 UI (1주)
- [ ] 이미지 업로드 컴포넌트 구현
- [ ] 드래그 앤 드롭 기능
- [ ] 카메라 촬영 지원
- [ ] 이미지 프리뷰 기능

### Phase 6: 분석 결과 시각화 (1주)
- [ ] 분석 결과 페이지 구현
- [ ] 점수 기반 리포트 UI
- [ ] 히트맵 시각화
- [ ] 그래프 차트 구현

### Phase 7: 추천 시스템 (1주)
- [ ] 추천 알고리즘 구현
- [ ] 제품 데이터베이스 구축
- [ ] 추천 UI 구현

### Phase 8: 히스토리 및 대시보드 (1주)
- [ ] 히스토리 조회 기능
- [ ] 데이터 시각화 (그래프)
- [ ] 비교 기능
- [ ] 관리자 대시보드

### Phase 9: 테스트 및 배포 (1주)
- [ ] 단위 테스트 작성
- [ ] 통합 테스트
- [ ] 배포 환경 설정 (Vercel + Railway)
- [ ] 프로덕션 배포

## 보안 고려사항
- 이미지 파일 암호화 저장
- HTTPS 적용
- Rate Limiting 구현
- 입력 데이터 검증
- CORS 설정
- 민감 정보 환경변수 관리

## 성능 최적화
- 이미지 CDN 활용
- 데이터베이스 인덱싱
- API 응답 캐싱
- 이미지 리사이징 및 압축
- 지연 로딩 구현

## 향후 확장 가능성
- 모바일 앱 개발 (React Native)
- AR 기반 실시간 분석
- dermatologist 연동 기능
- 커뮤니티 기능
- 구독 모델 도입
