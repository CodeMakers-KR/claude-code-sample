# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 프로젝트 개요

레시피 공유 커뮤니티 — 회원이 레시피를 등록하고, 피드백("맛있어요" / "익숙한 맛이에요" / "최고에요" / "입맛에 안맞아요")과 댓글(대댓글 포함)을 남길 수 있는 서비스.

두 개의 독립 프로젝트로 구성된다:
- `frontend/` — Vite + React 18 SPA
- `backend/` — Spring Boot 3 REST API

---

## Frontend 개발 명령어

```bash
cd frontend
npm run dev       # 개발 서버 (Vite)
npm run build     # 프로덕션 빌드
npm run lint      # ESLint
npm run preview   # 빌드 결과물 로컬 미리보기
```

---

## Backend 개발 명령어

```bash
cd backend
./mvnw spring-boot:run              # 애플리케이션 실행
./mvnw test                         # 전체 테스트
./mvnw test -Dtest=ClassName        # 단일 테스트 클래스 실행
./mvnw test -Dtest=ClassName#method # 단일 테스트 메서드 실행
./mvnw package -DskipTests          # 테스트 제외 빌드
```

---

## 아키텍처

### Frontend 구조

```
src/
  api/          # Axios 인스턴스 및 API 호출 함수 (환경변수에서 API Host URL 참조)
  components/   # UI 컴포넌트 + *.module.css
  pages/        # React Router Dom 라우트 컴포넌트
  hooks/        # Custom Hook
  utils/        # 기타 유틸
  stores/
    slices/     # Redux Toolkit slice
    actions/    # Thunk action
  public/       # 이미지, Favicon
```

- 전역 상태: Redux Toolkit (slices + thunk actions)
- 컴포넌트 합성이 필요한 경우 ContextAPI 사용 (Redux가 아닌 합성용)
- 인증 토큰은 LocalStorage / SessionStorage에 보관

### Backend 구조

메인 패키지: `kr.codemakers.recipe`

```
kr.codemakers.recipe/
  common/
    config/
      security/
        filters/    # JWT 필터 등 Security 필터 체인
    exceptions/
      handlers/     # 전역 예외 핸들러
    utils/
    error/          # Error Controller
  {domain}/         # 도메인(DB 테이블)별 소문자 패키지
    web/            # REST Controller
    vo/
      request/      # 요청 데이터 클래스
      response/     # 응답 데이터 클래스
    service/        # 업무 로직 인터페이스
      impl/         # 인터페이스 구현체
    dao/            # MyBatis Mapper 인터페이스
    utils/          # 도메인 전용 유틸 (필요 시)
```

MyBatis XML mapper 위치: `src/main/resources/{domain}/mapper/{DaoName}Mapper.xml`

**DTO 사용 금지** — 요청/응답 데이터는 반드시 `vo/request`, `vo/response` 패키지의 VO 클래스로 처리.

### DB

OracleXE 21c — **물리 모델에서 FK 미사용** (논리 모델에서만 관계 표현). PK, UK만 물리적으로 적용.

---

## 인증 및 보안

- **IP 기반 JWT**: Access Token + Refresh Token, 유효기간 4시간. 발급된 IP와 다른 IP에서는 토큰 무효 처리.
- 로그인 실패 30분 내 5회 → 60분 잠금.
- 회원 등급: `ROLE_USER`(일반), `ROLE_ADMIN`(관리자). 관리자는 시스템 관리에서만 생성 가능.

---

## 도메인 핵심 규칙

- 레시피 삭제: 댓글 또는 피드백이 하나라도 있으면 삭제 불가.
- 댓글 수정/삭제: 대댓글이 달린 댓글은 수정/삭제 불가. 삭제된 댓글은 "삭제된 댓글입니다"로 표시.
- 피드백: 회원 1인당 레시피 1개에 하나의 피드백만 등록 가능 (중복 선택 불가).
- 레시피/댓글 작성 에디터: WYSIWYG 에디터 사용.
- 실시간 채팅(WebSocket/STOMP): 1:1, 그룹, 비밀대화 지원. 메시지/쪽지 수신 시 알림 표시.
