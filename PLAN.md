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
- **백엔드 배포**: Railway, Render, 또는 AWS EC2
- **데이터베이스**: Supabase (PostgreSQL) 또는 Neon
- **파일 스토리지**: AWS S3, Cloudflare R2, 또는 aiofiles
- **CI/CD**: GitHub Actions
- **컨테이너**: Docker (백엔드)
- **태스크 큐**: Redis + Celery (비동기 분석 처리)

## 시스템 아키텍처 (Python 백엔드 통합)

### 아키텍처 개요
```
┌─────────────────┐
│   사용자 브라우저   │
└────────┬────────┘
         │ HTTPS
         ▼
┌─────────────────┐
│   Next.js 프론트  │ (Vercel)
│  - React UI      │
│  - 정적 파일      │
└────────┬────────┘
         │ REST API
         ▼
┌─────────────────┐
│   FastAPI 백엔드  │ (Railway/Render)
│  - FastAPI       │
│  - SQLAlchemy    │
│  - 인증/사용자 관리 │
│  - 이미지 업로드   │
│  - AI 분석 통합    │
│  - Celery 태스크   │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│   PostgreSQL    │ (Supabase/Neon)
│  - 사용자 데이터   │
│  - 분석 결과     │
└─────────────────┘
```

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
