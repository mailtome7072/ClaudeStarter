# /sprint-dev

sprint{n}.md 계획 문서를 기반으로 구현 단계에 진입합니다. 브랜치 생성, 구현 현황 파악, 가이드라인 적용을 자동으로 처리합니다.

> **실행 주체**: 이 커맨드는 **사용자가 직접** Claude Code에 입력하는 슬래시 커맨드입니다. 에이전트가 대신 실행하지 않으며, sprint-planner 완료 후 사용자가 직접 `/sprint-dev {n}`을 입력해야 합니다.

## 사용법

```
/sprint-dev [n]
```

- `n`: 스프린트 번호 (생략 시 현재 브랜치명에서 자동 추출)

## 실행 절차

### 1단계: 스프린트 번호 결정

`$ARGUMENTS`가 있으면 해당 번호를 사용합니다.
없으면 현재 브랜치명에서 숫자를 추출합니다 (예: `sprint3` → `n=3`).
여전히 불명확하면 `ROADMAP.md`의 `🔄 진행 중` 항목을 확인하거나 사용자에게 확인합니다.

### 2단계: 컨텍스트 로드

다음 파일을 순서대로 읽습니다:
1. `docs/sprint/sprint{n}.md` — 작업 목록, 완료 기준, 기술적 접근 방법
2. `ROADMAP.md` — 해당 스프린트 목표 및 Phase 위치 확인
3. `git log develop..HEAD --oneline` — 이미 완료된 커밋 확인 (재진입 시)
4. `git status` — 현재 작업 상태

### 3단계: 브랜치 확인 및 생성

현재 브랜치가 `sprint{n}`인지 확인합니다:

- `sprint{n}` 브랜치가 없으면 develop 기반으로 생성:
  ```bash
  git checkout develop
  git checkout -b sprint{n}
  ```
- `sprint{n}` 브랜치가 이미 있으면 전환:
  ```bash
  git checkout sprint{n}
  ```
- develop 브랜치가 최신인지 확인 후 브랜치 생성 (`git pull origin develop` 먼저)

### 4단계: 구현 현황 보고

사용자에게 다음 정보를 보고합니다:

```
Sprint {n} 구현 모드 진입

계획 문서: docs/sprint/sprint{n}.md
현재 브랜치: sprint{n}

작업 목록:
  ✅ (이미 완료된 커밋에서 추론한 항목)
  ⬜ (남은 작업 항목들)

완료 기준 (Definition of Done):
  - (sprint{n}.md의 완료 기준 항목들)
```

새 세션에서 재진입하는 경우 "마지막으로 완료된 작업" 정보를 명시합니다.

### 5단계: 구현 가이드라인 적용

코드 작성 시 다음을 자동 준수합니다:
- **karpathy-guidelines**: 파일 수정 전 반드시 읽기, git diff 확인, --no-verify 금지
- **백엔드 파일 접근 시**: `.claude/rules/backend.md` 자동 로드
- **프론트엔드 파일 접근 시**: `.claude/rules/frontend.md` 자동 로드
- **커밋**: 작업 단위별로 의미있는 커밋 메시지 (한국어)

### 완료 신호

구현이 완료되면 sprint-close agent를 사용합니다:

> "sprint{n} 구현 완료했어. sprint-close 실행해줘."

sprint-close 완료 후:

> "sprint-review 실행해줘."

## 중간 재진입 (새 세션에서 이어서 구현)

새 세션에서 이어서 구현할 때:

> "/sprint-dev {n}"

sprint{n}.md가 SSOT이므로 이 문서 하나만 읽으면 어디서든 진행 상황을 파악하고 이어서 구현할 수 있습니다.
