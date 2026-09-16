# UNION ALL 실무 참고 노트

> **같은 형태의 조회 결과를 아래로 합친 뒤, 하나의 목록처럼 다시 가공한다.**
>
> 기본 문법 · JOIN 비교 · 인라인 뷰 · 통합 조회 · 금액 집계 · 최신 이력

## 빠르게 찾아보기

| 원하는 작업 | 참고 항목 |
|---|---|
| 문법만 확인 | 1. 기본 문법 |
| JOIN과 차이 확인 | 2. JOIN / UNION / UNION ALL |
| 합친 결과를 필터링·집계 | 3. 서브쿼리로 다시 가공 |
| 과태료·세금 통합 화면 | 4. 업무별 내역 통합 조회 |
| 부과액·납부액·미납액 계산 | 5. 서로 다른 거래를 합산 |
| 현재·과거 데이터 중 최신 한 건 | 6. 최신 이력 조회 |
| 오류나 잘못된 합계 점검 | 7. 실무 체크리스트 |

예제는 **Oracle 기준의 설명용 SQL**이다. 테이블·컬럼은 가상의 이름이며 실제 시스템에 맞게 변경해야 한다. `:payer_id` 등은 바인드 변수다. 예제는 실제 업무 DB에서 실행 검증한 쿼리가 아니다.

---

## 1. 기본 문법

```sql
SELECT USER_ID, USER_NAME, AGE
FROM TB_MEMBER

UNION ALL

SELECT USER_ID, USER_NAME, AGE
FROM TB_WITHDRAWN_MEMBER
ORDER BY USER_ID;
```

현재 회원과 탈퇴 회원을 하나의 목록처럼 조회한다. 실제 테이블을 변경하거나 합치는 작업은 아니다.

### 합칠 때 지켜야 할 규칙

- 각 SELECT의 **컬럼 개수가 같아야 한다.**
- **같은 순번의 컬럼끼리 자료형이 호환**되어야 한다. Oracle에서는 숫자와 문자열처럼 자료형 그룹이 다르면 명시적으로 변환해야 한다.
- 컬럼명은 달라도 된다. 결과 컬럼명은 **첫 번째 SELECT**를 기준으로 정해진다.
- UNION ALL은 **중복 행을 그대로 유지**한다.
- 최종 정렬은 바깥쪽 또는 전체 쿼리 마지막의 `ORDER BY`로 지정한다. 작성한 SELECT 순서대로 출력된다고 보장되지 않는다.

```sql
-- 컬럼명이 달라도 의미와 자료형을 맞추면 합칠 수 있다.
SELECT USER_NAME AS NAME, AGE
FROM TB_MEMBER
UNION ALL
SELECT CUSTOMER_NAME AS NAME, CUSTOMER_AGE AS AGE
FROM TB_CUSTOMER;
```

## 2. JOIN / UNION / UNION ALL

| 구분 | 역할 | 중복 처리 | 대표 사용처 |
|---|---|---|---|
| JOIN | 조건으로 연결되는 행의 컬럼을 옆으로 결합 | 연결 조건과 건수에 따라 행이 늘어날 수 있음 | 회원 정보 + 부서명 |
| UNION | 조회 결과를 아래로 결합 | 선택한 모든 컬럼 값이 같은 행 제거 | 중복 없는 통합 목록 |
| UNION ALL | 조회 결과를 아래로 결합 | 모두 유지 | 거래 내역·통합 통계 |

**JOIN 예시:** A의 이름·나이·국가와 B의 키·머리색을 같은 사람 기준으로 연결한다.

| 이름 | 나이 | 국가 | 키 | 머리색 |
|---|---:|---|---:|---|
| 다혜 | 33 | 한국 | 168 | 검정 |

**UNION ALL 예시:** A의 회원 목록 아래에 B의 회원 목록을 이어 붙인다.

| 이름 | 나이 | 국가 |
|---|---:|---|
| 다혜 | 33 | 한국 |
| 민수 | 30 | 일본 |

> **주의:** UNION은 “같은 사람”을 판단하지 않는다. ID가 같아도 상태나 날짜 등 선택한 컬럼 중 하나가 다르면 두 행이 모두 남는다.

## 3. 서브쿼리로 다시 가공

FROM 안에 넣는 서브쿼리를 **인라인 뷰**라고 한다. 합친 결과에 `U`라는 별칭을 붙여 하나의 테이블처럼 조회할 수 있다.

### 3-1. 합친 뒤 조건 검색

```sql
SELECT U.USER_ID, U.USER_NAME, U.AGE
FROM (
    SELECT USER_ID, USER_NAME, AGE FROM TB_MEMBER
    UNION ALL
    SELECT USER_ID, USER_NAME, AGE FROM TB_WITHDRAWN_MEMBER
) U
WHERE U.AGE >= 30
ORDER BY U.AGE, U.USER_ID;
```

### 3-2. 합친 뒤 나이별 인원수 집계

```sql
SELECT U.AGE, COUNT(*) AS PERSON_COUNT
FROM (
    SELECT USER_ID, AGE FROM TB_MEMBER
    UNION ALL
    SELECT USER_ID, AGE FROM TB_WITHDRAWN_MEMBER
) U
GROUP BY U.AGE
ORDER BY U.AGE;
```

같은 사람이 양쪽 테이블에 있으면 두 번 집계된다. 사람 수가 목적이라면 공통 식별자와 중복 처리 기준을 먼저 정해야 한다.

## 4. 업무별 내역 통합 조회

**상황:** 과태료와 세금이 서로 다른 테이블에 있지만 납부자 화면에서는 함께 표시해야 한다.

```sql
SELECT U.WORK_TYPE,
       U.PAYER_ID,
       U.AMOUNT,
       U.IMPOSE_DATE
FROM (
    SELECT 'PARKING' AS WORK_TYPE,
           PAYER_ID,
           PENALTY_AMOUNT AS AMOUNT,
           IMPOSE_DATE
    FROM TB_PARKING_PENALTY
    WHERE PAYER_ID = :payer_id

    UNION ALL

    SELECT 'TAX' AS WORK_TYPE,
           TAXPAYER_ID AS PAYER_ID,
           TAX_AMOUNT AS AMOUNT,
           IMPOSE_DATE
    FROM TB_TAX_IMPOSITION
    WHERE TAXPAYER_ID = :payer_id
) U
ORDER BY U.IMPOSE_DATE DESC, U.WORK_TYPE;
```

| WORK_TYPE | PAYER_ID | AMOUNT | IMPOSE_DATE |
|---|---:|---:|---|
| PARKING | 101 | 40,000 | 2026-09-15 |
| TAX | 101 | 60,000 | 2026-09-10 |

**핵심 포인트**

- 서로 다른 컬럼명을 공통 출력 컬럼으로 맞춘다.
- 업무 구분값을 추가해 원본 출처를 표시한다.
- 두 테이블의 납부자 ID가 같은 식별 체계를 사용한다는 전제가 필요하다.
- 각 분기에 조건을 명시하면 조회 범위가 분명해진다. 실제 성능은 인덱스와 실행계획으로 확인한다.

## 5. 서로 다른 거래를 합산

**상황:** 부과 테이블과 납부 테이블을 합쳐 납부자별 총부과액·총납부액·미납액을 계산한다.

```sql
SELECT U.PAYER_ID,
       SUM(U.IMPOSE_AMOUNT) AS TOTAL_IMPOSE_AMOUNT,
       SUM(U.PAID_AMOUNT) AS TOTAL_PAID_AMOUNT,
       SUM(U.IMPOSE_AMOUNT) - SUM(U.PAID_AMOUNT) AS UNPAID_AMOUNT
FROM (
    SELECT PAYER_ID,
           NVL(AMOUNT, 0) AS IMPOSE_AMOUNT,
           0 AS PAID_AMOUNT
    FROM TB_IMPOSITION

    UNION ALL

    SELECT PAYER_ID,
           0 AS IMPOSE_AMOUNT,
           NVL(AMOUNT, 0) AS PAID_AMOUNT
    FROM TB_PAYMENT
) U
GROUP BY U.PAYER_ID
ORDER BY U.PAYER_ID;
```

| PAYER_ID | TOTAL_IMPOSE_AMOUNT | TOTAL_PAID_AMOUNT | UNPAID_AMOUNT |
|---|---:|---:|---:|
| 101 | 100,000 | 70,000 | 30,000 |
| 102 | 50,000 | 50,000 | 0 |

### 왜 바로 JOIN하지 않을까?

한 납부자의 부과가 2건, 납부가 3건이면 납부자 ID만으로 JOIN할 때 **2 × 3 = 6행**이 만들어질 수 있다. 그 상태에서 SUM하면 금액이 반복 합산된다.

해결 방법은 두 가지다.

1. 위 예시처럼 UNION ALL로 합친 후 집계한다.
2. 각 테이블을 납부자별로 먼저 집계한 뒤 JOIN한다.

### 실제 업무에 적용할 때

- 건별 미납액은 `IMPOSE_ID` 같은 부과번호까지 가져와 GROUP BY에 포함한다.
- 납부를 해당 부과 건에 정확히 연결해야 한다. 단순히 부과일과 납부일에 같은 기간 조건을 넣는 것만으로는 범위가 맞지 않을 수 있다.
- 취소, 감액, 환급, 선납, 초과납부의 처리 기준을 반영한다.
- 예제의 `NVL(AMOUNT, 0)`는 NULL을 0으로 간주한다. 금액 누락이 오류인 업무에서는 별도 점검이 필요하다.

## 6. 최신 이력 조회

**상황:** 현재·과거 테이블에 나뉜 상태 이력을 합쳐 대상별 최신 상태 한 건을 조회한다.

```sql
WITH ALL_HISTORY AS (
    SELECT TARGET_ID, STATUS, CHANGED_AT, HISTORY_ID,
           1 AS SOURCE_PRIORITY
    FROM TB_CURRENT_HISTORY

    UNION ALL

    SELECT TARGET_ID, STATUS, CHANGED_AT, HISTORY_ID,
           2 AS SOURCE_PRIORITY
    FROM TB_ARCHIVE_HISTORY
),
RANKED_HISTORY AS (
    SELECT TARGET_ID,
           STATUS,
           CHANGED_AT,
           ROW_NUMBER() OVER (
               PARTITION BY TARGET_ID
               ORDER BY CHANGED_AT DESC NULLS LAST,
                        SOURCE_PRIORITY ASC,
                        HISTORY_ID DESC
           ) AS RN
    FROM ALL_HISTORY
)
SELECT TARGET_ID, STATUS, CHANGED_AT
FROM RANKED_HISTORY
WHERE RN = 1
ORDER BY TARGET_ID;
```

| 구문 | 역할 |
|---|---|
| WITH ALL_HISTORY AS (...) | 합친 결과에 이름 부여 |
| PARTITION BY TARGET_ID | 대상별로 순위 계산 |
| CHANGED_AT DESC NULLS LAST | 최근 변경일시 우선, 날짜가 없는 행은 뒤로 |
| SOURCE_PRIORITY ASC | 일시가 같으면 현재 테이블 우선 |
| HISTORY_ID DESC | 같은 출처·일시라면 큰 이력 ID 우선 |
| WHERE RN = 1 | 대상별 첫 행만 선택 |

`HISTORY_ID`는 각 원본 테이블 안에서 고유하다고 가정한다. “동일 일시에는 현재 테이블 우선”은 예제의 업무 규칙이므로 실제 정책에 맞게 수정한다.

## 7. 실무 체크리스트

| 점검 항목 | 확인할 내용 |
|---|---|
| 컬럼 수 | 모든 SELECT의 개수가 같은가? |
| 컬럼 순서 | 이름 자리에는 이름, 금액 자리에는 금액이 오는가? |
| 자료형 | 같은 순번의 자료형이 호환되는가? |
| 데이터 의미 | 단위·통화·코드·ID 체계가 같은가? |
| 중복 | 양쪽에 같은 거래가 있어도 합산해도 되는가? |
| 집계 단위 | 사람별인지, 부과 건별인지 명확한가? |
| 조회 범위 | 각 분기가 동일한 업무 범위를 대상으로 하는가? |
| 정렬 | 최종 ORDER BY를 지정했는가? |
| 성능 | 필요한 컬럼·조건만 사용하고 실행계획을 확인했는가? |

### 한쪽에 없는 컬럼은 어떻게 맞출까?

```sql
SELECT USER_ID, USER_NAME, EMAIL
FROM TB_MEMBER
UNION ALL
SELECT USER_ID, USER_NAME,
       CAST(NULL AS VARCHAR2(200)) AS EMAIL
FROM TB_OLD_MEMBER;
```

없는 정보는 자료형을 맞춘 NULL로 채울 수 있다. 위 예시는 EMAIL이 VARCHAR2 계열이라고 가정한다. 금액 계산처럼 0이 올바른 의미일 때만 0으로 채운다.

### 성능 기억하기

UNION ALL은 UNION의 중복 제거 작업이 없어 일반적으로 그 비용을 줄일 수 있다. 하지만 **항상 더 빠르다고 단정할 수 없으며**, 중복 제거가 필요한 업무라면 정확한 결과가 우선이다.

---

## 복사용 기본 틀

```sql
SELECT U.CATEGORY,
       SUM(U.AMOUNT) AS TOTAL_AMOUNT
FROM (
    SELECT CATEGORY, AMOUNT
    FROM TABLE_A
    WHERE /* A 조건 */ 1 = 1

    UNION ALL

    SELECT CATEGORY, AMOUNT
    FROM TABLE_B
    WHERE /* B 조건 */ 1 = 1
) U
GROUP BY U.CATEGORY
ORDER BY U.CATEGORY;
```

> **기억할 한 문장:** UNION ALL로 아래로 합치고, 바깥 SELECT에서 검색·집계·순위 계산을 한다.
