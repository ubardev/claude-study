# mermaid-diagram

`src/` 디렉토리를 분석해 Mermaid 기반 컴포넌트 의존성 및 상태 흐름 다이어그램을 `docs/architecture/index.html`로 생성하고 브라우저로 여는 커맨드.

## 사용법

```
/mermaid-diagram
```

인자 없이 호출하면 현재 워킹 디렉토리의 `src/`를 분석한다.  
`$ARGUMENTS`에 경로가 있으면 해당 디렉토리를 분석한다.

## 절차

**Step 1 — src/ 탐색**

`find src -type f` 로 모든 소스 파일 목록을 파악한 뒤, 각 파일을 Read 도구로 열람해 다음을 추출한다:

- import 관계 (어느 컴포넌트가 무엇을 import 하는가)
- props 타입과 callback props (onXxx)
- Context 소비 여부 (`useXxx` 훅 사용)
- 로컬 상태 (`useState`)와 그 역할
- API 호출 경로

**Step 2 — 다이어그램 설계**

추출한 정보를 바탕으로 세 가지 Mermaid 다이어그램을 설계한다:

1. **컴포넌트 트리** (`graph TD`): 렌더 포함 관계 + props/callback 레이블
2. **상태 흐름** (`graph LR`): UI 상태(App 소유) vs 데이터 상태(Context 소유) 분리, 상태가 어느 컴포넌트로 내려가는지 표시
3. **데이터 시퀀스** (`sequenceDiagram`): 사용자 액션 → 컴포넌트 → Context 액션 → API → 낙관적 업데이트 흐름

**Step 3 — HTML 생성**

아래 조건을 만족하는 `docs/architecture/index.html`을 Write 도구로 생성한다:

- Mermaid는 CDN(`https://cdn.jsdelivr.net/npm/mermaid/dist/mermaid.min.js`)으로 로드
- 탭 UI: 버튼 클릭으로 세 다이어그램 전환
- 다이어그램 제목과 간단한 설명 포함
- `docs/` 디렉토리가 없으면 먼저 `mkdir -p docs/architecture` 실행
- 스타일: 가독성 좋은 배경색, 충분한 여백, 모바일 대응 viewport

**Step 4 — 브라우저 열기**

```bash
open docs/architecture/index.html
```

**Step 5 — 완료 보고**

생성된 파일 경로와 포함된 다이어그램 목록을 사용자에게 알린다.

## 출력

- `docs/architecture/index.html` — 3개 Mermaid 다이어그램이 담긴 단일 HTML
- 브라우저 자동 실행 (macOS: `open` 명령어)

## 주의사항

- Mermaid 노드 ID에 한글이나 특수문자를 쓰면 파싱 오류가 난다. 노드 ID는 영문/숫자만 사용하고 레이블에 한국어를 쓴다.
- `graph` 방향: 컴포넌트 트리는 `TD`(위→아래), 상태 흐름은 `LR`(왼→오른쪽).
- 기존 `docs/architecture/index.html`이 있으면 덮어쓴다 (아키텍처 문서는 항상 최신 상태 유지).
