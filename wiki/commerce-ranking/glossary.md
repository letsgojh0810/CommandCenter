# Glossary — 랭킹/메트릭

> 작성일: 2026-05-14 | 수정일: 2026-05-14 | 유형: 정책 | 관련 레포: letsgojh0810/commerce-backend

## 테이블

**product_metrics**
commerce-streamer가 Kafka 이벤트를 수신하여 실시간으로 갱신하는 누적 카운터 테이블. `viewCount`, `likeCount`, `salesCount` 필드를 포함합니다.

**product_metrics_daily**
MetricsSnapshotJob이 매일 00:05에 `product_metrics`의 증분을 일별로 저장하는 스냅샷 테이블. RankingAggregationJob의 입력 데이터로 사용됩니다.

**mv_product_rank_weekly**
RankingAggregationJob이 `product_metrics_daily`를 기반으로 생성하는 주간 랭킹 뷰 테이블.

**mv_product_rank_monthly**
RankingAggregationJob이 `product_metrics_daily`를 기반으로 생성하는 월간 랭킹 뷰 테이블.

## 개념

**carry-over**
RankingCarryOverScheduler가 매일 23:50에 실행하는 연산입니다. 당일 Redis ZSet 랭킹 점수의 10%를 다음날 초기 점수로 이월하여 랭킹의 연속성을 유지합니다.

**Idempotency**
`event_handled` 테이블로 동일한 Kafka 이벤트가 중복 처리되는 것을 방지합니다. 이벤트 ID를 기준으로 처리 여부를 확인합니다.
