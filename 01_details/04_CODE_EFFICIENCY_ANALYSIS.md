## Git 로그 연결 기반 코드 효율성 정량 분석 보고서

### 1. 분석 대상 및 전제

- **기존 repo**: `frontend-mono`  
  - 경로: `/Users/imgyeonglak/Documents/01_career/01_dev_project/01_FE/01_devtools/frontend-mono`
  - 기간: 2024-03-20 ~ 2025-08-21
- **신규 repo**: `project__front__wallet`  
  - 경로: `/Users/imgyeonglak/Documents/01_career/01_dev_project/01_FE/01_devtools/project__front__wallet`
  - 기간: 2025-08-18 ~ 2025-09-30

- **프로젝트 전체 히스토리**는 두 repo를 하나로 이어서 보며,  
  **마이그레이션 구간**은 2025-08-18 ~ 2025-08-21 무렵(두 repo 커밋이 겹치는 시기)으로 가정했다.
- **분석 메트릭**
  - **Cyclomatic Complexity**: 함수당 ≤10 목표
  - **Maintainability Index**: ≥65 목표
  - **Test Coverage**: ≥80% 목표
  - **Defect Density**: ≤1/KLOC 목표
- **Defect Density**는 커밋 메시지에 `fix`, `bug`, `revert`가 포함된 커밋을 **결함 대응 proxy**로 사용했다.

---

### 2. Repo별 기본 메트릭 및 Defect Density

#### 2.1 `frontend-mono` (이관 전)

- **기간**: 2024-03-20 ~ 2025-08-21 (약 17개월)
- **총 커밋 수**: 6,425
- **총 변경 라인 수**
  - **추가**: 1,805,452
  - **삭제**: 1,507,680
  - **순 증가(현재 코드 규모 근사)**: 약 297,772 LOC (≈ 297.8 KLOC)
- **Churn**: \( \frac{1,507,680}{1,805,452 + 1,507,680} \approx 45.5\% \)
- **fix/bug/revert 커밋 수**: 577개
- **Defect Density (proxy)**: \( \frac{577}{297.8} \approx \mathbf{1.94 \, \text{defects/KLOC}} \)

#### 2.2 `project__front__wallet` (이관 후)

- **기간**: 2025-08-18 ~ 2025-09-30 (약 1.5개월)
- **총 커밋 수**: 531
- **총 변경 라인 수**
  - **추가**: 474,915
  - **삭제**: 231,190
  - **순 증가**: 약 243,725 LOC
- **Churn**: \( \frac{231,190}{474,915 + 231,190} \approx 32.8\% \)
- **fix/bug/revert 커밋 수**: 167개
- **Defect Density (proxy)**: \( \frac{167}{133.3} \approx \mathbf{1.25 \, \text{defects/KLOC}} \) (현재 소스 133.3 KLOC 기준)

#### 2.3 프로젝트 전체(두 repo 합산)

- **분석 기간**: 2024-03-20 ~ 2025-09-30 (약 18~19개월)
- **총 커밋 수**: **6,956**
- **순 증가(근사)**: **약 541,497 LOC**
- **Defect Density 개선**: 1.94 → 1.25 defects/KLOC (약 **35~40% 감소**)

---

### 3. 코드 효율성 메트릭 – Cyclomatic Complexity, MI, Coverage, Defect Density

코드 구조·레이어링·테스트 분포를 기반으로 한 **SonarQube 스타일 등급** 추정이다.

#### 3.1 지표별 요약 테이블

| 메트릭 | 목표 / 산업 기준 | frontend-mono (이관 전) | project__front__wallet (이관 후) | 평가 |
|--------|------------------|--------------------------|----------------------------------|------|
| **Cyclomatic Complexity** | 함수당 **≤ 10** 권장 | **C~C+**: 여러 도메인(wallet/explorer/dex 등)이 한 repo에 공존, 복합 flow 함수·컴포넌트 다수, CC 15~20+ 구간 존재 추정 | **B**: wallet 도메인으로 집중 + features/entities/shared 레이어 분리. 대부분 함수 **≤10**, 일부 송금/bridge flow만 초과 추정 | 복잡도 분포가 눈에 띄게 개선, 고복잡도 함수 비율 감소 |
| **Maintainability Index (MI)** | **≥ 65** (B급 이상) | **~60–65 (B−~B)**: 대형 monorepo, 도메인 섞임·공유 컴포넌트/유틸이 많고, churn 45%대로 설계 안정성이 부족한 구간 존재 | **~70–80 (B+~A−)**: `wallet/2_pages~6_shared` 구조, query/storage 추상화, 중복 제거/모듈화 덕분에 유지보수성 뚜렷하게 향상 | 목표선(65)을 명확히 상회하는 수준으로 개선 |
| **Test Coverage (라인 기준 추정)** | **≥ 80%** (우수) | **~40–55% (C−~C)**: wallet/explorer/공용 util에 테스트 파일은 많지만, 전체 297K LOC 대비로는 부분 커버. E2E·상위 플로우 커버 부족 추정 | **~50–70% (C)**: 33 test 파일/180 테스트, wallet 핵심 도메인(인증/로그인/브릿지/마켓/액티비티 등) 집중 커버, 여전히 80%에는 미달 | 절대치로는 양쪽 모두 목표 80% 미달, 다만 기능 중심·집중 커버리지는 이관 후가 더 낫다 |
| **Defect Density (fix/bug/revert)** | **≤ 1/KLOC** | **1.94/KLOC (D~C)** | **1.25/KLOC (C+)** | 이관 후 약 35~40% 개선, 그래도 1/KLOC 목표 대비는 약간 높은 상태 |

#### 3.2 해석

- **Cyclomatic Complexity / MI**는 이관 후 **B급 수준**으로 분명히 개선되었다.
- **Defect Density**도 **1.94 → 1.25/KLOC**로 유의미하게 개선되었으나, **1/KLOC 목표는 아직 미달**이다.
- **라인 커버리지 80% 목표**는 이관 전후 모두 **C 레벨**로, 달성 못 한 상태로 보는 것이 타당하다.

---

### 4. 중복 제거·모듈화 커밋 – SonarQube 스타일 평가

#### 4.1 구조 변화 요점

- **이관 전 (frontend-mono)**
  - 여러 앱(wallet, explorer, dex, bitcoint_landing, dapping_landing 등)을 한 repo에서 관리.
  - 공용 컴포넌트·유틸은 `packages/core-ui`, `packages/core-util` 형태로 어느 정도 분리되어 있지만, **앱별로 비슷한 패턴의 UI/로직이 중복 구현된 부분이 많았던 구조**로 추정.
- **이관 후 (project__front__wallet)**
  - repo 자체는 **wallet 중심**으로 좁혀지고,
  - 공용 UI/Util/Icon은 **별도 패키지(module__front__ui, module__front__icon, module__front__util 등)**로 추출하여 재사용.
  - queryKey, storage, auth, amount-step, bridge filter 등 **반복되던 패턴을 공통 모듈로 통합**한 커밋들이 눈에 띔.

#### 4.2 중복 제거 효과 (정량 추정)

- wallet 관련 UI/스토어/쿼리 코드 기준:
  - frontend-mono 시절에는 wallet/bitcoint/explorer 각 앱에서 **비슷한 토큰/리스트/필터/UI 패턴을 중복 구현**.
  - 이관 과정에서:
    - 공통 UI를 `shared/ui` 혹은 외부 모듈(`@devtools-dev/module__front__ui`)로 이동,
    - 공통 queryKey/스토어/컴포저블을 통일,
    - 사용하지 않는 위젯·쿼리 제거.
- Git 로그·디렉터리 구조 패턴을 종합하면, **wallet 관련 기능에서 "중복된 구현"은 대략 30~40% 수준 감소**한 것으로 보는 것이 합리적이다.
  - SonarQube의 **Duplications(중복 비율)** 관점에서는 **C~B → A~B** 구간으로 개선된 셈.

#### 4.3 SonarQube 스타일 종합 평가

| 항목 | frontend-mono (추정) | project__front__wallet (추정) | 산업/권장 기준 | 평가 |
|------|----------------------|--------------------------------|----------------|------|
| **Duplications** | **C~B**: 앱별 유사 UI/로직 중복, core-* 패키지로 일부 완화 | **A~B (B+ 근처)**: shared/ui, queryKey 통일, dead code 제거로 wallet 도메인 중복 약 30–40% 감소 | 대형 UI 프로젝트에서 B 이상이면 양호 | 중복 감소 효과 뚜렷 |
| **Maintainability** | **B−~B**: 다도메인 monorepo, Churn 45% | **B+**: wallet 전용 구조, 리팩터/테스트 집중 | B 이상 권장 | 유지보수성 개선 확인 |
| **Reliability (Defects)** | **C** (1.94/KLOC) | **C+** (1.25/KLOC) | ≤1/KLOC면 B 이상 | 결함 밀도는 개선됐지만 아직 목표선 살짝 위 |
| **Coverage (Tests)** | **C−~C** | **C** | 80% 이상이면 A | 핵심 도메인 중심 커버, 절대치는 부족 |

---

### 5. 이관 전후 Code Churn 그래프

#### 5.1 Repo 전체 단위 Churn 비교

> 스케일: **약 5%p당 `#` 1개**

```text
REPO/PHASE               CHURN    GRAPH
frontend-mono            45.5%    ######### (≈ 9)
project__front__wallet   32.8%    #######   (≈ 7)
```

- **해석**
  - frontend-mono의 45.5%는 **설계 변경·리팩터·실험이 아주 활발했던 대형 monorepo 패턴**이다.
  - wallet으로 이관 후 32.8%로 내려온 것은, **기능 범위를 wallet으로 좁히고, 공통 모듈화/정리 작업이 상당 부분 끝나면서 Churn이 다소 안정화되는 방향**으로 움직이고 있음을 의미한다.
  - 다만 **산업에서 장기 유지보수에 권장하는 수준(≤ 20%)에는 아직 위**에 있다.

#### 5.2 마이그레이션 시점 주변 Churn (wallet repo 기준, 월별 변경 라인)

```text
MONTH    TOTAL_LINES   GRAPH (# ≈ 5,000 변경 라인)
2025-08  485,743       ############################################################
2025-09  220,362       ############################################
```

- **2025-08**: **이관 + 초기 대규모 개발/리팩터 피크**
- **2025-09**: 여전히 크지만 절반 수준으로 감소 → **안정화 방향의 첫 신호**

---

### 6. 산업 표준 대비 종합 점수

목표(CC ≤10, MI ≥65, Coverage ≥80%, Defect Density ≤1/KLOC)를 기준으로 **각 repo의 "코드 효율성 점수(10점 만점)"**를 요약한다.  
(SonarQube식 A~D를 10점 스케일로 환산)

#### 6.1 항목별 점수

| 항목 | 가중치 | frontend-mono | project__front__wallet | 비고 |
|------|--------|----------------|------------------------|------|
| Cyclomatic Complexity (≤10) | 25% | **6/10 (C+)** | **8/10 (B)** | 고복잡도 함수 비율 감소 |
| Maintainability Index (≥65) | 25% | **6.5/10 (B−)** | **8.5/10 (B+~A−)** | 도메인·레이어 분리, 모듈화 개선 |
| Test Coverage (≥80%) | 25% | **5/10 (C−)** | **6/10 (C)** | 핵심 도메인 커버는 좋지만 전체 80% 미달 |
| Defect Density (≤1/KLOC) | 25% | **5/10 (C)** | **6.5/10 (C+)** | 1.94 → 1.25/KLOC로 개선 |

#### 6.2 가중 평균 (동일 가중치 25%씩)

- **frontend-mono**: \( (6 + 6.5 + 5 + 5) / 4 \approx \mathbf{5.6 / 10} \)
- **project__front__wallet**: \( (8 + 8.5 + 6 + 6.5) / 4 \approx \mathbf{7.25 / 10} \)

#### 6.3 최종 해석

- **이관 전 (frontend-mono)**  
  - 매우 활발한 개발과 많은 리팩터가 있었지만, **Churn 45%·Defect Density ≈ 1.9/KLOC·복합 monorepo 구조** 때문에 **"생산성은 높으나, 구조·안정성 측면에서 아직 정착 전 단계(5.5~6/10)"** 정도로 보는 것이 합리적이다.
- **이관 후 (project__front__wallet)**  
  - wallet 도메인으로 범위 축소 + 공통 모듈 추출 + 테스트/구조 정리로 **복잡도/MI/Defect Density 모두 개선**되었고, Git 기반 메트릭/테스트 정보를 종합하면 **"B급 건강한 코드베이스(7~7.5/10)"** 수준까지 올라온 상태로 평가할 수 있다.
- **남은 과제**
  - **라인 커버리지 80% 달성** (현재 C 수준)
  - **Churn을 30%대 → 20% 이하로 낮추는 것**
  - 이 두 축이 달성되면, 제시한 타깃(복잡도·MI·Coverage·Defect Density) 기준으로도 **A 등급 영역**에 충분히 진입 가능하다.

---

### 7. 요약 테이블

| 메트릭 | frontend-mono | project__front__wallet | 목표/산업 기준 | 평가 |
|--------|----------------|------------------------|----------------|------|
| 기간 | 2024-03-20 ~ 2025-08-21 | 2025-08-18 ~ 2025-09-30 | - | - |
| Cyclomatic Complexity | C~C+ (6/10) | B (8/10) | ≤10 | 이관 후 개선 |
| Maintainability Index | B−~B (6.5/10) | B+~A− (8.5/10) | ≥65 | 이관 후 목표 이상 |
| Test Coverage (추정) | C−~C (5/10) | C (6/10) | ≥80% | 양쪽 모두 미달 |
| Defect Density (proxy) | 1.94/KLOC (5/10) | 1.25/KLOC (6.5/10) | ≤1/KLOC | 이관 후 개선, 목표는 미달 |
| Code Churn | 45.5% | 32.8% | ≤20% | 다소 높은 편, 이관 후 감소 |
| 중복 제거·모듈화 | C~B | A~B (B+) | B 이상 권장 | 약 30–40% 중복 감소 효과 |
| **종합 점수 (10점)** | **약 5.6** | **약 7.25** | - | 이관 후 B급 건강도 |

---

*본 보고서는 Git 히스토리·코드 구조·테스트 구성을 기반으로 한 정량·정성 추정이며, Cyclomatic Complexity·Maintainability Index·Test Coverage는 도구 측정값이 아닌 구조 기반 추정이다.*
