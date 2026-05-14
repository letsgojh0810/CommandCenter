# 브랜드

> 작성일: 2026-05-14 | 수정일: 2026-05-14 | 유형: 정책 | 관련 레포: letsgojh0810/commerce-backend

## 개요

브랜드는 상품의 상위 분류 단위입니다. 모든 상품은 반드시 하나의 브랜드에 속합니다. 브랜드 생성·수정·삭제는 어드민 API로만 가능하며, 일반 사용자는 조회만 할 수 있습니다.

## 브랜드 필드

| 필드 | 설명 |
|------|------|
| `name` | 브랜드 이름 |
| `description` | 브랜드 설명 |
| `imageUrl` | 브랜드 대표 이미지 URL |

## API 목록

### 사용자 API

| 메서드 | 경로 | 설명 | 인증 |
|--------|------|------|------|
| GET | `/api/v1/brands` | 브랜드 목록 조회 | 불필요 |
| GET | `/api/v1/brands/{brandId}` | 브랜드 상세 조회 | 불필요 |

### 어드민 API

| 메서드 | 경로 | 설명 | 인증 |
|--------|------|------|------|
| POST | `/api-admin/v1/brands` | 브랜드 생성 | X-Loopers-Ldap: loopers.admin |
| PUT | `/api-admin/v1/brands/{brandId}` | 브랜드 수정 | X-Loopers-Ldap: loopers.admin |
| DELETE | `/api-admin/v1/brands/{brandId}` | 브랜드 삭제 | X-Loopers-Ldap: loopers.admin |

## 관련 entity

`brand` — `ontology/abox/commerce-catalog.yaml` 참조
