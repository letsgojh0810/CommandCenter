# Architecture — 주문/결제/쿠폰

> 작성일: 2026-05-14 | 수정일: 2026-05-14 | 유형: 정책 | 관련 레포: letsgojh0810/commerce-backend

## 정책 인덱스

| 주제 | 핵심 정책 |
|------|----------|
| [주문 생성](주문-생성/README.md) | 대기열 토큰 검증, 재고 차감, 쿠폰 사용, OutboxEvent, 토큰 삭제 |
| [결제 흐름](결제-흐름/README.md) | PG 연동, CircuitBreaker, PaymentScheduler 30초 재조회 |
| [쿠폰 규칙](쿠폰-규칙/README.md) | FIXED/RATE 타입, 동기/비동기 발급, UserCoupon 상태 |
| [환불 정책](환불-정책/README.md) | 주문 취소 기반 환불, 재고·쿠폰 복구, payment_cancel_requests |

## 주문 생성 전체 흐름

```mermaid
sequenceDiagram
    participant Client
    participant OrderAPI
    participant Redis as Redis (queue:token)
    participant ProductDB as Product (stock)
    participant CouponDB as UserCoupon
    participant OrderDB as orders / order_items
    participant OutboxDB as outbox_events
    participant Scheduler
    participant Kafka

    Client->>OrderAPI: POST /api/v1/orders
    OrderAPI->>Redis: 대기열 토큰 검증 (optional)
    OrderAPI->>ProductDB: decreaseStock()
    OrderAPI->>CouponDB: 쿠폰 검증·사용 처리
    OrderAPI->>OrderDB: Order / OrderItem 생성 (PENDING_PAYMENT)
    OrderAPI->>OutboxDB: OutboxEvent 저장 (BEFORE_COMMIT)
    OrderAPI->>Redis: 대기열 토큰 삭제
    Scheduler->>OutboxDB: 1초마다 미발행 이벤트 조회
    Scheduler->>Kafka: order-events 발행
```

## 결제 흐름 (PG + CircuitBreaker)

```mermaid
sequenceDiagram
    participant Client
    participant PaymentAPI
    participant PG as PG (OpenFeign)
    participant CB as CircuitBreaker (pgCircuitBreaker)
    participant PaymentDB
    participant PaymentScheduler

    Client->>PaymentAPI: POST /api/v1/payments
    PaymentAPI->>CB: @CircuitBreaker + @Retry
    CB->>PG: PG 결제 요청
    alt 정상
        PG-->>PaymentAPI: 성공 응답
        PaymentAPI->>PaymentDB: Payment → SUCCESS
    else PG 장애 / CircuitBreaker Open
        CB-->>PaymentAPI: fallback
        PaymentAPI->>PaymentDB: Payment 상태 PENDING 유지
    end

    loop 30초마다
        PaymentScheduler->>PaymentDB: PENDING 결제 조회
        PaymentScheduler->>PG: PG 재조회
        PaymentScheduler->>PaymentDB: 결과 반영 (SUCCESS / FAILED)
    end

    Client->>PaymentAPI: POST /api/v1/payments/callback
    PaymentAPI->>PaymentDB: PG 콜백 수신 → 상태 갱신
```

## 정책 테이블

| 항목 | 정책 |
|------|------|
| 주문 생성 상태 초기값 | PENDING_PAYMENT |
| 결제 성공 상태 | PAID |
| 주문 취소 상태 | CANCELLED |
| PG 장애 시 동작 | Payment PENDING 유지 → PaymentScheduler 재조회 |
| Outbox 발행 주기 | 1초 |
| PaymentScheduler 주기 | 30초 |
| 쿠폰 타입 | FIXED (고정액), RATE (비율) |
| 비동기 쿠폰 발급 토픽 | coupon-issue-requests |
