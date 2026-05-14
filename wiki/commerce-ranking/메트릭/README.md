# 메트릭 수집

> 작성일: 2026-05-14 | 수정일: 2026-05-14 | 유형: 정책 | 관련 레포: letsgojh0810/commerce-backend

## 메트릭 수집 흐름

Kafka 이벤트를 수신하여 `product_metrics` 테이블을 갱신하고 Redis ZSet 점수를 적립합니다.

```
Kafka (catalog-events / order-events)
  → CatalogEventConsumer / OrderEventConsumer
  → product_metrics 갱신 (MySQL)
  → Redis ZSet 점수 적립
```

## 이벤트별 처리

| 이벤트 | 처리 내용 |
|--------|-----------|
| `LIKE_CREATED` | `likeCount` +1, `addLikeScore()` |
| `LIKE_CANCELLED` | `likeCount` -1 |
| `PRODUCT_VIEWED` | `viewCount` +1, `addViewScore()` |
| `ORDER_COMPLETED` | `salesCount` +N, `addOrderScore()` |

## Idempotency

`event_handled` 테이블을 사용하여 동일한 이벤트의 중복 처리를 방지합니다. 컨슈머는 이벤트 처리 전에 해당 이벤트 ID가 `event_handled`에 존재하는지 확인합니다.

## 낙관적 락

`product_metrics` 갱신 시 낙관적 락(Optimistic Lock)을 사용하여 동시 갱신 충돌을 방지합니다. 충돌 발생 시 재시도합니다.
