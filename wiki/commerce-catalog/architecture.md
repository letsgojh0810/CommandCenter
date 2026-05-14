# Architecture — 상품 카탈로그

> 정책 인덱스 + 데이터 흐름. 코드 구조/ERD는 ontology와 코드에 위임합니다.

## 데이터 흐름

### 상품 조회 (캐시 포함)

```mermaid
flowchart LR
    A[클라이언트] -->|GET /api/v1/products| B[ProductController]
    B --> C{Redis 캐시 확인}
    C -->|Hit| D[캐시 응답]
    C -->|Miss| E[(MySQL)]
    E --> F[Redis에 저장\nproductDetail: 10분\nproductList: 5분]
    F --> G[응답]
```

### 좋아요 이벤트 흐름

```mermaid
flowchart LR
    A[클라이언트] -->|POST /api/v1/products/:id/likes| B[LikeController]
    B --> C[likes 테이블 저장]
    C --> D[LikeCreatedEvent 발행]
    D -->|AFTER_COMMIT| E[product.likeCount 갱신\n낙관적 락 @Version]
    E --> F[Outbox 저장]
    F -->|Kafka| G[catalog-events 토픽]
    G --> H[메트릭·랭킹 도메인]
```

## 정책 인덱스

| 주제 | 책임 entity | 핵심 정책 |
|------|-------------|----------|
| [브랜드](브랜드/README.md) | `brand` | 브랜드 CRUD, 어드민 전용 생성/수정/삭제 |
| [상품 조회](상품-조회/README.md) | `product` | Redis 캐시, 정렬 옵션(LATEST/PRICE_ASC/LIKES_DESC), 캐시 무효화 |
| [좋아요](좋아요/README.md) | `like` | 낙관적 락, LikeCreatedEvent/LikeCancelledEvent, Kafka 발행 |

## 의존하는 인프라

| infra | 용도 |
|-------|------|
| `mysql` | 브랜드·상품·좋아요 마스터 |
| `redis` | 상품 상세(10분)·목록(5분) 캐시 |
| `kafka` | catalog-events 토픽 (메트릭·랭킹 도메인으로 이벤트 전달) |
