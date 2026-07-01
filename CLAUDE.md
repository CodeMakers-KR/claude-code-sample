# 레시피 공유 커뮤니티 설계

## 프로젝트 소개
다수의 회원이 참여해 자신만의 레시피를 공유하고 "맛있어요", "익숙한 맛이에요", "최고에요", "입맛에 안맞아요" 와 같은 피드백과 댓글 달기 등을 이용하는 레시피 공유 커뮤니티.

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
- [ ] 회원 가입 - 이메일(pk), 비밀번호, 이름, 닉네임(uk), 프로필 사진
- [ ] 로그인 - 이메일과 비밀번호를 입력해 로그인, 30분 내 5번 실패 시 60분간 로그인 시도 차단
- [ ] 비밀번호 찾기 - 이메일, 이름, 닉네임을 입력해 일치하는 정보가 있을 경우, 이메일로 비밀번호 변경 링크 전송.

#### 레시피
- [ ] 레시피 검색 - 테마별(날씨, 기온, 기분 등등), 주제별(생일, 기념일 등등), 재료별, 높은 만족도, 댓글 수, 키워드 검색 등등
- [ ] 레시피 조회 - 검색한 레시피의 내용을 조회, 레시피 내용 소개 하단에 레시피 요약 정보, 댓글, 피드백이 나오도록 한다.
- [ ] 레시피 등록 - 회원만 이용 가능. 레시피 제목, 준비물, 레시피 설명, 이미지, 유튜브 링크, 인스타그램 링크 등등 부가 설명자료 첨부할 수 있음. 위지윅 에디터 사용. 
- [ ] 레시피 수정 - 작성자만 이용 가능.
- [ ] 레시피 삭제 - 작성자만 이용 가능. 댓글이나 피드백이 있을 경우 삭제 불가능.
- [ ] 레시피 댓글 등록 - 회원만 이용 가능. 대댓글 등록 가능. 레시피 내용 인용할 수 있다. 위지윅 에디터 사용. 이미지, 유튜브 링크, 인스타그램 링크 등등 첨부 가능.
- [ ] 레시피 댓글 수정 - 댓글 작성자만 이용 가능. 대댓글이 있는 댓글은 수정 불가능.
- [ ] 레시피 댓글 삭제 - 댓글 작성자만 이용 가능. 대댓글이 있는 댓글은 삭제 불가능.
- [ ] 레시피 댓글 조회 - 댓글들이 조회된다. (작성자, 작성일, 수정일, 댓글 내용) 삭제된 댓글은 "삭제된 댓글입니다" 로 표현.
- [ ] 레시피 피드백 등록 - 회원만 이용가능. "맛있어요", "익숙한 맛이에요", "최고에요", "입맛에 안맞아요" 중 하나를 선택. 중복 선택은 불가능.

#### 실시간 채팅, 쪽지
- [ ] 실시간 채팅 - 회원간 실시간 채팅이 가능. 그룹 채팅, 1:1 채팅, 비밀대화.
      채팅이 도착할 경우 알림이 나타난다.
- [ ] 실시간 쪽지 - 회원간 쪽지를 주고 받을 수 있다.
      쪽지가 도착할 경우 알림이 나타난다.

#### 마이페이지
- [ ] 내가 등록한 레시피
- [ ] 내가 본 레시피
- [ ] 내가 등록한 댓글
- [ ] 내가 남긴 피드백
- [ ] 받은 쪽지
- [ ] 보낸 쪽지
- [ ] 채팅 기록
- [ ] 내 정보 수정 - 이메일을 제외한 나머지 정보를 수정할 수 있음.
- [ ] 탈퇴

#### 시스템 관리 (관리자만 사용 가능)
- [ ] 환경 변수 - 커뮤니티에서 사용하게되는 다양한 환경 변수를 관리 (예> API Host url, 레시피 검색시 테마 종류, 주제 종류, 재료 종류 등등)
- [ ] 사용자 관리 - 개인정보는 마스킹 처리
- [ ] 관리자 관리 - 기존 관리자 관리, 새로운 관리자 생성, 삭제 등

### Dashboard
- [ ] 마이페이지의 데이터를 기반으로 레시피 추천
- [ ] 마이페이지의 데이터를 기반으로 나의 취향과 전체 회원 취향과의 비교
- [ ] 내가 등록한 댓글이나 레시피에 새로 등록된 댓글 알림
- [ ] 내가 등록한 댓글이나 레시피에 새로 등록된 피드백 알림

## ERD
- 상세 요구사항을 기반으로한 ERD 생성
- PK, UK 사용
- FK: 논리적 모델에서만 사용. 물리적 모델에서는 사용 하지 않음.
