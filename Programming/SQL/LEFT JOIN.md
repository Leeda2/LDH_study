## SQL `LEFT JOIN` 조건 위치에 따른 데이터 누락 

### 주의해야 할 SQL
``` sql
SELECT
    A.USER\_ID,
    A.USER\_NAME,
    COUNT(C.ORDER\_ID) AS ORDER\_CNT
FROM TB\_USER A
LEFT JOIN TB\_ORDER C
       ON A.USER\_ID = C.USER\_ID
WHERE A.USE\_YN = 'Y'
  AND C.ORDER\_DATE >= TO\_DATE('20260901', 'YYYYMMDD')
GROUP BY
    A.USER\_ID,
    A.USER\_NAME;
```
=>`TB\_ORDER`를 `LEFT JOIN`했지만, `WHERE` 절에서 다음 조건을 사용

``` sql
C.ORDER\_DATE >= TO\_DATE('20260901', 'YYYYMMDD')
```
=> 주문 데이터가 없는 사용자는 `LEFT JOIN` 결과에서 `C.ORDER\_DATE`가
`NULL`이 된다.
따라서 `WHERE` 조건을 만족하지 못하고 해당 사용자 행이 최종 결과에서
제거된다.
즉, LEFT JOIN을 사용했음에도 결과적으로 INNER JOIN과 유사하게 동작하여
기준 테이블의 데이터가 누락될 수 있다.

### 누락없는 SQL
``` sql
SELECT
    A.USER\_ID,
    A.USER\_NAME,
    COUNT(C.ORDER\_ID) AS ORDER\_CNT
FROM TB\_USER A
LEFT JOIN TB\_ORDER C
       ON A.USER\_ID = C.USER\_ID
      AND C.ORDER\_DATE >= TO\_DATE('20260901', 'YYYYMMDD')
WHERE A.USE\_YN = 'Y'
GROUP BY
    A.USER\_ID,
    A.USER\_NAME;
```
=> 주문 날짜 조건을 `WHERE`가 아니라 `LEFT JOIN`의 `ON` 절에 작성한다.

### 설명
WHERE에 JOIN 대상 테이블 조건을 작성
``` sql
LEFT JOIN TB\_ORDER C
       ON A.USER\_ID = C.USER\_ID
WHERE C.ORDER\_DATE >= ...
```
=> `C` 데이터가 없는 경우 `NULL`이 되어 `WHERE`에서 탈락한다.
→ 기준 테이블(A)의 데이터가 누락될 수 있음
ON에 JOIN 대상 테이블 조건을 작성

``` sql
LEFT JOIN TB\_ORDER C
       ON A.USER\_ID = C.USER\_ID
      AND C.ORDER\_DATE >= ...
```
=> A 테이블 데이터는 유지하면서 조건에 맞는 C 데이터만 연결한다.
  → LEFT JOIN의 목적을 유지할 수 있음
  → `LEFT JOIN` 대상 테이블의 조건을 `WHERE`에 작성하면, 해당 테이블의
  → 값이 `NULL`인 행이 제거되어 사실상 `INNER JOIN`처럼 동작할 수 있다.

