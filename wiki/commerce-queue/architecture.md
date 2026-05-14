# Architecture — 선착순 대기열

> 작성일: 2026-05-14 | 수정일: 2026-05-14 | 유형: 정책 | 관련 레포: letsgojh0810/commerce-backend

## 전체 흐름

```mermaid
sequenceDiagram
    participant User
    participant QueueAPI
    participant Redis as Redis (ZSet)
    participant Scheduler as QueueScheduler (100ms)
    participant OrderAPI

    User->>QueueAPI: POST /api/v1/queue/enter
    QueueAPI->>Redis: ZADD queue:waiting score=timestamp userId
    loop 100ms마다
        Scheduler->>Redis: ZPOPMIN queue:waiting batchSize
        Scheduler->>Redis: SET queue:token:{userId} (Soft TTL)
        Scheduler->>Redis: SET queue:token:hard:{userId} (Hard TTL)
    end
    User->>QueueAPI: GET /api/v1/queue/position
    QueueAPI->>Redis: ZRANK queue:waiting userId
    User->>OrderAPI: POST /api/v1/orders (토큰 포함)
    OrderAPI->>Redis: queue:token:{userId} 검증
    OrderAPI->>Redis: queue:token:{userId} / queue:token:hard:{userId} 삭제
```

## Hard TTL + Soft TTL 이중 토큰 구조

| 키 | 역할 |
|----|------|
| `queue:token:{userId}` | Soft TTL 토큰. 주문 생성 시 검증 대상. |
| `queue:token:hard:{userId}` | Hard TTL 토큰. 만료 시 자동 무효화. Soft TTL 연장 상한선 역할. |

Soft TTL은 사용자가 활동 중일 때 갱신될 수 있으나, Hard TTL이 만료되면 무조건 입장 권한이 소멸됩니다. 이를 통해 입장 권한을 무한정 유지하는 것을 방지합니다.
