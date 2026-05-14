# Glossary — 주문/결제/쿠폰

> 작성일: 2026-05-14 | 수정일: 2026-05-14 | 유형: 정책 | 관련 레포: letsgojh0810/commerce-backend

## 패턴

**Transactional Outbox**
트랜잭션 BEFORE_COMMIT 시점에 outbox_events 테이블에 이벤트를 저장합니다. 별도 Scheduler가 1초 주기로 outbox_events를 읽어 Kafka로 발행합니다. 주문 생성과 이벤트 발행의 원자성을 보장하여 정확히 한 번 전달을 실현합니다.

**CircuitBreaker (pgCircuitBreaker)**
OpenFeign PG 클라이언트에 적용된 Resilience4j CircuitBreaker입니다. PG 장애가 누적되면 Open 상태로 전환되어 PG 호출을 차단하고 빠른 실패를 반환합니다. Payment 상태는 PENDING으로 유지되며 PaymentScheduler가 복구를 담당합니다.

## 주문 상태

| 상태 | 의미 |
|------|------|
| `PENDING_PAYMENT` | 주문 생성 완료, 결제 대기 중 |
| `PAID` | 결제 완료 |
| `CANCELLED` | 주문 취소 (재고·쿠폰 복구 완료) |

## 결제 상태

| 상태 | 의미 |
|------|------|
| `PENDING` | 결제 요청 중 또는 PG 장애로 대기 |
| `SUCCESS` | 결제 성공 |
| `FAILED` | 결제 실패 |

## 쿠폰 타입

| 타입 | 의미 |
|------|------|
| `FIXED` | 고정 할인액 (예: 5,000원 할인) |
| `RATE` | 비율 할인 (예: 10% 할인) |

**쿠폰 정책 필드**: `minOrderAmount`(최소 주문 금액), `validDays`(유효 기간 일수), `totalLimit`(전체 발급 한도)

## UserCoupon 상태

| 상태 | 의미 |
|------|------|
| `AVAILABLE` | 사용 가능 |
| `USED` | 사용 완료 |
| `EXPIRED` | 만료 |

## CardType

PG 결제 시 카드 타입: `SAMSUNG`, `KB`, `HYUNDAI`, `KAKAO`, `TOSS`, `NAVER`

## 기타

**OrderItem 스냅샷**: 주문 시점의 `productId`, `brandName`, `productName`, `price`, `quantity`를 order_items 테이블에 저장합니다. 이후 상품 정보가 변경되어도 주문 기록은 원본 값을 유지합니다.

**payment_cancel_requests**: 주문 취소 요청 추적 테이블. `orderId`, `pgTransactionId`, `status`, `reason` 필드를 포함합니다.
