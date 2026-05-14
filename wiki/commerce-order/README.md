# 주문/결제/쿠폰 (commerce-order)

> 작성일: 2026-05-14 | 수정일: 2026-05-14 | 유형: 정책 | 관련 레포: letsgojh0810/commerce-backend

## 개요

구매자의 주문 생성부터 PG 결제, 쿠폰 적용, 취소까지 책임지는 도메인입니다. Transactional Outbox 패턴으로 Kafka 이벤트의 정확히 한 번 전달을 보장하고, OpenFeign + Resilience4j로 PG 장애에 대응합니다.

## 주문 생성 흐름

```
POST /api/v1/orders
  → 대기열 토큰 검증 (optional, queue:token:{userId})
  → product.decreaseStock() — 재고 차감
  → 쿠폰 검증·사용 처리
  → Order / OrderItem 생성 (PENDING_PAYMENT)
  → OutboxEvent 저장 (BEFORE_COMMIT)
  → 대기열 토큰 삭제
  → Kafka 발행 (Scheduler, 1초 주기)
```

## PG 결제 연동

OpenFeign 클라이언트에 `@CircuitBreaker(name = "pgCircuitBreaker")` + `@Retry(name = "pgRetry")`를 적용합니다. PG 장애 시 CircuitBreaker가 Open 상태로 전환되어 빠른 실패를 반환하고, Payment 상태는 PENDING으로 유지됩니다. PaymentScheduler가 30초마다 PENDING 결제를 재조회하여 복구합니다.

## 쿠폰

| 발급 방식 | 흐름 |
|-----------|------|
| 동기 발급 | POST /api/v1/coupons/{couponId}/issue → 즉시 user_coupons 저장 |
| 비동기 발급 | POST /api/v1/coupons/{couponId}/issue-async → Kafka coupon-issue-requests 발행 → commerce-streamer 처리 |

쿠폰 타입: `FIXED`(고정 할인액) / `RATE`(비율 할인%).

## Transactional Outbox 패턴

트랜잭션 BEFORE_COMMIT 시점에 outbox_events 테이블에 이벤트를 저장합니다. 별도 Scheduler가 1초 주기로 outbox_events를 Kafka로 발행하여 정확히 한 번 전달을 보장합니다.

## 주요 주제

| 주제 | 위치 |
|------|------|
| 주문 생성·취소·OrderItem 스냅샷 | [주문-생성/](주문-생성/README.md) |
| PG 결제·CircuitBreaker·PaymentScheduler | [결제-흐름/](결제-흐름/README.md) |
| 쿠폰 정책·동기/비동기 발급 | [쿠폰-규칙/](쿠폰-규칙/README.md) |
| 주문 취소 기반 환불 흐름 | [환불-정책/](환불-정책/README.md) |

## 관련 ontology

`ontology/abox/commerce-order.yaml`
