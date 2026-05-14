# Glossary — 사용자

> 이 도메인에서 자주 쓰이는 용어를 풀어 정리합니다.

## 데이터/엔티티

**loginId**
사용자 로그인 식별자. 가입 시 지정하며 이후 변경 불가. 시스템 내에서 중복을 허용하지 않습니다. 인증 헤더(`X-Loopers-LoginId`)에 사용됩니다.

**BCrypt**
비밀번호 해시 알고리즘. Spring Security Crypto 라이브러리를 통해 적용합니다. users 테이블에는 평문 비밀번호가 아닌 BCrypt 해시값만 저장됩니다. 인증 시 입력된 평문을 해시와 비교(matches)하여 검증합니다.

**소프트 딜리트 (Soft Delete)**
`deleted_at` 컬럼에 삭제 시각을 기록하는 논리 삭제 방식. 실제 행은 users 테이블에 남아 있으며, 조회 시 `deleted_at IS NULL` 조건으로 필터링합니다.

**어드민 API**
`/api-admin/v1/*` 경로로 제공되는 관리자 전용 API. 일반 인증 헤더(`X-Loopers-LoginId`, `X-Loopers-LoginPw`) 외에 `X-Loopers-Ldap: loopers.admin` 헤더가 추가로 필요합니다.
