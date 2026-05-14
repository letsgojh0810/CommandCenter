# 환불 정책

> 작성일: 2026-05-14 | 수정일: 2026-05-14 | 유형: 정책 | 관련 레포: letsgojh0810/commerce-backend

## 개요

별도의 환불 전용 API는 없습니다. 주문 취소(`POST /api/v1/orders/{orderId}/cancel`)가 곧 환불 흐름입니다.

## payment_cancel_requests 테이블

취소 요청을 추적하는 테이블입니다.

| 필드 | 설명 |
|------|------|
| `orderId` | 취소 대상 주문 ID |
| `pgTransactionId` | PG 거래 ID |
| `status` | 취소 처리 상태 |
| `reason` | 취소 사유 |

## 취소 흐름

`POST /api/v1/orders/{orderId}/cancel`

1. **재고 복구**: `product.increaseStock()`으로 차감된 재고를 원복합니다.
2. **쿠폰 복구**: 사용된 UserCoupon 상태를 `AVAILABLE`로 복원합니다.
3. **OrderCancelledEvent 발행**: OutboxEvent에 `OrderCancelledEvent`를 저장하고 Kafka로 발행합니다.

## 관련 entity

`payment_cancel_requests`, `order`, `outbox_events` — `ontology/abox/commerce-order.yaml` 참조
