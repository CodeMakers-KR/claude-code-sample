<!-- @format -->

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

- Tod-man: TODO Management Application

## 프로젝트 개요

- Tod-man은 Todo 를 단계별로 관리할 수 있는 애플리케이션이다.

## 프로젝트 구조

- tod-man
  - client # front end
  - server # back end

### Server

- 요구사항기반 domain 단위로 구성
- server
  - /src/main/java
    - com.todo
      - [domain]
        - web # RestController
        - service # Service Interface & Implements Class # [Domain]Service, [Domain]ServiceImpl
        - dao # @Mapper 기반 인터페이스 # [Domain]Mapper
        - vo
          - request # 요청 Data VO
          - response # 응답 Data VO
        - utils # 필요시 생성
      - common
        - exceptions # 예외 클래스 (TodManException.java)
          - handler # 글로벌 예외 핸들러 (RestControllerAdvice 타입)
        - errors # Error API Endpoint
        - utils # 공통 유틸리티 작성
        - vo # 공통 VO 작성
        - config # 공통 설정 작성
  - /src/main/resources
    - application.yml
    - com/todo/[domain]/dao/mapper/[Domain]Mapper.xml # MyBatis Mapper 파일
  - /src/test/java # junit
    - com.todo
      - [domain]
        - web # api controller test
        - service # service test
        - dao # dao test

### Client

- client
  - public # 파비콘, 공통 이미지
  - src
    - components # 공통 컴포넌트들을 성격을 기준으로 모아서 컴포넌트로 관리
    - routers # 라우팅 설정 작성
    - pages # 페이지별로 컴포넌트 작성
    - hooks # 커스텀 훅 작성
    - utils # 필요 유틸리티 작성
    - store # Toolkit 스토어
      - slices # 슬라이스 스토어 작성
      - actions # Thunks 작성

## 기술 스택

### Server

- Language: Java 21
- Build Tool: Maven
- Packaging: Jar
- Framework: Spring Boot 4, Spring Security, MyBatis, Spring Validator
- Security: jsoup (입력 HTML 정화, 저장형 XSS 방어)
- Test: jUnit6
- Database: Oracle (Docker)

### Client

- Language: ECMAScript (Strict Mode)
- Build Tool: Vite
- Library: React 19+, React Router Dom
- State: state, Redux Toolkit
- Styling: Tailwind CSS, \*.module.css
- Web Client Library: window.fetch
- Drag & Drop: @dnd-kit/core (^6.3.1), @dnd-kit/sortable (^10.0.0)
- Test: Vitest, React Testing Library, jsdom

## 공통 정의

### status 값

| 값         | 칼럼        |
| ---------- | ----------- |
| todo       | todo        |
| inProgress | in progress |
| done       | done        |

- API 요청/응답, DB 저장값 모두 위 값을 그대로 사용 (대소문자 구분)

### priority 값

- HIGH, MEDIUM, LOW (기본값 MEDIUM)

### 날짜 직렬화 형식

| 타입          | 형식                               | 예시                |
| ------------- | ---------------------------------- | ------------------- |
| LocalDate     | `yyyy-MM-dd`                       | 2026-10-17          |
| LocalDateTime | `yyyy-MM-dd'T'HH:mm:ss` (ISO-8601) | 2026-10-17T11:30:00 |

### 날짜 검증 기준 (당일 포함)

- startDate: 오늘과 같거나 이후 (startDate >= 오늘)
- dueDate: startDate와 같거나 이후 (dueDate >= startDate)

## 기능적 요구사항 (Functional Requirements)

### FR-001: TODO 생성

- 설명: 사용자가 새로운 TODO를 생성한다.
- API Endpoint: POST /api/v1/todos

#### 입력 필드:

| 필드        | 타입      | 필수 | 제약 조건                          | 기본값 |
| ----------- | --------- | ---- | ---------------------------------- | ------ |
| title       | String    | Y    | 1~200자, 공백만(Empty String) 불가 | -      |
| description | String    | N    | 최대 1,000자                       | null   |
| priority    | String    | N    | HIGH, MEDIUM, LOW                  | MEDIUM |
| startDate   | LocalDate | N    | -                                  | null   |
| dueDate     | LocalDate | N    | -                                  | null   |

#### 성공 응답 필드:

| 필드          | 타입          | 값                         |
| ------------- | ------------- | -------------------------- |
| status        | int           | 201                        |
| data          | Object        | -                          |
| - id          | String        | 새 TODO id                 |
| - title       | String        | 새 TODO title              |
| - description | String        | 새 TODO description        |
| - priority    | String        | 새 TODO priority           |
| - status      | String        | 새 TODO status             |
| - position    | int           | 새 TODO position           |
| - startDate   | LocalDate     | 새 TODO startDate          |
| - startedAt   | LocalDateTime | 새 TODO startedAt          |
| - dueDate     | LocalDate     | 새 TODO dueDate            |
| - createdAt   | LocalDateTime | 새 TODO createdAt          |
| - updatedAt   | LocalDateTime | 새 TODO updatedAt          |
| - completedAt | LocalDateTime | 새 TODO completedAt (null) |

#### 실패 응답 필드:

| 필드      | 타입   | 값                                                                        |
| --------- | ------ | ------------------------------------------------------------------------- |
| error     | Object | -                                                                         |
| - status  | int    | 검증 실패: 400, System Error: 500                                         |
| - message | String | 에러 내용                                                                 |
| - errors  | List   | 입력 검증 실패 시에만 포함. 필드별 { field, message } 전체 (API_SPEC 1.2) |

#### 처리 규칙

- 생성시 항상 todo 가장 위에 배치
- 생성일(createdAt), 수정일(updatedAt) 자동 설정

#### 검증 에러 메시지

| 조건                               | 메시지                                              |
| ---------------------------------- | --------------------------------------------------- |
| 제목 누락                          | "제목을 입력해주세요"                               |
| 제목200자 초과                     | "제목은 200자 이내로 입력해주세요"                  |
| 제목 공백만 입력                   | "제목을 입력해주세요"                               |
| 설명 1000자 초과                   | "설명은 1000자 이내로 입력해주세요"                 |
| 잘못된 우선순위 값                 | "우선순위는 HIGH, MEDIUM, LOW 중 하나를 선택하세요" |
| 시작 예정일이 과거                 | "시작 예정일은 오늘 이후 날짜를 선택하세요"         |
| 완료 예정일이 시작 예정일보다 과거 | "완료 예정일은 시작 예정일 이후 날짜를 선택하세요"  |
| 완료 예정일만 선택                 | "시작 예정일을 먼저 선택해주세요"                   |

### FR-002: TODO 목록 조회

- 설명: Working Group에 노출할 TODO 목록 조회
- API Endpoint: GET /api/v1/todos

#### 성공 응답 필드:

| 필드            | 타입              | 값                      |
| --------------- | ----------------- | ----------------------- |
| status          | int               | 200                     |
| data            | Map<String, List> | -                       |
| - todo          | List              | -                       |
| - - id          | String            | TODO id                 |
| - - title       | String            | TODO title              |
| - - description | String            | TODO description        |
| - - priority    | String            | TODO priority           |
| - - status      | String            | TODO status             |
| - - position    | int               | TODO position           |
| - - startDate   | LocalDate         | TODO startDate          |
| - - startedAt   | LocalDateTime     | TODO startedAt          |
| - - dueDate     | LocalDate         | TODO dueDate            |
| - - createdAt   | LocalDateTime     | TODO createdAt          |
| - - updatedAt   | LocalDateTime     | TODO updatedAt          |
| - - completedAt | LocalDateTime     | TODO completedAt (null) |
| - inProgress    | List              | -                       |
| - - id          | String            | TODO id                 |
| - - title       | String            | TODO title              |
| - - description | String            | TODO description        |
| - - priority    | String            | TODO priority           |
| - - status      | String            | TODO status             |
| - - position    | int               | TODO position           |
| - - startDate   | LocalDate         | TODO startDate          |
| - - startedAt   | LocalDateTime     | TODO startedAt          |
| - - dueDate     | LocalDate         | TODO dueDate            |
| - - createdAt   | LocalDateTime     | TODO createdAt          |
| - - updatedAt   | LocalDateTime     | TODO updatedAt          |
| - - completedAt | LocalDateTime     | TODO completedAt (null) |
| - done          | List              | -                       |
| - - id          | String            | TODO id                 |
| - - title       | String            | TODO title              |
| - - description | String            | TODO description        |
| - - priority    | String            | TODO priority           |
| - - status      | String            | TODO status             |
| - - position    | int               | TODO position           |
| - - startDate   | LocalDate         | TODO startDate          |
| - - startedAt   | LocalDateTime     | TODO startedAt          |
| - - dueDate     | LocalDate         | TODO dueDate            |
| - - createdAt   | LocalDateTime     | TODO createdAt          |
| - - updatedAt   | LocalDateTime     | TODO updatedAt          |
| - - completedAt | LocalDateTime     | TODO completedAt        |

#### 실패 응답 필드:

| 필드      | 타입   | 값                |
| --------- | ------ | ----------------- |
| error     | Object | -                 |
| - status  | int    | System Error: 500 |
| - message | String | 에러 내용         |

#### 처리 규칙

- 각 칼럼은 position 오름차순 정렬
- done 칼럼은 completedAt 기준 24시간 이내 TODO만 조회

### FR-003: TODO 상세 조회

- 설명: 특정 TODO의 전체 정보를 조회한다.
- API Endpoint: GET /api/v1/todos/{id}

#### 성공 응답 필드:

| 필드          | 타입          | 값               |
| ------------- | ------------- | ---------------- |
| status        | int           | 200              |
| data          | Object        | -                |
| - id          | String        | TODO id          |
| - title       | String        | TODO title       |
| - description | String        | TODO description |
| - priority    | String        | TODO priority    |
| - status      | String        | TODO status      |
| - position    | int           | TODO position    |
| - startDate   | LocalDate     | TODO startDate   |
| - startedAt   | LocalDateTime | TODO startedAt   |
| - dueDate     | LocalDate     | TODO dueDate     |
| - createdAt   | LocalDateTime | TODO createdAt   |
| - updatedAt   | LocalDateTime | TODO updatedAt   |
| - completedAt | LocalDateTime | TODO completedAt |

#### 실패 응답 필드:

| 필드      | 타입   | 값                                         |
| --------- | ------ | ------------------------------------------ |
| error     | Object | -                                          |
| - status  | int    | 존재하지 않는 TODO: 404, System Error: 500 |
| - message | String | 에러 내용                                  |

### FR-004: TODO 수정

- 설명: TODO의 제목, 설명, 우선순위, 시작 예정일, 완료 예정일을 수정한다.
- API Endpoint: PUT /api/v1/todos/{id}

#### 입력 필드:

| 필드        | 타입      | 필수 | 제약 조건                          | 기본값 |
| ----------- | --------- | ---- | ---------------------------------- | ------ |
| title       | String    | Y    | 1~200자, 공백만(Empty String) 불가 | -      |
| description | String    | N    | 최대 1,000자                       | null   |
| priority    | String    | N    | HIGH, MEDIUM, LOW                  | MEDIUM |
| startDate   | LocalDate | N    | -                                  | null   |
| dueDate     | LocalDate | N    | -                                  | null   |

#### 성공 응답 필드:

| 필드          | 타입          | 값                      |
| ------------- | ------------- | ----------------------- |
| status        | int           | 200                     |
| data          | Object        | -                       |
| - id          | String        | 수정한 TODO id          |
| - title       | String        | 수정한 TODO title       |
| - description | String        | 수정한 TODO description |
| - priority    | String        | 수정한 TODO priority    |
| - status      | String        | 수정한 TODO status      |
| - position    | int           | 수정한 TODO position    |
| - startDate   | LocalDate     | 수정한 TODO startDate   |
| - startedAt   | LocalDateTime | 수정한 TODO startedAt   |
| - dueDate     | LocalDate     | 수정한 TODO dueDate     |
| - createdAt   | LocalDateTime | 수정한 TODO createdAt   |
| - updatedAt   | LocalDateTime | 수정한 TODO updatedAt   |
| - completedAt | LocalDateTime | 수정한 TODO completedAt |

#### 실패 응답 필드:

| 필드      | 타입   | 값                                                                        |
| --------- | ------ | ------------------------------------------------------------------------- |
| error     | Object | -                                                                         |
| - status  | int    | 검증 실패: 400, 존재하지 않는 TODO: 404, System Error: 500                |
| - message | String | 에러 내용                                                                 |
| - errors  | List   | 입력 검증 실패 시에만 포함. 필드별 { field, message } 전체 (API_SPEC 1.2) |

#### 처리 규칙

- 수정일(updatedAt) 자동 설정
- "시작 예정일이 과거" 검증은 startDate가 기존 값에서 변경된 경우에만 수행 (기존 startDate가 지난 TODO도 다른 필드 수정 가능)

#### 검증 에러 메시지

| 조건                               | 메시지                                              |
| ---------------------------------- | --------------------------------------------------- |
| 제목 누락                          | "제목을 입력해주세요"                               |
| 제목 200자 초과                    | "제목은 200자 이내로 입력해주세요"                  |
| 제목 공백만 입력                   | "제목을 입력해주세요"                               |
| 설명 1000자 초과                   | "설명은 1000자 이내로 입력해주세요"                 |
| 잘못된 우선순위 값                 | "우선순위는 HIGH, MEDIUM, LOW 중 하나를 선택하세요" |
| 시작 예정일이 과거                 | "시작 예정일은 오늘 이후 날짜를 선택하세요"         |
| 완료 예정일이 시작 예정일보다 과거 | "완료 예정일은 시작 예정일 이후 날짜를 선택하세요"  |
| 완료 예정일만 선택                 | "시작 예정일을 먼저 선택해주세요"                   |

### FR-005: TODO 완료 (완료일 업데이트)

- 설명: TODO를 done 칼럼으로 이동하여 완료 처리하고, 완료일(completedAt)을 자동 설정.
- API Endpoint: PUT /api/v1/todos/{id}/complete

#### 입력 필드:

| 필드     | 타입 | 필수 | 제약 조건                                                              | 기본값                     |
| -------- | ---- | ---- | ---------------------------------------------------------------------- | -------------------------- |
| position | int  | N    | done 칼럼 내 순서 index (0부터 시작). 서버가 실제 position 값을 재계산 | null (done 칼럼 가장 상단) |

- 요청 본문은 생략할 수 있다. 생략하면 position 미입력과 같게 처리한다.

#### 성공 응답 필드:

| 필드            | 타입              | 값                                            |
| --------------- | ----------------- | --------------------------------------------- |
| status          | int               | 200                                           |
| data            | Map<String, List> | 완료 처리로 영향받은 칼럼 (출발 칼럼 + done)  |
| - [status 값]   | List              | 해당 칼럼 전체 TODO (FR-007 성공 응답과 동일) |
| - - id          | String            | TODO id                                       |
| - - title       | String            | TODO title                                    |
| - - description | String            | TODO description                              |
| - - priority    | String            | TODO priority                                 |
| - - status      | String            | TODO status                                   |
| - - position    | int               | TODO position                                 |
| - - startDate   | LocalDate         | TODO startDate                                |
| - - startedAt   | LocalDateTime     | TODO startedAt                                |
| - - dueDate     | LocalDate         | TODO dueDate                                  |
| - - createdAt   | LocalDateTime     | TODO createdAt                                |
| - - updatedAt   | LocalDateTime     | TODO updatedAt                                |
| - - completedAt | LocalDateTime     | TODO completedAt                              |

- 예시: `{ "status": 200, "data": { "inProgress": [ ... ], "done": [ ... ] } }`

#### 실패 응답 필드:

| 필드      | 타입   | 값                                                                           |
| --------- | ------ | ---------------------------------------------------------------------------- |
| error     | Object | -                                                                            |
| - status  | int    | 검증 실패(이미 완료된 TODO): 400, 존재하지 않는 TODO: 404, System Error: 500 |
| - message | String | 에러 내용                                                                    |

#### 검증 에러 메시지

| 조건             | 메시지                   |
| ---------------- | ------------------------ |
| 이미 완료된 TODO | "이미 완료된 TODO입니다" |

#### 처리 규칙

- done으로 이동 시: completedAt = 현재 시각, status = done
- position 입력 시: 사용자가 Drop한 위치로 배치 (FR-007 position 재계산 로직 적용)
- position 미입력 시: done 칼럼 가장 상단에 배치
- done 칼럼에는 completedAt 기준 24시간 이내 Todo만 표시

### FR-006: TODO 삭제

- 설명: TODO를 영구 삭제한다.
- API Endpoint: DELETE /api/v1/todos/{id}

#### 성공 응답 필드:

| 필드   | 타입   | 값             |
| ------ | ------ | -------------- |
| status | int    | 200            |
| data   | String | 삭제한 TODO id |

#### 실패 응답 필드:

| 필드      | 타입   | 값                                         |
| --------- | ------ | ------------------------------------------ |
| error     | Object | -                                          |
| - status  | int    | 존재하지 않는 TODO: 404, System Error: 500 |
| - message | String | 에러 내용                                  |

### FR-007: 상태/순서 변경 (드래그 앤 드롭)

- 설명: TODO를 다른 칼럼으로 이동하거나 같은 칼럼 내에서 순서를 변경한다.
- API Endpoint: PUT /api/v1/todos/reorder

#### 입력 필드:

| 필드     | 타입   | 설명                                                                                                          |
| -------- | ------ | ------------------------------------------------------------------------------------------------------------- |
| id       | String | 이동할 TODO ID                                                                                                |
| status   | String | 이동 대상 칼럼 (todo, inProgress, done)                                                                       |
| position | int    | 이동 대상 칼럼 내 순서 index (0부터 시작, 이동하는 카드를 제외한 목록 기준). 서버가 실제 position 값을 재계산 |

#### 성공 응답 필드:

| 필드            | 타입              | 값                                                                                                              |
| --------------- | ----------------- | --------------------------------------------------------------------------------------------------------------- |
| status          | int               | 200                                                                                                             |
| data            | Map<String, List> | 이동으로 영향받은 칼럼 (출발·도착 칼럼, 같은 칼럼 내 이동 시 1개)                                               |
| - [status 값]   | List              | 해당 칼럼 전체 TODO (position 오름차순, done 칼럼은 FR-002와 동일하게 completedAt 기준 24시간 이내 TODO만 포함) |
| - - id          | String            | TODO id                                                                                                         |
| - - title       | String            | TODO title                                                                                                      |
| - - description | String            | TODO description                                                                                                |
| - - priority    | String            | TODO priority                                                                                                   |
| - - status      | String            | TODO status                                                                                                     |
| - - position    | int               | TODO position                                                                                                   |
| - - startDate   | LocalDate         | TODO startDate                                                                                                  |
| - - startedAt   | LocalDateTime     | TODO startedAt                                                                                                  |
| - - dueDate     | LocalDate         | TODO dueDate                                                                                                    |
| - - createdAt   | LocalDateTime     | TODO createdAt                                                                                                  |
| - - updatedAt   | LocalDateTime     | TODO updatedAt                                                                                                  |
| - - completedAt | LocalDateTime     | TODO completedAt                                                                                                |

- 예시: `{ "status": 200, "data": { "todo": [ ... ], "inProgress": [ ... ] } }`
- 클라이언트는 응답받은 칼럼의 리스트로 해당 칼럼 상태를 교체 (낙관적 업데이트 확정)

#### 실패 응답 필드:

| 필드      | 타입   | 값                                                                                                                                             |
| --------- | ------ | ---------------------------------------------------------------------------------------------------------------------------------------------- |
| error     | Object | -                                                                                                                                              |
| - status  | int    | 검증 실패(id 누락, status 누락, 잘못된 status 값, position 누락, done에서 done으로 순서 변경): 400, 존재하지 않는 TODO: 404, System Error: 500 |
| - message | String | 에러 내용                                                                                                                                      |
| - errors  | List   | 입력 검증 실패 시에만 포함. 필드별 { field, message } 전체 (API_SPEC 1.2)                                                                      |

#### 검증 에러 메시지

| 조건                        | 메시지                                               |
| --------------------------- | ---------------------------------------------------- |
| id 누락                     | "잘못된 요청입니다."                                 |
| status 누락                 | "이동할 상태를 입력해주세요"                         |
| 잘못된 status 값            | "상태는 todo, inProgress, done 중 하나를 선택하세요" |
| position 누락               | "이동할 위치를 입력해주세요"                         |
| done에서 done으로 순서 변경 | "완료된 TODO는 순서를 변경할 수 없습니다"            |

#### 처리 규칙:

- 상태(status)와 순서(position) 동시 업데이트
- 트랜잭션으로 원자성 보장

#### position 재계산 로직:

- 입력 index → 삽입 위치: index = 0이면 맨 앞, index = 칼럼 카드 수이면 맨 뒤, 그 외에는 index-1번째와 index번째 카드 사이
- 범위를 벗어난 index: 음수는 맨 앞, 칼럼 카드 수 초과는 맨 뒤로 보정
- 두 카드 사이에 삽입할 때: (prev + next) / 2로 계산
- 정수 나눗셈 결과 간격이 없을 때: 해당 칼럼 전체를 1024 간격으로 재정렬
- 맨 앞 삽입: 첫 번째 카드의 position - 1024
- 맨 뒤 삽입: 마지막 카드의 position + 1024

#### 비즈니스 로직:

- done에서 done으로 순서 변경은 할 수 없다.
- todo에서 inProgress로 이동 시: startedAt = 현재 시각, completedAt = null
- done에서 inProgress로 이동 시: completedAt = null, startedAt이 null이면 현재 시간 기록
- todo로 이동 시: startedAt = null, completedAt = null
- done 으로 이동시: done으로 옮긴 todo의 id와 position으로 FR-005 호출, FR-005 응답을 그대로 반환

## 비기능적 요구사항 (Non-Functional Requirements)

### NFR-001: 반응형

- 모바일: (360px ~ ): 단일 컬럼 스크롤 뷰, 터치 드래그 지원
- 태블릿: (768px ~ ): 2컬럼 그리드 (첫 번째 Row = todo, inProgress. 두 번째 Row = done).
- 데스크탑: (1024px ~ ): 3컬럼 가로 배치.

### NFR-002: 무결성

- 드래그 앤 드롭: 낙관적 업데이트 진행. UI 즉시 반영 -> API 성공시 확정 -> 실패시 롤백
- position 값은 정수로 관리. 충돌시 재정렬

### NFR-003: 보안 설정 (Spring Security)

- 인증/인가 없음: `/api/v1/**` 전체 permitAll
- CSRF 비활성화 (세션을 사용하지 않는 REST API)
- CORS 허용 Origin: Vite 개발 서버 `http://localhost:5173`

## User Scenario

### US-001: 새 할 일 등록

> 사용자는 떠오른 할 일을 빠르게 등록할 수 있다.

- [ ] "새 Todo" 버튼 클릭시 생성폼이 열린다.
- [ ] 제목만 입력하고 생성하면 todo 가장 위에 todo 카드가 생성된다.
- [ ] 우선 순위 미 선택시 "MEDIUM"이 기본 적용된다.
- [ ] 생성 완료 후 폼이 닫힌다.

#### 관련 FR: FR-001

### US-002: 상세 정보 설정

> 사용자는 할 일에 설명, 우선순위, 시작 예정일, 완료 예정일을 설정할 수 있다.

- [ ] 생성 시, 설명, 우선순위, 시작 예정일, 완료 예정일을 선택적으로 입력할 수 있다.
- [ ] 시작 예정일, 완료 예정일은 오늘 이후 날짜로만 선택가능하다. (오늘 포함)
- [ ] 완료 예정일은 항상 시작 예정일 보다 이후 날짜로만 선택 가능하다. (시작 예정일 당일 포함)
- [ ] 완료 예정일은 시작 예정일을 선택했을 때만 선택할 수 있다.

#### 관련 FR: FR-001

### US-003: 할 일 목록 파악

> 사용자는 모든 할 일들이 상태별로 분류된 목록을 확인할 수 있다.

- [ ] 3개 컬럼 (todo, in progress, done)에 할 일이 분류되어 표시된다.
- [ ] 각 컬럼 상단에 할 일 수가 표시된다.
- [ ] 컬럼 내 할 일은 position 순서대로 정렬된다.

#### 관련 FR: FR-002

### US-004: 드래그 앤 드롭 상태 변경

> 사용자는 할 일을 드래그해 다른 컬럼으로 옮길 수 있다.

- [ ] 할 일을 드래그 해 다른 컬럼에 드롭하면 상태가 즉시 변경된다.
- [ ] 같은 컬럼내에서 드래그 앤 드롭해 순서를 변경할 수 있다. (done 컬럼 제외)
- [ ] 어떤 컬럼에서든 어떤 컬럼으로 이동이 가능하다. (역방향 포함)
- [ ] API 실패 시 원래 위치로 롤백된다.

#### 관련 FR: FR-007

### US-005: 할 일 완료 처리

> 사용자는 할 일을 done으로 옮기면 완료처리 된다.

- [ ] todo 컬럼에서 inProgress 컬럼으로 이동시 시작일 (startedAt)이 기록된다.
- [ ] done 컬럼에서 inProgress 컬럼으로 이동시 시작일 (startedAt)이 없을 때만 기록된다.
- [ ] todo 컬럼으로 이동시 시작일 (startedAt)이 초기화 된다.
- [ ] done 컬럼으로 이동시 완료일 (completedAt)이 기록된다.
- [ ] done 컬럼에서 다른 컬럼으로 이동시 완료일 (completedAt)이 초기화 된다.

#### 관련 FR: FR-005, FR-007

### US-006: 할 일 수정

> 사용자는 기존 할 일의 내용을 수정할 수 있다.

- [ ] 할 일을 클릭하면 상세 모달이 열린다.
- [ ] 모달에서 제목, 설명, 우선순위, 시작 예정일, 완료 예정일을 수정할 수 있다.
- [ ] 수정 후 목록에 변경 사항이 즉시 반영된다.

#### 관련 FR: FR-003, FR-004

### US-007: 할 일 삭제

> 사용자는 불필요한 할 일을 삭제할 수 있다.

- [ ] 상세 모달에서 삭제 버튼이 있다.
- [ ] 삭제 버튼 클릭시 확인 다이얼로그가 표시된다.
- [ ] 확인 후 할 일이 영구 삭제되고 목록에서 사라진다.

#### 관련 FR: FR-006

### US-008: 할 일 검색

> 사용자는 제목으로 할 일을 찾을 수 있다.

- [ ] 보드 위 한 줄 검색창에 검색어를 입력하면 모든 칼럼(todo, in progress, done)에서 제목에 검색어가 포함된 카드만 표시된다.
- [ ] 띄어쓰기를 무시하고 비교한다. (예: 검색어 "대신" → 제목 "선발대 신설" 표시)
- [ ] 검색어를 지우면 전체 카드가 다시 표시된다.

#### 관련 FR: FR-002 (조회한 목록을 클라이언트에서 필터링, API 변경 없음)

## 코딩 컨벤션

### Client

- Component

```
const ExamComponent = () => {
    // on[TagName or ClassName][EventName]Handler
    const onButtonClickHandler = () => {
        alert("Clicked");
    };

    return (
        <div>
            <button onClick={onButtonClickHandler}>Click Me</button>
        </div>
    );
};
export default ExamComponent
```

- Promise 패턴 대신 async/await 패턴 사용
- src/utils/fetchUtil.js 내부에 공통 fetch 코드 필요.
  - Request, Response에 대응할 Interceptor 기능 구현.
- Redux Thunk
  - `createAsyncThunk`를 사용하지 않는다.
  - 요청 단계별 action은 slice의 `reducers`에 `[thunk명]Start`, `[thunk명]Success`, `[thunk명]Failure`로 정의한다. action은 일반 객체(`{ type, payload }`)다.
  - thunk는 `store/actions`에 `(dispatch, getState)`를 받는 함수를 반환하는 함수로 직접 작성한다. (async/await)
  - thunk 반환값: 성공 `{ ok: true, data }`, 실패 `{ ok: false, error }` (`error` = `{ status, message, errors }`). 컴포넌트는 `ok`로 성공·실패를 판단한다.
  - 요청을 구분하는 id가 필요하면 `@reduxjs/toolkit`의 `nanoid()`로 만든다.

```
// store/slices/todoSlice.js
reducers: {
    fetchTodosStart: (state) => { state.loading.list = true },
    fetchTodosSuccess: (state, action) => { state.loading.list = false; state.columns = action.payload },
    fetchTodosFailure: (state, action) => { state.loading.list = false; state.error = action.payload.message },
}

// store/actions/todoActions.js
export const fetchTodos = () => async (dispatch) => {
    dispatch(fetchTodosStart())
    try {
        const data = await todoApi.getTodos()
        dispatch(fetchTodosSuccess(data))
        return { ok: true, data }
    } catch (error) {
        const payload = toErrorPayload(error)
        dispatch(fetchTodosFailure(payload))
        return { ok: false, error: payload }
    }
}

// 컴포넌트
const result = await dispatch(fetchTodos())
if (result.ok) { ... }
```

### Server

- 기능 개발 시작 전, 반드시 아래 단계로 구현.
  1. junit 실패 테스트 케이스 작성
  2. 실패 검증 (Validation, Exception)
  3. junit 성공 테스트 케이스 작성
  4. 성공 검증
  - Given - When - Then - Verify 패턴으로 junit 작성.

### MyBatis

#### SELECT

- 단일 테이블내에서 조회시 resultType 사용
- 여러 테이블에서 복합 컬럼 조회시 resultMap 사용
  - association, collection 적극 사용 권장.
  - VO는 `has a` 규칙으로 구성.

#### Query 작성 규칙

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

## 금지사항

### 절대 하지 말아야 할 것.

- DTO 사용.
- 명세에 없는 기능 추가.
- front-end에서 db에 직접 접근.
- 공식 문서확인 없이 추측으로 구현.
- 삼항 연산자 사용.
- if, for, while 등에서 중괄호 생략.
- js 세미콜론 생략.

### 반드시 확인 받아야 할 것.

- DB 스키마 변경.
- API Spec 변경.
- 의존 라이브러리(패키지) 추가/변경/삭제

## 개발 중 에러 발생시 대처 상황

- 재시도 하지 않음.
- 에러가 발생한 원인과 대처 방법을 간단히 제공.
