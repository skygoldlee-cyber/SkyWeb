# Phase 3 실행계획 — SkinLens 엔진 래핑 (워커)

> **상위 문서**: PLAN.md §4(AI 분석 계층 — 핵심), §3(흐름), §9(워커 호스트)
> **기간(현실안)**: 2주
> **완료 기준**: 워커가 엔진 호출 → 점수/히트맵 DB 저장, `engine_version` 기록
> **성격**: **이 프로젝트에서 가장 중요한 결정의 실현.** 신규 모델 개발 없음. 기존 `AnalysisService` 호출·운영만.

---

## 0. 목표 한 줄

작업 큐의 메시지를 소비하는 **얇은 어댑터 워커**가 기존 SkinLens `AnalysisService.analyze()`를
그대로 호출하고, 점수/히트맵/버전을 DB·스토리지에 저장한다. 워커는 분석 로직을 **소유하지 않는다**.

---

## 1. 선행 조건

- [ ] Phase 2 완료(`analyses` 레코드 `queued` 생성 + enqueue)
- [ ] SkinLens 엔진의 정식 진입점 `AnalysisService.analyze()` 입출력 **계약 확정** (§11 시작 전 항목)
- [ ] 워커 호스트 후보의 메모리/CPU(또는 GPU) 가늠 (PyTorch + 복원 모델 적재)

---

## 2. 작업 단위

### 2.0 의존성 설치
```bash
pip install celery[redis] flower  # Celery, Redis, 모니터링
# SkinLens 엔진은 별도 패키지로 설치 (2.1 참조)
```

### 2.1 엔진 연결(seam) — §4.2
- [ ] SkinLens를 워커 이미지에 **패키지로 설치** (택1)
  - **옵션 A**: 프라이빗 인덱스 (`pip install --index-url https://... skinlens`)
  - **옵션 B**: 로컬 개발 (`pip install -e ../skinlens`)
  - **옵션 C**: Docker 멀티스테이지 (`COPY --from=builder /app/skinlens ./skinlens`)
- [ ] **DTO 계약 고정**: 웹 계층은 엔진 내부 구조가 아니라 `analyze()`의 입출력에만 의존
  - 출력을 Pydantic DTO로 받음(엔진이 내부 진화해도 웹이 안 깨지게)
  - 출력 항목: 10개 직교 카테고리 점수 + 18개 리포트 항목 + 히트맵 + `engine_version`/`score_schema_version`
- [ ] **무거운 의존성 격리**: PyTorch·OpenCV·복원 모델 가중치는 **워커 이미지에만**. API 컨테이너는 경량 유지.

### 2.2 Celery 승격 (§2 단계적 도입)
- [ ] Phase 2의 enqueue 추상화 구현체를 **Celery + Redis**로 교체
- [ ] `@shared_task(bind=True, max_retries=2)` — 재시도/동시성 제어
- [ ] 워커 동시성(concurrency)을 모델 메모리에 맞춰 설정(과도한 동시 적재 방지)
- [ ] (선택) Flower 모니터링 — compose `profiles: ["monitoring"]`

### 2.3 태스크 본문 (§4.1 개념 코드 기준)
- [ ] `run_analysis(analysis_id, image_keys, options)`
  1. 상태 `processing`으로 전이
  2. S3/R2에서 이미지 로드(`download_from_s3`)
  3. **`AnalysisService.analyze(image_paths, options)` 호출** ← 신규 모델 없음
  4. 히트맵 S3 업로드 → `heatmap_keys`
  5. 결과 DB 저장(`save_analysis_result`): 점수/리포트/`engine_version`/`score_schema_version`/`completed_at`, `status=done`
  6. 예외 시 `mark_failed(analysis_id, error=str(e))` 후 raise

### 2.4 버전·재현성 (§4.2)
- [ ] 분석마다 `engine_version`, `score_schema_version` **함께 저장**
- [ ] 이유 기록: 재현성 + MDC95 기반 변화 검출/리포트 비교(Phase 6)와 정합

### 2.5 호스팅 (§4.3, §9)
- [ ] 워커 호스트 사양을 **실제 모델 메모리 사용량 측정**으로 결정 (보급형 저메모리 티어 주의)
- [ ] 모델 가중치: 이미지에 굽기 vs 시작 시 S3 다운로드 → **콜드스타트 vs 이미지 크기** 트레이드오프 선택
- [ ] 워커용 Dockerfile(PyTorch/OpenCV/복원모델 + 엔진)
  ```dockerfile
  FROM python:3.11-slim
  WORKDIR /app
  COPY requirements-worker.txt .
  RUN pip install -r requirements-worker.txt
  COPY skinlens ./skinlens  # 또는 pip install
  COPY app ./app
  CMD celery -A app.worker worker --loglevel=info
  ```

### 2.6 운영 안정성
- [ ] 멱등성: 같은 `analysis_id` 재처리 시 중복 저장 방지
- [ ] 타임아웃/실패 분류: 모델 OOM, 손상 이미지, 얼굴 미검출 등 → `error`에 사유 구분 저장
- [ ] 로깅: 분석별/실행별 로그 (SkinLens 본체의 per-execution 로깅 관례 준용)

---

## 3. 산출물

1. Celery 워커(엔진 어댑터) + 워커 Dockerfile
2. 엔진 입출력 Pydantic DTO(계약)
3. `processing → done/failed` 상태 전이 + 결과/버전 저장
4. 워커 호스트 사양·비용 실측 메모
5. `.env.example`(키 목록만, 값 없음)
   ```bash
   CELERY_BROKER_URL=redis://localhost:6379/0
   CELERY_RESULT_BACKEND=redis://localhost:6379/0
   SKINLENS_ENGINE_VERSION=1.0.0
   WORKER_CONCURRENCY=2
   ```

---

## 4. 완료 기준 (Definition of Done)

- [ ] 업로드 → (워커) → `done` 까지 E2E 1건 성공
- [ ] DB에 `category_scores`(10), `report_items`(18), `overall_score`, `heatmap_keys`, `engine_version` 채워짐
- [ ] 실패 케이스(손상 이미지 등)가 `failed` + `error` 사유로 안전하게 마감
- [ ] API 컨테이너 이미지에 PyTorch/복원모델이 **포함되지 않음** 확인

### 테스트 전략
- 엔진 `AnalysisService.analyze()` mock 사용 (실제 엔진 호출 없이 워커 로직 테스트)
- Celery 태스크 동기 실행 테스트 (`task.apply()`)
- 실패 케이스 모킹 (OOM, 손상 이미지, 얼굴 미검출)
- 멱등성 테스트 (동일 analysis_id 재처리)

---

## 5. 리스크 / 주의

- **엔진을 다시 만들지 말 것**: 분석 로직은 이미 SkinLens에 있다. 워커는 호출·저장만.
- **계약 누수 금지**: 엔진 내부 클래스/필드에 직접 의존하면 엔진 진화 시 웹이 깨진다 → DTO로 차단.
- **메모리 폭주**: 워커 동시성 × 모델 메모리. 측정 없이 동시성 올리면 OOM.
- **자격증명**: 워커 배포 산출물(zip/이미지)에 평문 시크릿 금지(반복 적발 이력).
- **WebSocket/비동기 함정**: 과거 `asyncio.run`을 ThreadPoolExecutor에서 호출한 버그 사례 — 워커는 동기 태스크로 단순하게.

---

## 6. 다음 단계 연결

- Phase 5(시각화)가 여기 저장한 `category_scores`/`report_items`/`heatmap_keys`를 렌더한다 → **JSONB 구조(키 이름) 확정**이 곧 프론트 계약.
- Phase 6(추이/비교)이 `engine_version`/`score_schema_version`으로 비교 가능 여부를 판단한다.

---

## 7. PWA 통합 고려사항 (워커 관점)

Phase 3은 워커 단계이지만, PWA 통합을 위해 다음 사항을 고려합니다.

### PWA와 워커 통합 포인트
- **분석 완료 알림**: 워커가 분석 완료 시 PWA 푸시 알림 트리거 (Phase 4 이후 구현)
- **오프라인 결과 캐싱**: 분석 결과를 PWA가 오프라인에서 조회할 수 있도록 캐싱 전략 고려
- **진행 상태 실시간 전송**: 워커 진행 상태를 SSE/WebSocket으로 PWA에 전달 가능성 (v2)
- **결과 데이터 최적화**: PWA 캐싱을 위해 결과 데이터 크기 최적화

### 검증 체크리스트 (워커)
- [ ] 분석 완료 이벤트 발생 (PWA 푸시 알림 연동 준비)
- [ ] 결과 데이터 크기 최적화 (PWA 캐싱 효율)
- [ ] 진행 상태 로깅 (PWA 실시간 상태 표시 준비)
- [ ] 에러 상태 세분화 (PWA 사용자 친화적 에러 메시지)

> 실제 PWA 구현은 Phase 4(프론트)에서 시작됩니다.
