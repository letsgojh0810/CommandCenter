# Commerce Hub Wiki

Commerce Hub 프로젝트의 비즈니스 정책 문서 인덱스입니다.

## 도메인 목록

| 도메인 | 설명 | 문서 |
|--------|------|------|
| `commerce-user` | 회원 가입·인증·프로필 | [README](commerce-user/README.md) |
| `commerce-catalog` | 브랜드·상품·좋아요 | [README](commerce-catalog/README.md) · [architecture](commerce-catalog/architecture.md) |
| `commerce-order` | 주문·결제·쿠폰 | [README](commerce-order/README.md) · [architecture](commerce-order/architecture.md) |
| `commerce-queue` | 선착순 대기열 | [README](commerce-queue/README.md) · [architecture](commerce-queue/architecture.md) |
| `commerce-ranking` | 랭킹·메트릭 | [README](commerce-ranking/README.md) · [architecture](commerce-ranking/architecture.md) |

## 공통 문서

- [공통 용어 사전](glossary.md)
- [온톨로지 설계 배경](ontology-design.md)

## 탐색 원칙

도메인 관련 정보를 찾을 때는 **ontology → wiki → 코드** 순서로 탐색합니다.

1. `ontology/index.yaml`에서 도메인 맵을 파악합니다.
2. 관련 A-Box 파일에서 entity/relation을 확인합니다.
3. entity의 `wiki_doc` 필드를 따라 이 wiki로 진입합니다.
4. 코드 위치는 entity의 `repo` + `package` 필드를 참조합니다.
