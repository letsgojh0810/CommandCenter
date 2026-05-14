# 쿠폰 규칙

> 작성일: 2026-05-14 | 수정일: 2026-05-14 | 유형: 정책 | 관련 레포: letsgojh0810/commerce-backend

## 쿠폰 정책

| 타입 | 설명 |
|------|------|
| `FIXED` | 고정 할인액 (예: 5,000원 할인) |
| `RATE` | 비율 할인 (예: 10% 할인) |

주요 정책 필드: `minOrderAmount`(최소 주문 금액), `validDays`(발급 후 유효 기간 일수), `totalLimit`(전체 발급 한도)

## 동기 발급

`POST /api/v1/coupons/{couponId}/issue`

요청 즉시 `user_coupons` 테이블에 저장합니다. 발급 가능 여부(totalLimit, 중복 발급 여부 등)를 검증한 후 UserCoupon을 생성합니다.

## 비동기 발급

`POST /api/v1/coupons/{couponId}/issue-async`

Kafka `coupon-issue-requests` 토픽으로 발급 요청을 발행합니다. commerce-streamer가 컨슈머로 동작하여 실제 발급을 처리합니다.

발급 상태 조회: `GET /api/v1/coupons/issue-requests/{requestId}`

## 내 쿠폰 목록

`GET /api/v1/coupons/users/me/coupons`

로그인 사용자의 UserCoupon 목록을 반환합니다.

## UserCoupon 상태 흐름

```
AVAILABLE → USED      (주문 생성 시 쿠폰 사용)
AVAILABLE → EXPIRED   (유효 기간 만료)
USED → AVAILABLE      (주문 취소 시 쿠폰 복구)
```
