# 회원가입·인증

> 작성일: 2026-05-14 | 수정일: 2026-05-14 | 유형: 정책 | 관련 레포: letsgojh0810/commerce-backend

## 회원 가입 흐름

1. 클라이언트가 `POST /api/v1/users`로 loginId, 비밀번호, 기타 프로필 정보를 전송합니다.
2. loginId 중복 여부를 users 테이블에서 확인합니다. 중복이면 400 에러를 반환합니다.
3. 비밀번호 정책을 검증합니다. 위반 시 400 에러를 반환합니다.
4. 비밀번호를 BCrypt로 인코딩합니다 (Spring Security Crypto).
5. users 테이블에 사용자 정보를 저장합니다.
6. 201 Created와 함께 생성된 사용자 정보를 반환합니다.

## 비밀번호 정책

- 길이: **8자 이상 16자 이하**
- 구성: **영문 + 숫자 + 특수문자** 모두 포함 필수
- 금지: **생년월일 포함 불가** (입력된 birthDate와 비교)

정책 위반 시 400 Bad Request를 반환하며, 위반 항목을 에러 메시지에 포함합니다.

## 인증 방식

JWT 토큰을 발급하지 않습니다. 보호 API를 호출할 때마다 아래 헤더를 전송해야 합니다.

| 헤더 | 값 | 비고 |
|------|-----|------|
| `X-Loopers-LoginId` | 사용자 loginId | 필수 |
| `X-Loopers-LoginPw` | 사용자 비밀번호 (평문) | 필수 |

서버는 매 요청마다 users 테이블에서 loginId로 사용자를 조회하고, BCrypt `matches()`로 비밀번호를 검증합니다.

## 어드민 인증

어드민 API(`/api-admin/v1/*`)는 일반 인증 헤더 외에 아래 헤더가 추가로 필요합니다.

| 헤더 | 값 |
|------|-----|
| `X-Loopers-Ldap` | `loopers.admin` |

## API 목록

| 메서드 | 경로 | 설명 | 인증 |
|--------|------|------|------|
| POST | `/api/v1/users` | 회원 가입 | 불필요 |
| GET | `/api/v1/users/me` | 내 정보 조회 | X-Loopers 헤더 |
| PUT | `/api/v1/users/me` | 내 정보 수정 | X-Loopers 헤더 |
| PUT | `/api/v1/users/me/password` | 비밀번호 변경 | X-Loopers 헤더 |
| GET | `/api-admin/v1/users` | 어드민 사용자 목록 조회 | X-Loopers 헤더 + X-Loopers-Ldap |
| GET | `/api-admin/v1/users/{userId}` | 어드민 사용자 상세 조회 | X-Loopers 헤더 + X-Loopers-Ldap |
| PUT | `/api-admin/v1/users/{userId}` | 어드민 사용자 수정 | X-Loopers 헤더 + X-Loopers-Ldap |
| DELETE | `/api-admin/v1/users/{userId}` | 어드민 사용자 삭제(소프트 딜리트) | X-Loopers 헤더 + X-Loopers-Ldap |

## 관련 entity

`user` — `ontology/abox/commerce-user.yaml` 참조
