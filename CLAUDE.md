# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 프로젝트 개요

React 19 + TypeScript + Vite로 만든 한국어 노트 앱. JSON Server를 목 백엔드로 사용하는 학습용 프로젝트.

## 명령어

```bash
npm run dev        # Vite(localhost:5173) + JSON Server(localhost:3001) 동시 실행
npm run build      # tsc 타입 검사 + vite 프로덕션 빌드
npm run lint       # ESLint 자동 수정
npm run format     # Prettier 전체 포맷
npm test           # Vitest 1회 실행
npm test:watch     # Vitest 감시 모드
npm run server     # JSON Server만 단독 실행
```

## 아키텍처

**상태 분리 원칙**: UI 선택 상태(`selectedNoteId`, `isCreating`)는 `App`이 소유, 데이터 상태(`notes`, `loading`, `error`)는 `NotesContext`가 소유. 두 레이어를 섞지 않는다.

**컴포넌트 트리**:
```
App (selectedNoteId, isCreating)
└── NotesProvider
    └── Layout
        ├── NoteList → NoteItem  (사이드바)
        └── NoteEditor           (생성/편집 폼)
```

**데이터 모델** (`src/types/note.ts`): `{ id, title, content, createdAt, updatedAt }`. `tags` 필드는 계획됐지만 미구현.

## 컴포넌트 구현 패턴

- **named export** 사용 — `export function NoteEditor(...)` (default export 금지)
- Props 타입은 컴포넌트 바로 위에 `interface ComponentNameProps`로 선언
- 이른 반환(early return)으로 loading → error → empty → 본문 순서로 처리

```tsx
// 패턴 예시
if (loading) return <p>로딩 중...</p>;
if (error)   return <p>오류: {error}</p>;
if (notes.length === 0) return <p>노트가 없습니다</p>;
return <>{/* 본문 */}</>;
```

- 컴포넌트에서 API를 직접 호출하지 않는다 — 반드시 `useNotes()` 훅을 통해 호출
- `NoteEditor`처럼 외부 상태로 폼을 동기화할 때는 `useEffect`를 사용하고, 의존성 경고는 `// eslint-disable-line react-hooks/exhaustive-deps`로 억제

## 상태 관리 패턴

- Context API: `NotesContext` (데이터) + `useNotes()` (소비 훅) 쌍으로 구성
- `useNotes()`는 Provider 외부에서 호출 시 즉시 throw — 범위 벗어난 사용을 방지
- **낙관적 업데이트**: API 성공 후 재요청 없이 로컬 상태를 직접 갱신

```ts
// 추가: setNotes(prev => [...prev, newNote])
// 수정: setNotes(prev => prev.map(n => n.id === id ? updated : n))
// 삭제: setNotes(prev => prev.filter(n => n.id !== id))
```

- `loading`/`error`는 Context에서 관리, 컴포넌트는 `saving` 같은 로컬 액션 상태만 관리

## API 호출 패턴

모든 HTTP 호출은 `src/api/notes.ts`에 집중. 컴포넌트나 Context에서 `fetch`를 직접 사용하지 않는다.

```ts
// 표준 패턴
const res = await fetch(`${API_URL}/notes`);
if (!res.ok) throw new Error('Failed to ...');
return res.json();
```

- 생성 입력 타입: `Omit<Note, 'id' | 'createdAt' | 'updatedAt'>`
- 수정 입력 타입: `Partial<Note>`
- 타임스탬프(`createdAt`, `updatedAt`)는 API 함수 내에서 ISO 문자열로 생성 — 컴포넌트가 관여하지 않음
- 에러 전파 방향: API 함수 throw → Context catch → 컴포넌트 catch + `console.error()`
- `alert()` 사용 금지 — 에러는 반드시 `console.error()`로만 처리

## 네이밍 규칙

| 대상 | 규칙 | 예시 |
|------|------|------|
| 컴포넌트 | PascalCase | `NoteEditor`, `NoteItem` |
| Props 타입 | `컴포넌트명Props` | `NoteEditorProps` |
| Context 타입 | `컨텍스트명Type` | `NotesContextType` |
| 컴포넌트 내 핸들러 | `handle` 접두사 | `handleSave`, `handleDone` |
| props 콜백 | `on` 접두사 | `onDone`, `onSelect`, `onDelete` |
| API 함수 / Context 액션 | 동사+명사 통일 | `createNote`, `updateNote`, `deleteNote`, `fetchNotes` |
| 불리언 상태 | `is`/`has` 접두사 | `isCreating`, `loading`, `saving` |

## 코드 컨벤션

- TypeScript strict 모드 — 암묵적 `any`, 미사용 변수/파라미터 금지
- Prettier: 작은따옴표, 들여쓰기 2칸, 줄 길이 100자, trailing comma 전체 적용
- 한국어 주석 정상 — 로직 설명에 한국어 사용
- Tailwind CSS v4 — 커스텀 테마 변수는 `src/index.css`에 정의 (`tailwind.config.js` 없음)
- 테스트: Vitest globals + jsdom + `@testing-library/react`, 설정 파일 `src/test-setup.ts`
