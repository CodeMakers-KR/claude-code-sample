# 대형 커뮤니티 프로젝트 아키텍처 설계 요청
## 주요기능
- 사용자 관리 (권한 부여를 포함한)
- 로그인
- 커뮤니티 게시판 관리 (게시판 생성, 수정, 삭제, 조회, 권한 제어)
- 관리자 관리
- 중고 장터

## 중고 장터
### 제약 조건
- 3레벨 이상의 사용자만 이용 가능
- 3레벨 미만의 사용자는 읽기만 가능

### 주요 기능
- 중고 상품 등록
- 거래 제안
- 할인 제안
- 위법 상품 신고

## 기술 스택
- Front-End: React.js 18+ & ECMAScript
- Back-End: Spring Boot 3.4.x & Java 21 & MyBatis & Spring Security 
- Database: Oracle 21c XE
- DevOps: Docker & Docker Compose
- Prod OS: Azure VM Ubuntu (2cpu, 4GB Ram, 30GB SSD)
- CI/CD: Jenkins
