# 랭킹 집계

> 작성일: 2026-05-14 | 수정일: 2026-05-14 | 유형: 정책 | 관련 레포: letsgojh0810/commerce-backend

## MetricsSnapshotJob

- 실행 주기: 매일 00:05
- 동작: `product_metrics`(실시간 누적)의 증분을 `product_metrics_daily`(일별 스냅샷)에 저장합니다.

## RankingAggregationJob

- 입력: `product_metrics_daily`
- 출력: `mv_product_rank_weekly`, `mv_product_rank_monthly`
- 점수 공식:
  ```
  점수 = 0.1 × viewSum + 0.2 × likeSum + 0.7 × salesSum
  ```
- 대상: TOP 100 상품

## Redis 실시간 랭킹

이벤트 수신 시 Redis ZSet에 점수를 실시간으로 적립합니다.

| 메서드 | 이벤트 |
|--------|--------|
| `addLikeScore()` | LIKE_CREATED |
| `addViewScore()` | PRODUCT_VIEWED |
| `addOrderScore()` | ORDER_COMPLETED |

## RankingCarryOverScheduler

- 실행 주기: 매일 23:50
- 동작: 당일 Redis ZSet 점수 × 0.1을 다음날 초기 점수로 이월합니다.
- 목적: 오늘의 인기 상품이 다음날 랭킹에서 갑자기 0점이 되지 않도록 연속성을 유지합니다.

## 랭킹 조회 API

`GET /api/v1/rankings?date=&page=&size=&period=daily|weekly|monthly`

- `period=daily`: Redis ZSet 기반 실시간 랭킹
- `period=weekly`: `mv_product_rank_weekly` 기반
- `period=monthly`: `mv_product_rank_monthly` 기반
