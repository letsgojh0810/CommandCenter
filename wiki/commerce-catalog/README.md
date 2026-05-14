# 상품 카탈로그 (commerce-catalog)

> 작성일: 2026-05-14 | 수정일: 2026-05-14 | 유형: 정책 | 관련 레포: letsgojh0810/commerce-backend

## 배경

브랜드·상품·좋아요를 관리하는 도메인입니다. 상품 조회는 Redis 캐시를 통해 읽기 성능을 보장하고, 좋아요 이벤트를 통해 상품의 likeCount를 갱신합니다. 상품 변경 및 좋아요 이벤트는 Kafka `catalog-events` 토픽으로 메트릭·랭킹 도메인에 전달됩니다.

## 핵심 정책

- **Redis 캐시**: 상품 상세(`productDetail`) TTL 10분, 상품 목록(`productList`) TTL 5분. 상품 변경 시 캐시 무효화.
- **좋아요 동시성**: 낙관적 락(`@Version`)으로 동시 좋아요 충돌을 방지합니다.
- **이벤트 전파**: 좋아요 생성/취소 이벤트는 AFTER_COMMIT 후 product.likeCount를 갱신하고, Outbox 경유로 Kafka `catalog-events`에 발행합니다.

## 관련 레포

`letsgojh0810/commerce-backend`

## 주요 주제 문서

| 주제 | 문서 링크 | 설명 |
|------|-----------|------|
| 브랜드 | [브랜드/](브랜드/README.md) | 브랜드 조회·어드민 관리 |
| 상품 조회 | [상품-조회/](상품-조회/README.md) | 목록/상세 조회, Redis 캐시 전략, 정렬 옵션 |
| 좋아요 | [좋아요/](좋아요/README.md) | 좋아요/취소, 낙관적 락, 이벤트 흐름 |

## 관련 ontology

`ontology/abox/commerce-catalog.yaml`
