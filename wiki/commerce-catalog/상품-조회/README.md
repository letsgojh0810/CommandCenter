# 상품 조회

> 작성일: 2026-05-14 | 수정일: 2026-05-14 | 유형: 정책 | 관련 레포: letsgojh0810/commerce-backend

## 상품 목록 조회

`GET /api/v1/products`

| 파라미터 | 타입 | 설명 |
|---------|------|------|
| `brandId` | Long (optional) | 특정 브랜드로 필터링 |
| `sort` | String (optional) | 정렬 옵션 (기본값: `LATEST`) |
| `page` | int (optional) | 페이지 번호 (0-based) |
| `size` | int (optional) | 페이지 크기 |

## 상품 상세 조회

`GET /api/v1/products/{productId}`

productId에 해당하는 단일 상품의 상세 정보를 반환합니다. 존재하지 않는 productId 요청 시 404를 반환합니다.

## 정렬 옵션

| 옵션 | 설명 |
|------|------|
| `LATEST` | 최신 등록순 (기본값) |
| `PRICE_ASC` | 가격 낮은 순 |
| `LIKES_DESC` | 좋아요 많은 순 |

## 캐시 전략

| 캐시 키 | TTL | 무효화 시점 |
|---------|-----|-----------|
| `productDetail:{productId}` | 10분 | 해당 상품 정보 변경 시 |
| `productList:{brandId}:{sort}:{page}:{size}` | 5분 | 상품 정보 변경 시 |

- 캐시 히트 시 Redis에서 즉시 응답합니다.
- 캐시 미스 시 MySQL에서 조회 후 Redis에 저장하고 응답합니다.
- 상품 생성·수정·삭제 시 관련 캐시를 무효화합니다.

## 어드민 API

| 메서드 | 경로 | 설명 | 인증 |
|--------|------|------|------|
| POST | `/api-admin/v1/products` | 상품 생성 | X-Loopers-Ldap: loopers.admin |
| PUT | `/api-admin/v1/products/{productId}` | 상품 수정 | X-Loopers-Ldap: loopers.admin |
| DELETE | `/api-admin/v1/products/{productId}` | 상품 삭제 | X-Loopers-Ldap: loopers.admin |

## 관련 entity

`product`, `brand` — `ontology/abox/commerce-catalog.yaml` 참조
