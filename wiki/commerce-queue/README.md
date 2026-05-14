# 선착순 대기열 (commerce-queue)

> 작성일: 2026-05-14 | 수정일: 2026-05-14 | 유형: 정책 | 관련 레포: letsgojh0810/commerce-backend

## 개요

Redis ZSet 기반의 선착순 대기열 도메인입니다. 주문 생성 전 트래픽을 제어하여 서버 부하를 완충합니다. `queue:enabled` 플래그로 대기열 활성화 여부를 제어할 수 있습니다.

## API

| 엔드포인트 | 설명 |
|-----------|------|
| `POST /api/v1/queue/enter` | 대기열 입장 |
| `GET /api/v1/queue/position` | 현재 대기 순위 조회 |

## 동작 원리

1. 사용자가 `POST /api/v1/queue/enter`를 호출하면 Redis ZSet(`queue:waiting`)에 추가됩니다. score는 입장 타임스탬프입니다.
2. QueueScheduler가 100ms마다 실행되어 `batchSize`만큼 ZSet에서 사용자를 팝하고 입장 토큰을 발급합니다.
3. 토큰을 발급받은 사용자는 주문 생성(`POST /api/v1/orders`) 시 토큰 검증을 통과합니다.
4. 주문 생성 완료 후 토큰이 삭제됩니다.

## 토큰 구조

Hard TTL + Soft TTL 이중 토큰 구조로 운영합니다. 상세는 [대기열-흐름/](대기열-흐름/README.md)을 참조하세요.

## 주요 주제

| 주제 | 위치 |
|------|------|
| 전체 흐름·토큰 구조·비활성화 시나리오 | [대기열-흐름/](대기열-흐름/README.md) |

## 관련 ontology

`ontology/abox/commerce-queue.yaml`
