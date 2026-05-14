# Commerce Hub — Command Center

<!-- deployed-url:start -->
🌐 **Knowledge Graph**: https://letsgojh0810.github.io/commerce-hub/
<!-- deployed-url:end -->

커머스 서비스의 **도메인 지식·비즈니스 정책·코드 위치**를 한 곳에서 관리하는 AI 기반 팀 공용 작업 공간입니다. PO·디자이너·FE·BE 누구든 Claude Code를 통해 같은 방식으로 작업합니다.

---

## 왜 Command Center인가

코드베이스가 커질수록 "이 로직이 어디에 있지?", "이 정책은 왜 이렇게 돼 있지?"를 파악하는 비용이 늘어납니다. Command Center는 이 문제를 세 계층으로 나눠 해결합니다.

```
ontology  ←  "어디에 있나" (코드 위치·관계 지도)
   wiki   ←  "왜 이렇게 됐나" (의사결정·비즈니스 정책)
projects  ←  실제 코드 (격리된 작업 환경)
```

- **온톨로지가 지도다.** 도메인 개념·관계·코드 패키지 위치가 YAML로 명시됩니다. "주문 취소가 재고에 어떤 영향을 주나?"를 코드 전체를 뒤지지 않고 파악합니다.
- **Wiki가 맥락이다.** 코드에 드러나지 않는 의사결정 근거·정책·트레이드오프를 사람이 읽는 문서로 보존합니다.
- **Claude가 연결한다.** 질문이 들어오면 ontology → wiki → 코드 순으로 진입해 역할에 관계없이 일관된 답변을 만들어냅니다.

---

## 프로젝트 구성

| 레이어 | 내용 |
|--------|------|
| **지식** | `ontology/` (entity·관계 YAML) + `wiki/` (정책 문서 Markdown) |
| **시각화** | `site/` — D3.js 기반 React 앱. ontology/wiki를 그래프로 렌더링, GitHub Pages 자동 배포 |
| **코드** | `projects/commerce-backend/` (Kotlin · Spring Boot · Gradle 멀티모듈) |
|  | `projects/commerce-frontend/` (React · Vite · Tailwind CSS) |

### 온톨로지 도메인

| 도메인 | 설명 |
|--------|------|
| `commerce-catalog` | 상품 등록·분류·가격·검색 노출 |
| `commerce-order` | 주문 생성·결제·환불·쿠폰 |
| `commerce-inventory` | 재고 점유·확정·입출고·저재고 알림 |

---

## 주요 스킬

Claude Code에서 `/` 명령으로 작업 흐름을 제어합니다.

| 스킬 | 역할 |
|------|------|
| `/dev` | PRD → 설계 → 구현 → 리뷰 → PR 전 사이클 |
| `/new-domain` | 새 도메인 ontology + wiki 동시 생성 |
| `/domain-audit` | ontology ↔ wiki ↔ 코드 3자 정합성 점검 |
| `/commit` | 이슈 키 파싱·린트·민감 파일 감지 포함 커밋 |
| `/pull-request` | 커밋 히스토리 기반 PR 자동 생성 |
| `/worktree` | 격리된 feature 작업 환경 생성·전환·정리 |
| `/lens` | 코드 탐색·영향 분석·리포트 |
| `/agent-browser` | 브라우저 자동화 (QA·스크린샷·폼 테스트) |

---

## 작업 흐름

```
1. ontology/index.yaml 으로 도메인 구조 파악
2. 해당 abox/{도메인}.yaml 에서 entity·관계·코드 위치 확인
3. wiki/{도메인}/ 에서 비즈니스 정책·의사결정 배경 확인
4. projects/{레포}/worktrees/ 에서 코드 작업 (격리 환경)
5. /commit → /pull-request 로 변경사항 반영
```

ontology와 wiki는 항상 코드 변경과 함께 갱신됩니다. 코드만 바뀌고 지식 레이어가 낡아지는 상황을 `/domain-audit`으로 주기적으로 점검합니다.

---

## 로컬 시각화 실행

```bash
cd site && npm install && npm run build:data && npm run dev
```

`build:data`가 ontology/wiki를 읽어 JSON으로 변환한 뒤 Vite dev server가 그래프를 렌더링합니다. ontology/wiki 수정 후에는 `build:data`를 재실행하세요.
