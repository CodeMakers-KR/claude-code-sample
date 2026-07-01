# Coding Guide
## Backend
- TDD
  - 단위 기능 개발 전 JUNIT 먼저 작성
  - Red-Green-Refactor 패턴
  - Given-When-Then-Verify 패턴
- SOLID 설계
- Method 는 20 line 이하.
- Cache
  - 자주 호출되거나 참조되는 데이터 캐싱
- 인스턴스 멤버를 호출할 때 "this." 반드시 사용.
- Lombok 사용 금지
- Document 작성 대상
  - RESTful API Spec (Swagger)
  - Class
  - Method
  - Field
  - Parameters & Arguments

## TDD 테스트 커버리지 기준
- Class: 100%
- Line: 100%
- Condition: 100%

## Sanitizing
- 서버에서 SQL Injection, Command Injection, XSS, CSRF 사용할 수 없도록 요청 파라미터 검증 및 수정
- 첨부파일 Mime-Type White List 관리.
  - 이미지파일, 비디오파일, 오디오파일 만 등록 가능
  - 압축 파일은 내부 파일의 Mime-Type 모두 확인.
  - Apache Tika, jMimeMagic 중 비교하여 빠르고 정확하며 가벼운 라이브러리 사용

## MyBatis 규칙
### SELECT
- 단일 테이블내에서 조회시 resultType 사용
- 여러 테이블에서 복합 컬럼 조회시 resultMap 사용
  - association, collection 적극 사용 권장.
  - VO는 `has a` 규칙으로 구성.

### Query 작성 규칙
- PK값 규칙: {테이블명 2자리 약어}-{YYYYMMDD}-{LPAD(시퀀스값, 6, '0')}
- Pagination
  - HEADER:
    ```
      SELECT *
        FROM (SELECT ROWNUM AS RNUM
                   , T.*
                FROM (
    ```
                
  - FOOTER:
    ```
                     ) T
               WHERE ROWNUM < (#{페이지번호} + 1) * #{출력 행 수})
       WHERE RNUM >= #{페이지번호} * #{출력 행 수} + 1
    ```
- 비교 연산자 ( < ) 사용시 `<![CDATA[]]>` 사용
  ```
  WHERE NUM <![CDATA[ < ]]> 10
  ```
- SELECT Query 작성 규칙 (Scala, Inline View, Sub query 동일)
  - 컬럼에 새 이름 부여시 "AS" 생략하지 않음.
  ```
    SELECT T1.COLUMN1
         , T2.COLUMN2 AS NEW_COL_NAME
         , T3.COLUMN3
         , CASE
             WHEN CONDITION THEN
               VALUE
             ELSE
               VALUE
           END AS COLUMN_ALIAS
      FROM TABLE1 T1
     INNER JOIN TABLE2 T2
        ON T1.PK = T2.FK
      LEFT OUTER JOIN TABLE3 T3
        ON T2.PK = T3.FK
     INNER JOIN (SELECT COLUMN1
                      , COLUMN2
                   FROM TABLE4
                  WHERE CONDITION) T4
        ON T3.PK = T4.FK
     WHERE CONDITION
       AND CONDITION
     GROUP BY COLUMN1
         , COLUMN2
     ORDER BY COLUMN1 ASC
         , COLUMN2 DESC
  ```
- INSERT Query 작성 규칙
  ```
    INSERT INTO TABLE
     (COLUMN1
    , COLUMN2)
    VALUES
     (VAL1
    , VAL2)
  ```
- UPDATE Query 작성 규칙
  ```
    UPDATE TABLE
       SET COLUMN = VALUE
         , COLUMN2 = VALUE
     WHERE CONDITION
  ```
- DELETE Query 작성 규칙
  ```
    DELETE
      FROM TABLE
     WHERE CONDITION
  ```
