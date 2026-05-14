# 랭킹/메트릭 (commerce-ranking)

> 작성일: 2026-05-14 | 수정일: 2026-05-14 | 유형: 정책 | 관련 레포: letsgojh0810/commerce-backend

## 개요

Kafka 이벤트 기반으로 상품 메트릭(조회수, 좋아요수, 판매수)을 실시간으로 집계하고, Spring Batch로 주간/월간 랭킹을 생성하는 도메인입니다.

- 실시간 메트릭 집계: commerce-streamer (Kafka 컨슈머)
- 주간/월간 랭킹 생성: commerce-batch (Spring Batch)

## 점수 공식

```
랭킹 점수 = 0.1 × viewSum + 0.2 × likeSum + 0.7 × salesSum
```

TOP 100 상품을 대상으로 집계합니다.

## 랭킹 조회 API

`GET /api/v1/rankings?date=&page=&size=&period=daily|weekly|monthly`

| 파라미터 | 설명 |
|---------|------|
| `date` | 기준 날짜 |
| `page` / `size` | 페이지네이션 |
| `period` | `daily`, `weekly`, `monthly` 중 선택 |

## 주요 주제

| 주제 | 위치 |
|------|------|
| 메트릭 수집·Idempotency | [메트릭/](메트릭/README.md) |
| 랭킹 집계·Batch·CarryOver | [랭킹-집계/](랭킹-집계/README.md) |

## 관련 ontology

`ontology/abox/commerce-ranking.yaml`
