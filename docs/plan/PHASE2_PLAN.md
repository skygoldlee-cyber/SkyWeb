# Phase 2 실행계획 — 업로드 + S3/R2 + 작업 큐 enqueue

> **상위 문서**: PLAN.md §3(통신 흐름), §5(analyses), §6(분석 API), §8(업로드 검증)
> **기간(현실안)**: 1–2주
> **완료 기준**: 업로드 후 `status=queued` 레코드 생성 + 작업 enqueue + **작업 ID 즉시 반환**
> **성격**: "요청-응답 안에서 동기 분석 금지" 원칙을 구조로 못 박는 단계. **실제 분석은 안 함**(Phase 3).

---

## 0. 목표 한 줄

이미지를 받아 스토리지에 저장하고, `analyses` 레코드를 `queued`로 만든 뒤 작업 큐에 넣고
**즉시 작업 ID를 반환**한다. 무거운 분석은 워커로 떠넘기는 비동기 골격을 완성한다.

---

## 1. 선행 조건

- [ ] Phase 1 완료(인증·`get_current_user`·Alembic)
- [ ] S3 또는 Cloudflare R2 버킷(개발용) + 자격증명(env)
- [ ] Redis 접근(로컬 docker 또는 관리형)

---

## 2. 작업 단위

### 2.0 의존성 설치
```bash
pip install boto3 redis celery  # S3/R2, Redis, Celery (Phase 3에서 사용)
```

### 2.1 analyses 스키마 (PLAN.md §5)
- [ ] `analyses` ORM 모델 (점수 하드코딩 금지 — JSONB + NUMERIC 원칙)
  - `id UUID PK`, `user_id FK`, `status`(queued/processing/done/failed)
  - `image_keys JSONB`(다중 뷰), `heatmap_keys JSONB`
  - `skin_type`, `overall_score NUMERIC(5,2)`, `category_scores JSONB`, `report_items JSONB`
  - `engine_version`, `score_schema_version`, `error`, `created_at`, `completed_at`
- [ ] 인덱스: `(user_id, created_at DESC)`, `(status)`
- [ ] Alembic 마이그레이션 추가 + 왕복 확인

### 2.2 스토리지 어댑터 (S3/R2)
- [ ] `app/storage.py`: `upload_original()`, `generate_signed_url()` (히트맵 업로드는 Phase 3 워커)
- [ ] **서명된 URL만 노출**(직접 공개 금지 — §8), 만료 설정
- [ ] 키 네이밍 규칙: `users/{user_id}/analyses/{analysis_id}/original_{view}.jpg`

### 2.3 업로드 엔드포인트 (§6)
- [ ] `POST /api/v1/analyses` (인증 필수)
  1. 업로드 파일 검증: **타입(jpg/png) + 크기 상한**(§8), 다중 이미지 허용(다중 뷰)
  2. S3/R2에 원본 저장 → `image_keys`
  3. `analyses` 레코드 생성 `status=queued`
  4. 작업 enqueue
  5. `{ analysis_id, status: "queued" }` **즉시 반환**
- [ ] `GET /api/v1/analyses/{id}` — 본인 소유 검사(IDOR 방지) 후 상태/결과 반환
- [ ] `GET /api/v1/analyses` — 본인 것만, 페이지네이션
- [ ] `GET /api/v1/analyses/{id}/heatmap` — 서명 URL (결과 없으면 적절한 상태코드)

### 2.4 작업 큐 enqueue (단계적 도입 — §2)
- [ ] **v1 극초기**: FastAPI `BackgroundTasks`로 enqueue 인터페이스만 잡아도 OK
- [ ] **권장**: 처음부터 enqueue 추상화(`enqueue_analysis(analysis_id, image_keys, options)`)를 만들어
      Phase 3에서 Celery로 **구현만 교체**되도록 경계 고정
- [ ] Redis 연결 설정(env), 큐 이름/직렬화 규칙 정의
- [ ] `options` 페이로드 설계: 다중 뷰 on/off, 복원(restore) on/off 등 (엔진 입력과 정합)

### 2.5 검증/에러
- [ ] Pydantic 요청 모델로 입력 검증
- [ ] 잘못된 파일/초과 크기 → 명확한 4xx + 표준 에러 스키마
- [ ] enqueue 실패 시 레코드 상태 처리(생성 롤백 또는 failed 마킹) 정책 결정

---

## 3. 산출물

1. `analyses` 테이블 + 마이그레이션
2. `POST/GET /api/v1/analyses` (+ `/{id}`, `/{id}/heatmap`)
3. 스토리지 어댑터(업로드 + 서명 URL)
4. enqueue 추상화 + Redis 연결
5. `.env.example`(키 목록만, 값 없음)
   ```bash
   S3_BUCKET_NAME=your-bucket-name
   S3_ENDPOINT_URL=https://your-endpoint.r2.cloudflarestorage.com
   AWS_ACCESS_KEY_ID=your-access-key
   AWS_SECRET_ACCESS_KEY=your-secret-key
   REDIS_URL=redis://localhost:6379/0
   ```

---

## 4. 완료 기준 (Definition of Done)

- [ ] 인증된 사용자가 이미지 업로드 → S3/R2에 원본 저장 확인
- [ ] DB에 `status=queued` 레코드 생성 + `image_keys` 채워짐
- [ ] 응답이 **즉시**(분석 대기 없이) `analysis_id`를 반환
- [ ] 큐에 작업 메시지가 쌓이는 것 확인(Redis 또는 로그)
- [ ] 타인의 `analysis_id` 조회가 차단(IDOR)

### 테스트 전략
- S3/R2 mock 사용 (moto 또는 실제 버킷 테스트)
- Redis mock 사용 (fakeredis)
- 업로드 크기/타입 검증 테스트
- IDOR 방지 테스트

---

## 5. 리스크 / 주의

- **동기 처리 유혹 금지**: 분석은 수 초~수십 초. 절대 요청 안에서 돌리지 않는다.
- **enqueue 경계 고정**: 지금 `BackgroundTasks`여도 인터페이스를 Celery 교체 가능하게. (Phase 3 비용 절감)
- **업로드 검증 빈틈**: 확장자만 믿지 말고 MIME/매직바이트 확인, 크기 상한 강제.
- **경로/SSRF**: 키 생성 시 사용자 입력 그대로 경로에 쓰지 않기(과거 SkinLens path traversal·SSRF 사례).

---

## 6. 다음 단계 연결

- Phase 3 워커가 이 큐의 메시지를 소비해 `processing → done/failed`로 전이시키고 결과를 채운다.
- `options` 페이로드 스키마는 Phase 3의 `AnalysisService.analyze(options=...)` 입력 계약과 **반드시 일치**시킬 것.
- enqueue 추상화(`enqueue_analysis`)는 Phase 3에서 Celery로 구현만 교체됨 → 인터페이스 고정 중요
