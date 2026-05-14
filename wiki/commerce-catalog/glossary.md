# Glossary — 상품 카탈로그

> 이 도메인에서 자주 쓰이는 용어를 풀어 정리합니다.

## 데이터/엔티티

**product (상품)**
카탈로그의 핵심 데이터 단위. `brandId`, `name`, `description`, `price`, `stock`, `imageUrl`, `likeCount` 필드를 가집니다. likeCount는 좋아요 이벤트가 발생할 때 낙관적 락으로 갱신됩니다.

**brand (브랜드)**
상품의 상위 분류 단위. 상품은 반드시 하나의 브랜드에 속합니다. 브랜드 생성·수정·삭제는 어드민 API로만 가능합니다.

**productDetail 캐시**
상품 상세 조회 결과를 저장하는 Redis 캐시. TTL은 10분입니다. 상품 정보가 변경되면 해당 캐시는 무효화됩니다.

**productList 캐시**
상품 목록 조회 결과를 저장하는 Redis 캐시. TTL은 5분입니다. 상품 정보가 변경되면 해당 캐시는 무효화됩니다.

**likeCount**
상품의 좋아요 수. 좋아요 생성/취소 이벤트(`LikeCreatedEvent`, `LikeCancelledEvent`)가 AFTER_COMMIT 후 product 엔티티의 likeCount를 갱신합니다. 동시 갱신 충돌은 낙관적 락(`@Version`)으로 방지합니다.

## 정렬 옵션

**LATEST**
상품을 최신 등록순으로 정렬합니다. 기본 정렬 옵션입니다.

**PRICE_ASC**
상품을 가격 낮은 순으로 정렬합니다.

**LIKES_DESC**
상품을 좋아요 수 많은 순으로 정렬합니다.
