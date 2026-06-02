# 블로그 포스트 디자인 스펙
## 금융 정산 마이그레이션에서 소수점 정밀도를 보장하는 법: 6종 데이터베이스 실증 가이드

**작성일**: 2026-06-02  
**상태**: 승인 완료  
**원본 파일**: `bq-migration2.md`

---

## 1. 개요

### 목표
`bq-migration2.md` 내용을 블로그에 게시 가능한 형태로 재구성한다. 실제 고객 케이스(카드사 리볼빙 수수료)를 가상 케이스(대출 이자 계산)로 대체하고, 금융권 고정밀 정산 환경에서 다중 데이터베이스를 실증 검증한 기술 가이드로 포지셔닝한다.

### 포스트 메타데이터

```yaml
title: "금융 정산 마이그레이션에서 소수점 정밀도를 보장하는 법: 6종 데이터베이스 실증 가이드"
categories: data
tags: [BigQuery, Oracle, Spanner, PostgreSQL, MySQL, Presto, Trino, 소수점 정밀도, 마이그레이션, 금융 정산]
```

### 포스트 성격
- **깊이**: 심층 기술 백서형 (원본 수준의 코드·수식·검증 데이터 포함)
- **훅**: 엔지니어링 원칙/레퍼런스 가이드형 — 독자 반응 "북마크해두자"
- **구조**: 문제→원인→해결 선형 흐름

---

## 2. 가상 케이스 매핑

### 핵심 대체 원칙
실제 케이스가 드러나지 않도록 도메인, 컬럼명, 수식 상수를 모두 변경한다.  
통계적 수치(일치율 %)는 플랫폼 고유 특성이므로 그대로 사용한다.  
특정 오차 발생 사례의 대출관리번호·금액은 새로 창작한다.

### 컬럼 매핑

| 원본 (리볼빙 수수료) | 대체 (대출 이자) | 설명 |
|---|---|---|
| `월TS카드매출` (테이블) | `월별대출이자산출` | 기준월 대출 이자 정산 테이블 |
| `회원고객번호` | `고객관리번호` | 고객 식별 번호 |
| `매출관리번호` | `대출관리번호` | 대출 건 식별 번호 |
| `매출액` | `대출잔액` | 이자 계산 기준 잔액 |
| `리볼빙금리` | `약정금리` | 연 이자율 (%) |
| `매출확정일자` | `대출실행일자` | 이자 기산 시작일 |
| `최초결제일자` | `이자계산기준일` | 이자 정산 기준일 |

### 수식 대체

**원본**:
```
TRUNC((매출액 × (리볼빙금리/100)) × ((최초결제일자 - 매출확정일자)/365 + 0.375), 0)
```

**대체**:
```
TRUNC((대출잔액 × (약정금리/100)) × ((이자계산기준일 - 대출실행일자)/365 + 30/365), 0)
```

**상수 변경 이유**: 원본의 `0.375`와 동일한 숫자를 유지하면 원본 케이스가 노출될 수 있으므로 `30/365`로 변경.  
`30`의 내러티브: "금융기관 내부 규정상 이자는 대출 실행일 익일부터 계산되며, 정산 주기 내 이자 발생 하한 보장을 위한 기산가산일수(30일) 조건이 수식에 포함된다."

**후순위 나눗셈 적용 (TO-BE)**:
```
TRUNC((대출잔액 × 약정금리 × (경과일수 + 30)) / 36500, 0)
```

### Python 스크립트 수정 사항
- `0.375` → `30/365` (또는 상수 `30`)
- `136.875` → `30`
- `36500` 분모는 동일 유지

---

## 3. 핵심 용어

| 항목 | 채택 용어 | 비고 |
|---|---|---|
| 기법 명칭 | **후순위 나눗셈** | 첫 등장 시 `후순위 나눗셈(Division Deferral)` 병기 |
| 기존 용어 | 지연 나눗셈(Delay Division) | 원본 문서 용어, 본문에서 대체 |

**채택 이유**: "지연"은 시간적 개념, "후순위"는 연산 순서 개념으로 기법의 본질을 더 정확히 표현. 금융권에서 익숙한 어휘("후순위채권" 등)라 독자 친화적.

---

## 4. 전체 목차 및 섹션별 설계

### Section 1. 도입

**목적**: 문제의 중요성 각인  
**내용**:
- "정산 시스템에서 1원은 단순한 반올림 오차가 아닙니다" 리드
- 금융 정산에서 허용 오차가 0인 이유 (감사, 규제, 고객 신뢰)
- Oracle에서 완벽하던 대출 이자 산출 쿼리를 BigQuery로 이식했을 때 발생하는 현상 예고
- 이 포스트가 다루는 것: 원인 분석 + 6종 DB 실증 + 재발 방지 수칙

---

### Section 2. 마이그레이션 개요

**목적**: 검증 범위와 결론 선요약 제시  
**내용**:
- 원천: Oracle Database (대출 이자 산출, 10만 건)
- 검증 대상 6종: BigQuery, Cloud Spanner, PostgreSQL, MySQL, Presto, Trino
- 검증 목표: Oracle 결과와 1원도 다르지 않음

**종합 결과 선요약표** (상단 배치):

| 데이터베이스 | 수치 타입 | AS-IS 일치율 | 후순위 나눗셈 적용 |
|---|---|---|---|
| Oracle | `NUMBER` | 100.0000% | 100.0000% |
| BigQuery | `BIGNUMERIC` | 99.9990% | 100.0000% |
| Cloud Spanner | `NUMERIC` | 99.9840% | 100.0000% |
| PostgreSQL | `NUMERIC` | 100.0000% | 100.0000% |
| MySQL | `DECIMAL` | 99.9980% | 100.0000% |
| Trino | `DECIMAL` | 81.7680% | 100.0000% |
| Presto | `DECIMAL` | 4.5840% | 100.0000% |

---

### Section 3. 대출 이자 수식 & 스키마 이행

**목적**: 검증 대상 수식과 데이터 구조 확립  
**내용**:
- 스키마 매핑 테이블 (Oracle → BigQuery, `BIGNUMERIC` 선택 이유)
- `schema.json` (컬럼명 대체 버전)
- `prepare_and_load.py` (컬럼명 대체 버전)

**수식 정의**:

$$
\text{이자} = \text{TRUNC}\!\left(\text{대출잔액} \times \frac{\text{약정금리}}{100} \times \frac{\text{경과일수} + 30}{365},\ 0\right)
$$

Oracle 원천 SQL (대체 버전):
```sql
SELECT TRUNC(
  (N10.대출잔액 * (N10.약정금리/100)) *
  ((N10.이자계산기준일 - N10.대출실행일자)/365 + 30/365)
, 0) AS 이자금액
FROM 월별대출이자산출 N10
```

---

### Section 4. 문제 발견: AS-IS 쿼리의 1원 오차

**목적**: 문제를 구체적 수치로 실증  
**내용**:
- BigQuery AS-IS 직접 번역 쿼리
- 30건 초기 검증: `TEST001` 1원 오차 발견
- 5,003건 정수 경계 시뮬레이션 결과

**핵심 수치**:
- AS-IS 오차율: **47.43%** (5,003건 중 2,373건)
- TO-BE 오차율: **0.00%**
- 오차 패턴: 항상 `-1원` (하향 절사)

---

### Section 5. 원인 분석: DB별 수치 연산 아키텍처

**목적**: 플랫폼별 정밀도 처리 방식의 근본적 차이 해설  

#### 5.1 플랫폼별 정밀도 처리 방식

중간 나눗셈에서 무한 순환소수 발생:

$$
\frac{30}{365} = 0.08219178082191780821\ldots \quad \text{(무한 순환소수)}
$$

| 플랫폼 | 처리 방식 | 오차 여부 |
|---|---|---|
| Oracle `NUMBER` | Base-100 소프트웨어 가상 레지스터, 80자리 임시 버퍼, 조기 정규화 없음 | 없음 |
| BigQuery `BIGNUMERIC` | scale=38 고정, 연산 단계마다 강제 정규화 | 있음 |
| Spanner `NUMERIC` | scale=9, ROUND_HALF_UP | 있음 (양방향) |
| PostgreSQL `NUMERIC` | 임의 정밀도(Arbitrary Precision), Oracle과 동일 | 없음 |
| MySQL `DECIMAL` | `div_precision_increment` 한계 | 있음 |
| Presto/Trino | `max(s1,s2)` 엄격 적용, 조기 스케일 축소 | 심각 |

Oracle 80자리 임시 버퍼:

$$
\text{임시 버퍼} = 40\text{ bytes (Base-100)} \equiv 80\text{ 십진 자리}
$$

$$
\frac{30}{365} \approx 0.\underbrace{082191780\ldots}_{80\text{자리 보존}} \quad \Rightarrow \quad \text{중간 절사 없음}
$$

오차 발생 조건:

$$
\text{참값} = N \in \mathbb{Z} \quad \Rightarrow \quad \text{AS-IS 결과} = N - 1 \text{ (하향 절사)}
$$

#### 5.2 David Goldberg 논문과의 간극

- 고전적 IEEE 754 부동소수점 이론 vs. 현대 분산 DWH의 고정소수점 정규화 오차
- MPP 환경의 비결정론적 셔플, CBO 수식 변환이 추가 변수로 개입
- Goldberg 이론만으로는 현대 분산 정산 시스템의 오차를 완전히 설명 불가

#### 5.3 UNION ALL과 묵시적 타입 격하

- `BIGNUMERIC` + `FLOAT64` 병합 시 `FLOAT64`로 강제 격하
- 6종 DB 교차 실증 (`pg_typeof()`, `DESCRIBE`, `typeof()`)
- 실증 결과 대조표 포함

---

### Section 6. 해결: 후순위 나눗셈(Division Deferral)

**목적**: 해결책의 원리와 구현을 명확히 제시  

**수학적 동치 증명**:

AS-IS:

$$
\text{이자}_{\text{AS-IS}} = \text{TRUNC}\!\left(\text{잔액} \times \frac{\text{금리}}{100} \times \left(\frac{D}{365} + \frac{30}{365}\right),\ 0\right)
$$

대수적 통분:

$$
= \text{TRUNC}\!\left(\text{잔액} \times \frac{\text{금리}}{100} \times \frac{D + 30}{365},\ 0\right)
$$

$$
= \text{TRUNC}\!\left(\frac{\text{잔액} \times \text{금리} \times (D + 30)}{36{,}500},\ 0\right)
$$

$$
\therefore\quad \text{이자}_{\text{TO-BE}} = \text{TRUNC}\!\left(\frac{\text{잔액} \times \text{금리} \times (D + 30)}{36{,}500},\ 0\right) \quad \blacksquare
$$

**핵심 아이디어**: 모든 중간 나눗셈을 곱셈으로 통합, 단 1회 나눗셈을 `TRUNC` 직전으로 후순위 배치.

**BigQuery TO-BE 쿼리**:
```sql
SELECT 대출관리번호,
       TRUNC(
         (CAST(대출잔액 AS BIGNUMERIC)
          * CAST(약정금리 AS BIGNUMERIC)
          * (CAST(DATE_DIFF(이자계산기준일, 대출실행일자, DAY) AS BIGNUMERIC)
             + BIGNUMERIC '30'))
         / BIGNUMERIC '36500'
       , 0) AS 이자금액
  FROM `월별대출이자산출`
```

**UNION ALL 4-Tier 방어 전략**:
1. Tier 1: 모든 브랜치 수치 컬럼 명시적 `CAST(... AS BIGNUMERIC)`
2. Tier 2: 소수 리터럴 `BIGNUMERIC '값'` 프리픽스 표기
3. Tier 3: 나눗셈 포함 수식의 피연산자 사전 업캐스팅
4. Tier 4: 마이그레이션 린터로 빌드 타임 사전 검증

---

### Section 7. 6종 DB 교차 검증 결과 (10만 건)

**목적**: 결론의 신뢰성을 대규모 실증 데이터로 뒷받침  
**내용**:
- `compare_results.py` 스크립트 (상수 수정 버전: `0.375` → `30/365`)
- `verify.py` 정수 경계 시뮬레이션 (상수 수정: `136.875` → `30`)
- DB별 오차 발생 사례 (대출관리번호·금액 새로 창작)
- Spanner 양방향 오차(+1원/-1원) 특성 설명 (BigQuery는 항상 -1원)
- `bq CLI --max_rows=150000` 파라미터 세팅 이유

---

### Section 8. 프로덕션 방어 수칙

**목적**: 북마크 가치 확보, 실무 즉시 적용 가능  
**형태**: 체크리스트

```
□ 모든 수치 컬럼에 명시적 CAST 선언 (BIGNUMERIC/DECIMAL)
□ 소수 리터럴에 타입 프리픽스 사용 (BIGNUMERIC '30', DECIMAL '30')
□ UNION ALL 전 브랜치 타입 통일 (Tier 1)
□ CTE 중간 결과 컬럼 타입 명시적 고정
□ 나눗셈은 항상 마지막 단계로 — 후순위 나눗셈(Division Deferral)
□ 대량 검증 시 bq CLI --max_rows 명시
□ 마이그레이션 전 정수 경계 케이스 전용 시뮬레이션 실행
```

---

### Section 9. 결론

**목적**: 핵심 교훈 압축 전달  
**내용**:
- 후순위 나눗셈 하나로 6종 DB 전체 100.0000% 달성
- 핵심 교훈: "수학적으로 동치인 수식이라도 연산 순서가 정밀도를 결정한다"
- Oracle에서 오차가 없던 이유는 운이 아니라 80자리 임시 버퍼 아키텍처 덕분
- 플랫폼 마이그레이션 시 수식 재검토는 선택이 아닌 필수

---

## 5. 수식 렌더링 환경

- **엔진**: KaTeX (이미 `_layouts/default.html`에 설치됨)
- **블록 수식**: `$$...$$`
- **인라인 수식**: `$...$`
- **별도 설정 불필요**

---

## 6. 참고문헌 및 공식 문서 인용

포스트 내 관련 섹션에 원문 인용과 링크를 유지한다.

### David Goldberg 논문 (Section 5.2)

> Goldberg, D. (1991). *What Every Computer Scientist Should Know About Floating-Point Arithmetic*. ACM Computing Surveys, 23(1), 5–48.

- DOI: [10.1145/103162.103163](https://dl.acm.org/doi/10.1145/103162.103163)
- Oracle 호스팅 전문: [docs.oracle.com/…/ncg_goldberg.html](https://docs.oracle.com/cd/E19957-01/806-3568/ncg_goldberg.html)

### 각 데이터베이스 공식 문서

| 플랫폼 | 문서 항목 | URL |
|---|---|---|
| Oracle | Numeric Data Types (NUMBER) | https://docs.oracle.com/en/database/oracle/oracle-database/21/sqlrf/Data-Types.html |
| BigQuery | NUMERIC / BIGNUMERIC types | https://cloud.google.com/bigquery/docs/reference/standard-sql/data-types#numeric_types |
| Cloud Spanner | NUMERIC type | https://cloud.google.com/spanner/docs/reference/standard-sql/data-types#numeric_type |
| PostgreSQL | Numeric Types (NUMERIC/DECIMAL) | https://www.postgresql.org/docs/current/datatype-numeric.html |
| MySQL | Fixed-Point Types (DECIMAL) | https://dev.mysql.com/doc/refman/8.0/en/fixed-point-types.html |
| Trino | DECIMAL type | https://trino.io/docs/current/language/types.html#decimal |
| Presto | DECIMAL type | https://prestodb.io/docs/current/language/types.html#decimal |

### 인용 배치 원칙

- **Section 5.2**: Goldberg 논문 제목·연도·DOI 링크 인용, 논문이 다루지 못하는 현대 분산 DWH 한계 서술 시 출처 명시
- **Section 5.1**: 각 DB 타입 설명 후 해당 공식 문서 링크를 인라인으로 배치 (예: `BIGNUMERIC` 설명 → BigQuery 문서 링크)
- **Section 3**: Oracle NUMBER 공식 문서 링크를 스키마 매핑 설명에 포함
- 원문 인용이 있는 경우 인용 블록(`>`)으로 표시

---

## 7. 구현 시 주의사항

1. **오차 사례 수치 창작**: `SCALE006532`, `76,551원` 등 원본 식별자와 금액은 모두 새로 창작. 대출관리번호 패턴은 `LOAN043224` 형태(LOAN + 6자리 숫자)로 통일. 패턴(1원 하향 차이)만 유지.
2. **Python 스크립트**: 컬럼명과 상수만 변경, 로직 동일 유지.
3. **이미지 없음**: 수식과 코드 블록, Mermaid 다이어그램으로 시각화 충분.
4. **카테고리**: `data` (기존 카테고리 유지, URL 변경 없음).
