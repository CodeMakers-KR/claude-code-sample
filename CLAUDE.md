# 레시피 공유 커뮤니티 설계

## 프로젝트 소개
- 주제: 다수의 회원이 참여해 자신만의 레시피를 공유하고 "맛있어요", "익숙한 맛이에요", "최고에요", "입맛에 안맞아요" 와 같은 피드백과 댓글 달기 등을 이용하는 레시피 공유 커뮤니티.
- 기간: 21일
- 팀: 6명 (풀스택 개발자 6명)
- 예산: 무료

## 기술 환경
### Front-end 환경
- VS Code
- React.js (18+)
- React Redux Toolkit
- React Router Dom
- ECMA Script
- Axios
- Local Storage
- Session Storage
- Vite
- Jest

#### Front-end 프로젝트 구조
- local port: 3000
- components: 컴포넌트와 *.module.css 작성
- pages: Router DOM에 사용되는 컴포넌트들을 작성.
- hooks: Custom Hook을 작성
- utils: 기타 Util을 작성
- public: Image, Favicon
- api: Axios 코드 작성 (환경변수 > API Host url 참조)
- stores: 상태 관리 폴더
  - slices: slice store 작성
  - actions: thunk action 작성
- Component 합성이 필요할 경우 ContextAPI로 합성.

### Backend 환경
- STS Eclipse
- Spring Boot (3+)
- Spring Security
- JWT
- MyBatis
- Oracle Driver
- Spring Boot Validator
- Open JDK 21
- Spring Web Socket
- STOMP
- jUnit
- Swagger

### DB
- OracleXE 21c
  
#### Backend 프로젝트 구조
- local port: 8080
- CORS: localhost:3000
- RESTful API Application
- Main package : kr.codemakers.recipe
- 기본 패키지
  - common
    - exceptions
      - handlers
    - utils
    - config
      - security
        - filters
    - error: Error Controller 작성
- Domain(DB Table)별로 패키지 생성 (소문자)
  - 도메인 패키지 마다 다음 패키지를 생성:
    - web: API Controller 플래스 파일
    - vo
      - request: Request Data 클래스 파일
      - response: Response Data 클래스 파일
    - service: 업무 로직을 작성하기 위한 인터페이스 파일
      - impl: 인터페이스 구현 파일
    - dao: MyBatis 호출을 위한 Mapper 인터페이스 파일
    - utils: 필요할 경우 생성
- src/main/resources 아래에 패키지마다 mapper 폴더 작성
- mapper 파일의 이름은 Dao 인터페이스 이름 뒤에 Mapper.xml을 붙인다.
- DTO 사용 금지

## 인증 방식
- 4시간 동안 유효한 IP 기반 JWT (Access Token, Refresh Token)
  - 다른 PC에서 발급받은 JWT는 사용할 수 없다.
  
## 회원 등급
- 일반 사용자: 일반 권한을 가지고 있는 사용자, 관리자 권한은 없음. 회원가입을 통해 생성
- 관리자: 일반 권한 + 관리자 권한을 가지고 있는 사용자. 시스템 관리 > 관리자 등록을 통해 생성

## 상세 요구 사항
### 필수 기능
#### 회원
- [ ] 회원 가입 - 이메일(pk), 비밀번호, 이름, 닉네임(uk), 프로필 사진(첨부파일)
  - [ ] 가입 완료시 500 포인트 지급
  - [ ] 비밀번호 규칙: 대문자, 소문자, 숫자, 특수문자 모두 1개 이상 포함하여 10자리 이상
  - [ ] 닉네임 규칙: 특수기호 사용 불가, 3자리 이상
  - [ ] 프로필 사진은 서버에 업로드되어 파일 시스템에 저장 (Oracle BLOB 사용 금지)
    - [ ] 첨부파일 이름 난독화
    - [ ] 확장자 사용 불가능
    - [ ] 이미지 파일만 업로드 가능
    - [ ] 프로필 사진 크기가 300px x 300px 보다 클 경우 300px x 300px 사이즈의 썸네일 생성.
    - [ ] 프로필 사진을 브라우저에서 볼 수 있어야 함.
- [ ] 로그인 - 이메일과 비밀번호를 입력해 로그인, 30분 내 5번 실패 시 60분간 로그인 시도 차단
  - [ ] 하루 한번 로그인 할 때마다 50포인트 지급
- [ ] 비밀번호 찾기 - 이메일, 이름, 닉네임을 입력해 일치하는 정보가 있을 경우, 이메일로 비밀번호 변경 링크 전송.

#### 포인트 구매
- [ ] 회원 포인트 구매
  - [ ] 결제 원화 100% 비율로 포인트 지급 + 10% 추가 적립. (예> 1,000원 결제시 1,100포인트 지급)
  - [ ] 1,000, 5,000, 10,000, 50,000 결제
  - [ ] 결제 시 영수증을 메일로 전송
  - [ ] 구매후 7일간 환불 가능. 단, 구매 후 결제분 사용 이력이 있을 경우 환불 불가

#### 레시피
- [ ] 레시피 검색 - 테마별(날씨, 기온, 기분 등등), 주제별(생일, 기념일 등등), 재료별, 높은 만족도, 댓글 수, 키워드 검색 등등
- [ ] 레시피 열람 - 검색한 레시피의 내용을 열람, 레시피 내용 소개 하단에 레시피 요약 정보, 댓글, 피드백이 나오도록 한다.
  - [ ] 내용을 조회할 때마다 10 포인트 차감 (조회 이력 관계없이 볼 때마다 차감)
- [ ] 레시피 구매 - 500 포인트 지급 후 영구 소장. 구매한 레시피는 포인트 차감 발생하지 않음.
  - [ ] 레시피 구매 취소 - 구매 이후 한번도 열람하지 않은 경우 500포인트 반환. 한번이라도 열람한 경우 구매 취소 불가
- [ ] 즐겨찾는 레시피 등록 및 해체 토글 기능.
  - [ ] 즐겨찾기 레시피 등록 시 로그인 사용자에게 100 포인트 차감.
  - [ ] 즐겨 찾기 해제시엔 포인트 변동 없음.
- [ ] 레시피 등록 - 회원만 이용 가능. 레시피 제목, 준비물, 레시피 설명, 이미지, 유튜브 영상, 인스타그램 영상 등등 부가 설명자료 첨부할 수 있음. 위지윅 에디터 사용.
  - [ ] 등록할 때마다 100 포인트를 회원에게 지급
- [ ] 레시피 수정 - 작성자만 이용 가능.
  - [ ] 레시피 수정할 때마다 80 포인트 차감
- [ ] 레시피 삭제 - 작성자만 이용 가능. 댓글이나 피드백이 있을 경우 삭제 불가능.
  - [ ] 레시피 삭제할 때마다 100 포인트 차감
  - [ ] 댓글이나 피드백이 있는 레시피는 200포인트 차감시 삭제 가능. - 댓글 및 피드백을 등록한 다른 사용자는 포인트 차감 없음.
- [ ] 레시피 댓글 등록 - 회원만 이용 가능. 대댓글 등록 가능. 레시피 내용 인용할 수 있다. 위지윅 에디터 사용. 이미지, 유튜브 링크, 인스타그램 링크 등등 첨부 가능.
- [ ] 레시피 댓글 수정 - 댓글 작성자만 이용 가능. 대댓글이 있는 댓글은 수정 불가능.
- [ ] 레시피 댓글 삭제 - 댓글 작성자만 이용 가능. 대댓글이 있는 댓글은 삭제 불가능.
- [ ] 레시피 댓글 조회 - 댓글들이 조회된다. (작성자, 작성일, 수정일, 댓글 내용) 삭제된 댓글은 "삭제된 댓글입니다" 로 표현.
- [ ] 레시피 피드백 등록 - 회원만 이용가능. "맛있어요", "익숙한 맛이에요", "최고에요", "입맛에 안맞아요", "성의 없어요" 중 하나를 선택. 중복 선택은 불가능.
  - [ ] "맛있어요" 피드백을 받은 레시피 등록자에게 30포인트 지급
  - [ ] "익숙한 맛이에요" 피드백을 받은 레시피 등록자에게 20포인트 지급
  - [ ] "최고에요" 피드백을 받은 레시피 등록자에게 50포인트 지급
  - [ ] "입맛에 안맞아요" 피드백을 받은 레시피 등록자에게 5포인트 지급
  - [ ] "성의 없어요" 피드백을 받은 레시피 등록자에게 20포인트 차감

#### 실시간 채팅, 쪽지
- [ ] 실시간 채팅 - 회원간 실시간 채팅이 가능. 그룹 채팅, 1:1 채팅, 비밀대화.
      채팅이 도착할 경우 알림이 나타난다.
- [ ] 실시간 쪽지 - 회원간 쪽지를 주고 받을 수 있다.
      쪽지가 도착할 경우 알림이 나타난다.

#### 마이페이지
- [ ] 내가 등록한 레시피
- [ ] 즐겨찾는 레시피
- [ ] 내가 구매한 레시피
- [ ] 내가 본 레시피
- [ ] 내가 등록한 댓글
- [ ] 내가 남긴 피드백
- [ ] 결제이력
- [ ] 포인트 적립 및 사용 이력
- [ ] 받은 쪽지
- [ ] 보낸 쪽지
- [ ] 채팅 기록
- [ ] 내 정보 수정 - 이메일을 제외한 나머지 정보를 수정할 수 있음.
  -  [ ] 비밀번호 변경시 이전 비밀번호는 사용하지 못해야 함.
  -  [ ] 프로필 사진 변경시, 이전의 프로필 사진은 서버에서 삭제.
- [ ] 탈퇴

#### 시스템 관리 (관리자만 사용 가능)
- [ ] 환경 변수 - 커뮤니티에서 사용하게되는 다양한 환경 변수를 관리 (예> API Host url, 레시피 검색시 테마 종류, 주제 종류, 재료 종류 등등)
- [ ] 사용자 관리 - 개인정보는 마스킹 처리
- [ ] 관리자 관리 - 기존 관리자 관리, 새로운 관리자 생성, 삭제 등

### 대시보드
- [ ] 마이페이지의 데이터를 기반으로 레시피 추천
- [ ] 마이페이지의 데이터를 기반으로 나의 취향과 전체 회원 취향과의 비교
- [ ] 내가 등록한 댓글이나 레시피에 새로 등록된 댓글 알림
- [ ] 내가 등록한 댓글이나 레시피에 새로 등록된 피드백 알림
- [ ] 방금 등록된 레시피
- [ ] 인플루언서(즐겨찾기 횟수 + 구매횟수 + 긍정적 피드백 수 + 열람횟수 가 가장 높은)의 레시피 소개
- [ ] 내가 즐겨찾는 레시피

## 개발 패턴
### Backend
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

#### TDD 테스트 커버리지 기준
- Class: 100%
- Line: 100%
- Condition: 100%

### Front-End
- 반응형 웹 (Mobile 기준, Desktop 확장)
- TDD
  - 단위 기능 개발전 JEST 먼저 작성
  - Strict Mode에서 중복 호출 제거
  - 과도한 Re-Rendering 발생하지 않도록 함.
- Props Type 지정
  - Component Props 지정하여 사용.
- Document 작성 대상
  - Component
  - Hooks
  - Page
  - Utils
  - Reducer
  - Thunks
  - Axios

## Sanitizing
- 서버에서 SQL Injection, Command Injection, XSS, CSRF 사용할 수 없도록 요청 파라미터 검증 및 수정
- 첨부파일 Mime-Type White List 관리.
  - 이미지파일, 비디오파일, 오디오파일 만 등록 가능
  - 압축 파일은 내부 파일의 Mime-Type 모두 확인.
  - Apache Tika, jMimeMagic 중 비교하여 빠르고 정확하며 가벼운 라이브러리 사용

## ERD
- 상세 요구사항을 기반으로한 ERD 생성
- 테이블명과 컬럼명은 자음 위주의 3~4자리 약어로 생성
- VARCHAR2 Type은 CHAR 길이 사용 (예> VARCHAR2(10) ==> VARCHAR2(10 CHAR))
- 테이블, 컬럼에 COMMENT 작성
- 테이블명과 컬럼명에 Oracle 예약어 사용 금지.
- 시퀀스
  - 네이밍 규칙: SEQ_{테이블명}_PK
  - Cycle: YES
  - 최소값: 1
  - 최대값: 999999
- 식별 관계는 사용하지 않음: 비 식별 관계만 사용.
- PK, UK 사용
- FK: 논리적 모델에서만 사용. 물리적 모델에서는 사용 하지 않음.

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
