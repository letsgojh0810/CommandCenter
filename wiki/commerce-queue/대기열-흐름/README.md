# 대기열 흐름

> 작성일: 2026-05-14 | 수정일: 2026-05-14 | 유형: 정책 | 관련 레포: letsgojh0810/commerce-backend

## 전체 흐름

1. 사용자가 `POST /api/v1/queue/enter`를 호출합니다.
2. `queue:waiting` ZSet에 `score=타임스탬프`로 추가됩니다.
3. QueueScheduler가 100ms마다 실행되어 ZSet에서 `batchSize`만큼 팝합니다.
4. 팝된 사용자에게 두 개의 토큰을 발급합니다.
   - `queue:token:{userId}` (Soft TTL)
   - `queue:token:hard:{userId}` (Hard TTL)
5. 사용자가 `POST /api/v1/orders` 호출 시 `queue:token:{userId}` 존재를 검증합니다.
6. 주문 생성 완료 후 두 토큰을 모두 삭제합니다.

## Hard TTL + Soft TTL 이중 토큰 구조

Soft TTL 단독 사용 시 사용자가 활동을 유지하는 한 입장 권한이 무기한 연장될 수 있습니다. Hard TTL은 이를 방지하기 위해 발급 시점으로부터 고정된 만료 시간을 설정합니다.

| 토큰 | 역할 | 만료 조건 |
|------|------|-----------|
| `queue:token:{userId}` | 주문 생성 시 검증 대상 | Soft TTL 만료 또는 토큰 삭제 |
| `queue:token:hard:{userId}` | 입장 권한 최대 유지 시간 | Hard TTL 만료 시 자동 소멸 |

## queue:enabled 비활성화 시나리오

`queue:enabled` 플래그가 비활성 상태이면 주문 생성 시 대기열 토큰 검증을 건너뜁니다. 트래픽이 낮은 시간대나 점검 상황에서 대기열 없이 주문을 바로 처리할 수 있습니다.
