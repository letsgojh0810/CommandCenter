# 좋아요

> 작성일: 2026-05-14 | 수정일: 2026-05-14 | 유형: 정책 | 관련 레포: letsgojh0810/commerce-backend

## 개요

사용자가 상품에 좋아요를 표시하거나 취소하는 기능입니다. 동시 요청에 의한 likeCount 불일치를 낙관적 락으로 방지하며, 좋아요 이벤트는 AFTER_COMMIT 후 처리됩니다.

## API 목록

| 메서드 | 경로 | 설명 | 인증 |
|--------|------|------|------|
| POST | `/api/v1/products/{productId}/likes` | 좋아요 | X-Loopers 헤더 |
| DELETE | `/api/v1/products/{productId}/likes` | 좋아요 취소 | X-Loopers 헤더 |
| GET | `/api/v1/users/{userId}/likes` | 내 좋아요 목록 조회 | X-Loopers 헤더 |

## 낙관적 락

`product` 엔티티에 `@Version` 어노테이션을 적용하여 동시 좋아요 충돌을 방지합니다. 같은 시점에 여러 요청이 likeCount를 갱신하려 할 때, 버전 불일치가 발생한 요청은 재시도합니다.

## 이벤트 흐름

1. 좋아요/취소 API 요청이 성공하면 likes 테이블에 저장합니다.
2. 트랜잭션 커밋 이후(`AFTER_COMMIT`) `LikeCreatedEvent` 또는 `LikeCancelledEvent`가 발행됩니다.
3. 이벤트 핸들러가 `product.likeCount`를 갱신합니다 (낙관적 락 적용).
4. 변경 사항은 Outbox 패턴을 경유하여 Kafka `catalog-events` 토픽에 발행됩니다.
5. 메트릭·랭킹 도메인이 `catalog-events` 토픽을 구독하여 이벤트를 소비합니다.

```
좋아요 API 요청
    → likes 테이블 저장
    → [AFTER_COMMIT] LikeCreatedEvent 발행
    → product.likeCount 갱신 (낙관적 락)
    → Outbox 저장
    → Kafka catalog-events 발행
    → 메트릭·랭킹 도메인 소비
```

## 관련 entity

`like`, `product` — `ontology/abox/commerce-catalog.yaml` 참조
