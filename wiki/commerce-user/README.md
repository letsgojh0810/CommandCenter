# 사용자 (commerce-user)

> 작성일: 2026-05-14 | 수정일: 2026-05-14 | 유형: 정책 | 관련 레포: letsgojh0810/commerce-backend

## 배경

회원 가입·인증·프로필 관리를 담당하는 도메인입니다. 사용자는 loginId와 비밀번호로 가입하고, 이후 모든 보호 API는 요청 헤더에 자격증명을 실어 매 요청마다 인증합니다. JWT 토큰을 발급하지 않으며, 세션 저장소도 사용하지 않습니다.

## 인증 방식

| 헤더 | 값 | 용도 |
|------|-----|------|
| `X-Loopers-LoginId` | 사용자 loginId | 모든 보호 API 인증 |
| `X-Loopers-LoginPw` | 사용자 비밀번호 (평문) | 모든 보호 API 인증 |
| `X-Loopers-Ldap` | `loopers.admin` | 어드민 API 전용 접근 제어 |

- 보호 API를 호출할 때마다 `X-Loopers-LoginId` + `X-Loopers-LoginPw` 헤더를 함께 전송해야 합니다.
- 어드민 API(`/api-admin/v1/*`)는 `X-Loopers-Ldap: loopers.admin` 헤더가 추가로 필요합니다.

## 관련 레포

`letsgojh0810/commerce-backend`

## 주요 주제 문서

| 주제 | 문서 링크 | 설명 |
|------|-----------|------|
| 회원가입·인증 흐름 | [회원가입-인증/](회원가입-인증/README.md) | 가입 흐름, 비밀번호 정책, 인증 방식, API 목록 |

## 관련 ontology

`ontology/abox/commerce-user.yaml`
