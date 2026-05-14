# Glossary — 선착순 대기열

> 작성일: 2026-05-14 | 수정일: 2026-05-14 | 유형: 정책 | 관련 레포: letsgojh0810/commerce-backend

## Redis 키

**queue:waiting**
대기 중인 사용자 집합을 저장하는 Redis ZSet. score는 입장 타임스탬프(epoch millis)로, 선착순 처리를 보장합니다.

**queue:token:{userId}**
입장 허용된 사용자에게 발급되는 Soft TTL 토큰. 주문 생성 시 이 키의 존재 여부로 입장 권한을 검증합니다.

**queue:token:hard:{userId}**
입장 허용 시 함께 발급되는 Hard TTL 토큰. 만료되면 자동으로 입장 권한이 소멸됩니다. Soft TTL의 연장 상한선 역할을 합니다.

**queue:enabled**
대기열 활성화 여부를 저장하는 Redis 플래그. 이 값이 비활성 상태이면 대기열 검증을 건너뜁니다.

## 기타

**batchSize**
QueueScheduler 1회 실행 시 `queue:waiting` ZSet에서 팝하여 입장을 허용하는 사용자 수.

**QueueScheduler**
100ms 주기로 실행되어 `queue:waiting`에서 `batchSize`만큼 사용자를 꺼내 입장 토큰을 발급하는 스케줄러.
