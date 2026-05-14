# Architecture — 사용자

> 정책 인덱스 + 데이터 흐름. 코드 구조/ERD는 ontology와 코드에 위임합니다.

## 데이터 흐름

### 회원가입 흐름

```mermaid
flowchart LR
    A[클라이언트] -->|POST /api/v1/users| B[UserController]
    B --> C{loginId 중복 확인}
    C -->|중복| D[400 에러 반환]
    C -->|사용 가능| E{비밀번호 정책 검증}
    E -->|위반| F[400 에러 반환]
    E -->|통과| G[BCrypt 인코딩]
    G --> H[(users DB)]
    H --> I[201 Created]
```

### 보호 API 인증 흐름

```mermaid
flowchart LR
    A[클라이언트] -->|헤더: X-Loopers-LoginId\nX-Loopers-LoginPw| B[AuthFilter]
    B --> C{users DB 조회}
    C -->|미존재| D[401 Unauthorized]
    C -->|존재| E{BCrypt 비밀번호 비교}
    E -->|불일치| D
    E -->|일치| F[보호 API 처리]
    F --> G[응답]
```

## 정책 인덱스

| 주제 | 책임 entity | 핵심 정책 |
|------|-------------|----------|
| [회원가입·인증](회원가입-인증/README.md) | `user` | loginId 중복 불가, 비밀번호 정책, BCrypt 인코딩, 헤더 기반 인증 |
