# Architecture — 랭킹/메트릭

> 작성일: 2026-05-14 | 수정일: 2026-05-14 | 유형: 정책 | 관련 레포: letsgojh0810/commerce-backend

## 실시간 메트릭 수집 흐름

```mermaid
graph LR
    A[catalog-events / order-events<br/>Kafka 토픽] --> B[CatalogEventConsumer<br/>OrderEventConsumer]
    B --> C[product_metrics<br/>MySQL - 실시간 누적]
    B --> D[Redis ZSet<br/>실시간 랭킹 점수]
```

| 이벤트 | 처리 |
|--------|------|
| `LIKE_CREATED` | likeCount +1, Redis addLikeScore |
| `LIKE_CANCELLED` | likeCount -1 |
| `PRODUCT_VIEWED` | viewCount +1, Redis addViewScore |
| `ORDER_COMPLETED` | salesCount +N, Redis addOrderScore |

## Spring Batch 집계 흐름

```mermaid
graph TD
    A[product_metrics<br/>실시간 테이블] -->|MetricsSnapshotJob<br/>매일 00:05| B[product_metrics_daily<br/>일별 스냅샷]
    B -->|RankingAggregationJob| C[mv_product_rank_weekly]
    B -->|RankingAggregationJob| D[mv_product_rank_monthly]
```

## RankingCarryOverScheduler

매일 23:50에 실행됩니다. 당일 Redis ZSet 점수의 10%를 다음날로 이월하여 랭킹의 연속성을 유지합니다.

```
다음날 초기 점수 = 당일 점수 × 0.1
```
