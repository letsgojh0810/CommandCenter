# 주문 생성

> 작성일: 2026-05-14 | 수정일: 2026-05-14 | 유형: 정책 | 관련 레포: letsgojh0810/commerce-backend

## POST /api/v1/orders 흐름

주문 생성은 다음 단계를 순서대로 수행합니다.

1. **대기열 토큰 검증** (optional): Redis `queue:token:{userId}` 키 존재 여부를 확인합니다. 대기열이 활성화된 경우에만 검증합니다.
2. **재고 차감**: `product.decreaseStock()`을 호출하여 상품 재고를 차감합니다. 재고 부족 시 주문 생성을 중단합니다.
3. **쿠폰 검증·사용**: 쿠폰 유효성(유효 기간, 최소 주문 금액, totalLimit 등)을 검증하고 UserCoupon 상태를 `USED`로 전환합니다.
4. **Order / OrderItem 생성**: `orders` 테이블에 Order(상태 `PENDING_PAYMENT`)를, `order_items` 테이블에 OrderItem 스냅샷을 저장합니다.
5. **OutboxEvent 저장**: 트랜잭션 BEFORE_COMMIT 시점에 `outbox_events` 테이블에 이벤트를 기록합니다.
6. **대기열 토큰 삭제**: Redis에서 `queue:token:{userId}` 및 `queue:token:hard:{userId}` 키를 삭제합니다.

Scheduler가 1초 주기로 outbox_events를 읽어 Kafka order-events 토픽으로 발행합니다.

## 주문 취소

`POST /api/v1/orders/{orderId}/cancel`

주문 취소 시 다음 작업을 수행합니다.

- 재고 복구: `product.increaseStock()`
- 쿠폰 복구: UserCoupon 상태를 `AVAILABLE`로 복원
- OutboxEvent 저장: `OrderCancelledEvent`를 outbox_events에 기록 → Kafka 발행

## OrderItem 스냅샷

주문 시점의 상품 정보를 `order_items` 테이블에 고정 저장합니다.

| 필드 | 설명 |
|------|------|
| `productId` | 상품 식별자 |
| `brandName` | 브랜드명 (주문 시점 기준) |
| `productName` | 상품명 (주문 시점 기준) |
| `price` | 단가 (주문 시점 기준) |
| `quantity` | 수량 |

이후 상품 정보(가격, 이름 등)가 변경되어도 주문 기록은 원본 값을 유지합니다.

## 주문 상태 흐름

```
PENDING_PAYMENT
    ├── 결제 성공 → PAID
    └── 취소 → CANCELLED
```
