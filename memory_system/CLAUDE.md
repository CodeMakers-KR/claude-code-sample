# 레시피 공유 커뮤니티 설계

## 프로젝트 소개
- 주제: 다수의 회원이 참여해 자신만의 레시피를 공유하는 커뮤니티
- 기간: 21일
- 팀: 6명 (풀스택 개발자 6명)
- 예산: 무료

## 제약사항
- SALT, 결제정보, 민감 개인정보는 반드시 별도 데이터베이스에 다중 암호화(DKE:Double Key Encryption) 되어있어야 한다.
- DB에서 암/복호화 할 수 있도록 함수 개발 필요.
- 비밀번호는 SHA 알고리즘으로 암호화.

## 인증 방식
- 4시간 동안 유효한 IP 기반 JWT (Access Token, Refresh Token)
  - 다른 PC에서 발급받은 JWT는 사용할 수 없다.
  
## 회원 등급
- 일반 사용자: 일반 권한을 가지고 있는 사용자, 관리자 권한은 없음. 회원가입을 통해 생성
- 관리자: 일반 권한 + 관리자 권한을 가지고 있는 사용자. 시스템 관리 > 관리자 등록을 통해 생성

## 이 프로젝트에서 참고할 메모리 파일 목록
- 사용기술: memory/00_skills.md
- 프로젝트 구조: memory/10_architectures.md
- 요구사항: memory/20_require_functions.md
- DB 규칙: memory/30_db.md
- Frontend Coding Guide: memory/40_front_end_coding_guide.md
- Backend Coding Guide: memory/41_back_end_coding_guide.md
