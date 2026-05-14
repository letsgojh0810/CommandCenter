# 결제 흐름

> 작성일: 2026-05-14 | 수정일: 2026-05-14 | 유형: 정책 | 관련 레포: letsgojh0810/commerce-backend

## API

| 엔드포인트 | 설명 |
|-----------|------|
| `POST /api/v1/payments` | 결제 요청 |
| `POST /api/v1/payments/callback` | PG 콜백 수신 |

## PG 연동

OpenFeign 클라이언트를 사용하며, Resilience4j를 통해 다음 어노테이션을 적용합니다.

- `@CircuitBreaker(name = "pgCircuitBreaker")`: PG 장애 누적 시 Open 상태로 전환하여 빠른 실패 반환
- `@Retry(name = "pgRetry")`: 일시적 오류에 대한 자동 재시도

## Payment 상태 흐름

```
PENDING
  ├── PG 성공 → SUCCESS
  └── PG 실패 / CircuitBreaker Open → PENDING 유지
                                          └── PaymentScheduler(30초) → SUCCESS / FAILED
```

## 장애 시나리오

CircuitBreaker가 Open 상태일 때 PG 호출이 차단됩니다. Payment는 `PENDING` 상태를 유지하며, PaymentScheduler가 30초마다 PENDING 결제를 조회하여 PG에 재확인하고 상태를 갱신합니다.

## CardType

PG 결제 시 지원 카드 타입: `SAMSUNG`, `KB`, `HYUNDAI`, `KAKAO`, `TOSS`, `NAVER`
