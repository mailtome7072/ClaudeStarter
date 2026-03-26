# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 목적
이 문서는 AI 협업 도구(Claude 등)가 프로젝트 문서를 작성·검수할 때 따라야 할 지침을 정의한다.
사람 팀원은 README.md, PRD.md, ROADMAP.md, CHANGELOG.md를 참고하고, AI는 CLAUDE.md를 참고하여 일관된 산출물을 생성한다.

## 아키텍처 개요

> 디렉토리 구조 상세는 `ARCHITECTURE.md` 참조. 아래에는 AI 협업에 필요한 핵심 흐름만 기술.

이 저장소는 **Claude Code 협업 스타터 템플릿**이다. 실제 앱 코드는 스프린트 진행 중에 추가된다.

**핵심 흐름**: PRD.md → ROADMAP.md → sprint{n} 브랜치 → develop PR → main 배포
**에이전트 역할**: `prd-to-roadmap`(PRD→ROADMAP) → `sprint-planner`(계획) → `sprint-close`(PR+검증) → `deploy-prod`(프로덕션) / `hotfix-close`(긴급패치)

> **에이전트 메모리**: `.claude/agents/agent-memory/`는 에이전트별 서브디렉토리로 구성된다.
> ```
> .claude/agents/agent-memory/
> ├── sprint-planner/MEMORY.md   # 스프린트 번호·Velocity 누적
> ├── prd-to-roadmap/MEMORY.md   # PRD→ROADMAP 변환 이력
> └── deploy-prod/MEMORY.md      # 배포 이력
> ```
> 변경 시 반드시 git commit하여 팀 전체와 동기화한다.

## 빌드 및 테스트 명령어

### 사전 요구사항
- Node.js v20 이상, Python 3.12 이상, Docker Desktop
- 신규 참여자 온보딩: `docs/setup-guide.md` 참조

### 초기 환경 설정

> **신규 클론 후 필수**: `ARCHITECTURE.md`의 5개 변수(`project_name`, `github_org` 등)를 채운 뒤 `/setup-project` 실행.
> (미실행 시 `deploy.yml`, `CLAUDE.md`의 저장소 URL이 플레이스홀더 상태로 남음)

```bash
./SETUP.sh           # Node.js 확인, pnpm 설치, Python venv 생성, .env 복사
git checkout -b develop  # 최초 1회: develop 브랜치 생성 (스프린트 기반 브랜치)
```

### 프론트엔드 (pnpm)

> `app/frontend/`에 코드가 생성된 후 사용 가능하다. (첫 스프린트 이후)

```bash
pnpm install         # 의존성 설치
pnpm build           # 빌드
pnpm test            # 전체 테스트
pnpm test -- --testPathPattern=<파일명>  # 단일 테스트 파일 실행
pnpm lint            # 린트
pnpm lint --fix      # 자동 수정 가능한 린트 오류 수정
```

### 백엔드 (Python/pytest)

> `app/backend/`에 코드가 생성된 후 사용 가능하다. (첫 스프린트 이후)
> 로컬 테스트 실행 전 **PostgreSQL 16**과 **Redis 7**이 필요하다. (`docker compose up`으로 서비스 기동)
> 필요한 환경변수: `POSTGRES_HOST`, `POSTGRES_PORT`, `POSTGRES_DB`, `POSTGRES_USER`, `POSTGRES_PASSWORD`, `REDIS_HOST`, `REDIS_PORT`, `JWT_SECRET`, `SECRET_KEY`

```bash
source .venv/bin/activate
pip install -r app/backend/requirements.txt
pytest app/backend/tests/                        # 전체 테스트
pytest app/backend/tests/test_foo.py             # 단일 테스트 파일
pytest app/backend/tests/test_foo.py::test_bar   # 단일 테스트 함수
```

### Docker (스테이징 검증)
```bash
docker compose up --build         # 로컬 스테이징 실행 (docker-compose.yml 사용)
docker compose -f docker-compose.prod.yml up   # 프로덕션 설정으로 실행 (서버 배포 시 사용)
```

## 저장소
- **원격 저장소**: https://github.com/${github_org}/${github_repo}.git _(⚠️ `/setup-project` 실행 후 실제 URL로 치환됨)_
- **브랜치 전략**: strategy/branch-strategy.md 참고
- **문서 저장 위치**: docs/ 하위 폴더 (sprint, test-reports, deploy-history 등) — 구조 상세는 `docs/README.md` 참조

## 슬래시 커맨드

| 커맨드 | 구분 | 설명 |
|--------|------|------|
| `/setup-project` | 프로젝트 커스텀 | `ARCHITECTURE.md` 변수 → `README.md`, `CLAUDE.md`, `deploy.yml`, `PRD.md` 플레이스홀더 일괄 치환 |
| `/restart` | 프로젝트 커스텀 | Docker Compose 서비스 재시작 |

### 내장 스킬 (`.claude/skills/`)

| 스킬 | 용도 |
|------|------|
| `karpathy-guidelines` | 코드 작성·수정 시 적용 원칙 |
| `writing-plans` | 계획 문서 작성 형식·INVEST 기준 정의 (sprint-planner agent가 주로 참조하며, 직접 호출도 가능) |
| `code-review` | PR 코드 리뷰 체크리스트 |
| `test-checklist` | 테스트 보고서 작성 형식 |
| `retrospective` | 스프린트 회고 진행 형식 |

## 환경 변수 관리 지시
- `.env` 파일은 프로젝트 루트에 위치하며, 각자 환경에서 작성한다.
- `.env` 파일은 절대 Git에 커밋하지 않는다 (`.gitignore`에 포함).
- `.env.example` 파일을 제공하여 필요한 변수 이름과 형식을 안내한다.
- 민감한 값(API 키, DB 접속 정보 등)은 사람이 직접 채운다.

## AI 협업 개발 원칙 (Karpathy)

> 상세 원칙은 `.claude/skills/karpathy-guidelines.md` 참조.

- 파일 수정 전 반드시 읽고 현재 상태를 파악한다.
- AI 생성 코드도 커밋 전 `git diff`로 의도치 않은 변경을 직접 확인한다.
- CI 실패 시 원인을 파악하고 수정한다 — `--no-verify` 우회 금지.
- 복잡한 작업은 추상화 계층 단위로 분해하여 요청한다 (DB 설계 + API + 프론트를 한 번에 요청 금지).
- AI 생성 코드도 `code-review` 스킬의 "AI 생성 코드 리뷰 추가 체크" 항목을 통과해야 한다.

## 언어 및 커뮤니케이션 규칙
- 기본 응답 언어: 한국어
- 코드 주석: 한국어로 작성
- 커밋 메시지: 한국어로 작성
- 문서화: 한국어로 작성
- 변수명/함수명: 영어 (코드 표준 준수)

## CI/CD 정책

> 파이프라인 기술 상세 (명령어, YAML 예시, 이미지 태그 규칙 등)는 [`docs/ci-policy.md`](docs/ci-policy.md) 참조.

모든 브랜치 전략은 `karpathy-guidelines` 스킬을 준수한다.

### Main 브랜치
- `develop` → `main` merge는 QA 통과 후 deploy-prod agent를 사용한다.
- merge 시 GitHub Actions가 GHCR 이미지 빌드 → 서버 SSH 배포를 자동 수행한다.
- 📎 배포 절차: `docs/dev-process.md` 섹션 6.2 / 롤백: 섹션 6.4 / Notion 업데이트: 섹션 8.5

**필수 GitHub Secrets** (Settings > Secrets and variables > Actions):
- `CR_PAT`: GHCR push용 PAT (패키지 write 권한 포함)
- `LIGHTSAIL_HOST`: 프로덕션 서버 IP 또는 도메인
- `LIGHTSAIL_USER`: SSH 접속 사용자명
- `LIGHTSAIL_SSH_KEY`: SSH 개인키 (PEM 형식)

### Develop 브랜치
- `sprint{n}` → `develop` PR은 sprint-close agent가 생성한다.
- `develop` merge 후 로컬 Docker로 스테이징 검증한다. (`docker compose up --build`)
- GHCR push는 하지 않으며, 프로덕션 배포는 `main` merge 시에만 수행한다.
- 📎 검증 매트릭스: `docs/dev-process.md` 섹션 5 (Sprint 컬럼) / 스테이징 절차: 섹션 6.1

### Hotfix 브랜치
> Hotfix 추천 기준 SSOT: [`docs/dev-process.md`](docs/dev-process.md) 섹션 2
> 요건: 파일 3개 이하, 코드 50줄 이하, DB 변경 없음, 새 의존성 없음

- `main` 기반으로 `hotfix/{설명}` 브랜치를 생성한다.
- sprint-planner agent는 사용하지 않는다.
- 구현 완료 후 hotfix-close agent를 사용하여 마무리한다 (PR to main, 경량 검증, DEPLOY.md 기록, develop 역머지 안내).
- 프로덕션 배포는 main merge 시 GitHub Actions가 자동 수행한다.
- 📎 검증 매트릭스: `docs/dev-process.md` 섹션 5 (Hotfix 컬럼) / 롤백: 섹션 6.4

## Bash 명령 실행 규칙

- Bash 명령 실행 시 `cd /path &&` 접두사를 사용하지 마세요. 작업 디렉토리가 이미 프로젝트 루트로 설정되어 있습니다.
- 특히 git 명령은 반드시 `git ...` 형태로 직접 실행하세요. (`cd ... && git ...` 금지)
- `.claude/settings.json`의 기본 허용 명령은 `git *`만 포함됩니다. 개발 중 pnpm, pytest, docker 등 명령이 필요하면 `/update-config` 스킬로 권한을 추가하거나 `.claude/settings.json`의 `permissions.allow`를 직접 수정하세요.

## 개발시 유의해야할 사항

1. **plan 모드에서 수정사항을 받으면 반드시 Hotfix vs Sprint 의사결정을 먼저 수행한다.**
  - 수정사항의 긴급도, 변경 범위, DB 변경 여부, 의존성 추가 여부를 분석.
  - 기준 SSOT: `docs/dev-process.md` 섹션 2. Sprint 추천 기준 요약: 새 기능·여러 모듈·DB 변경·새 의존성·파일 4개 이상·코드 50줄 초과 중 하나라도 해당 시.
  - 사용자의 최종 결정을 받은 후 해당 프로세스를 따른다.

2. sprint 관련 문서 구조:
  - 스프린트 계획/완료 문서: `docs/sprint/sprint{n}.md`
  - 스프린트 첨부 파일 (스크린샷, 보고서 등): `docs/sprint/sprint{n}/`

3. sprint 개발이 plan 모드로 진행될 때는 다음을 꼭 준수합니다.
  - karpathy-guidelines skill을 준수한다.
  - sprint 가 새로 시작될 때는 `develop` 기반으로 `sprint{n}` 브랜치를 생성하고 해당 브랜치에서 작업한다. (worktree 사용하지 말아주세요)
    (`git checkout develop && git checkout -b sprint{n}`)
  - 다음과 같이 agent를 활용한다.
    - sprint-planner agent가 계획 수립 작업을 수행하도록 한다.
    - 구현/검증 단계에서는 각 task의 내용에 따라 적절한 agent가 있는지 확인 한 후 적극 활용한다.
    - 스프린트 구현이 완료되면 sprint-close agent를 사용하여 마무리 작업을 수행한다.
  - CI/배포 상세 절차는 위 CI/CD 정책을 참조한다.

4. hotfix 개발이 plan 모드로 진행될 때는 다음을 꼭 준수한다.
  - karpathy-guidelines skill을 준수한다.
  - `main` 기반으로 `hotfix/{설명}` 브랜치를 생성한다. (worktree 사용하지 말아주세요)
  - CI/배포 상세 절차는 위 CI/CD 정책 > Hotfix 브랜치를 참조한다.
  - 배포 후 실서버 검증이 필요하면 deploy-prod agent의 5단계(실서버 자동 검증)를 참조한다.

5. 검증 매트릭스 상세: `docs/dev-process.md` 섹션 5 참조
6. 배포 후 수동 작업: `DEPLOY.md` 참조 — 배포마다 리셋되는 일회성 체크리스트. 완료 기록은 `docs/deploy-history/`에 아카이브.
7. 체크리스트 작성 형식:
  - 완료 항목: `- ✅ 항목 내용`
  - 미완료 항목: `- ⬜ 항목 내용`
  - GFM `[x]`/`[ ]` 대신 이모지를 사용하여 마크다운 미리보기에서 시각적 구분을 보장한다.
  - 진행 상태 (ROADMAP.md 등): `📋 예정` → `🔄 진행 중` → `✅ 완료` / `⏸️ 보류`

## 문제 해결 참조

- **CI 실패** (pytest, pnpm test, pnpm lint): `docs/dev-process.md` 섹션 9.1
- **Docker 빌드 실패**: `docs/dev-process.md` 섹션 9.2
- **develop 브랜치 충돌**: `docs/dev-process.md` 섹션 9.3
- **잘못된 브랜치에서 작업 시작** (sprint → develop 기반 재생성 등): `docs/dev-process.md` 섹션 9.4

## 워크플로우 지침

각 워크플로우별 상세 포맷은 `strategy/` 하위 문서를 참조한다.

| 워크플로우 | 입력 | 출력 위치 | 전략 문서 |
|-----------|------|-----------|-----------|
| PRD → ROADMAP | PRD.md | ROADMAP.md | strategy/planning.md |
| Sprint Planning | ROADMAP.md | docs/sprint/sprint{n}.md | strategy/planning.md |
| Sprint Retrospective | - | docs/sprint-retrospectives/ | strategy/retrospectives.md / .claude/skills/retrospective.md |
| Test Report | - | docs/test-reports/ | strategy/testing.md / .claude/skills/test-checklist.md |
| Risk Register | - | docs/risk-register/ | strategy/risk-management.md |
| Deployment Log | - | docs/deploy-history/ | strategy/deployment.md |
| Code Review | - | docs/code-review-checklist.md | strategy/code-review.md / .claude/skills/code-review.md |
| CHANGELOG | - | CHANGELOG.md | - |

**CHANGELOG 버전 표기**: `## [x.y.z] - YYYY-MM-DD` / 카테고리: Added / Changed / Fixed / Removed / 최신 버전은 최상단에 추가

모든 산출물은 Markdown 형식, 한국어로 작성하며 문서 간 연결 관계(PRD → ROADMAP → Sprint → Retrospective → Deployment → CHANGELOG)를 유지한다.
