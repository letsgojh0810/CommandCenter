# Status — 주문/결제/쿠폰

> 작성일: 2026-05-14 | 수정일: 2026-05-14 | 유형: 정책 | 관련 레포: letsgojh0810/commerce-backend

## 주문

| AC | 상태 |
|----|------|
| 주문 생성 API (POST /api/v1/orders) | ✅ |
| 주문 취소 API (POST /api/v1/orders/{orderId}/cancel) | ✅ |

## 결제

| AC | 상태 |
|----|------|
| 결제 요청 API (POST /api/v1/payments) | ✅ |
| PG 콜백 수신 API (POST /api/v1/payments/callback) | ✅ |

## 쿠폰

| AC | 상태 |
|----|------|
| 쿠폰 동기 발급 API (POST /api/v1/coupons/{couponId}/issue) | ✅ |
| 쿠폰 비동기 발급 API (POST /api/v1/coupons/{couponId}/issue-async) | ✅ |

## 인프라/패턴

| AC | 상태 |
|----|------|
| Transactional Outbox 구현 | ✅ |
| PaymentScheduler (30초마다 PG 재조회) | ✅ |
