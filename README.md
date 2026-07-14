# GeneralRoleCLAUDE.md

Claude Code를 **[설계] 메인 에이전트 하나 + coding/review 서브에이전트** 구조로 운영하기 위한 범용 CLAUDE.md 템플릿입니다.
사용자는 [설계] 세션 하나만 직접 컨트롤하고, [설계]가 `Agent` tool로 coding·review 서브에이전트를 호출해 구현·검토를 지시합니다.

## 사용 방법

1. 이 레포를 clone합니다.
2. `GeneralRoleCLAUDE.md`를 새 프로젝트 루트에 `CLAUDE.md`로, `.claude/agents/`를 그대로 복사합니다.
3. `[설계]` 세션 하나만 시작합니다 — 플레이스홀더 채우기 질문부터 자동으로 진행되며, 이후 구현·검토는 [설계]가 내부적으로 서브에이전트를 호출해 처리합니다.

```bash
git clone <this-repo>
cp GeneralRoleCLAUDE.md /your/new/project/CLAUDE.md
cp -r .claude /your/new/project/.claude
cd /your/new/project
claude "[설계] CLAUDE.md를 읽고 메인 에이전트(오케스트레이터)로 시작해."
```

## 역할 구성

| 역할 | 목적 | 호출 주체 |
|------|------|-----------|
| [설계] | 아키텍처 설계, 모듈 인터페이스 정의, coding/review 서브에이전트 지시 | 사용자가 직접 대화 |
| `coding` 서브에이전트 (`.claude/agents/coding.md`) | CLAUDE.md 기반 실제 구현 | [설계]가 `Agent` tool로 호출 |
| `review` 서브에이전트 (`.claude/agents/review.md`) | 구현 코드와 설계 명세 대조 검토 | [설계]가 `Agent` tool로 호출 |

## 의사결정 흐름

```
사용자 ↔ [설계] (유일하게 직접 대화하는 세션)
              ├─ coding 서브에이전트 호출 → CLAUDE.local.md 갱신
              ├─ review 서브에이전트 호출 → REVIEW.md에 제안 기록
              └─ [설계]가 REVIEW.md 판단
                    ├── 승인: CLAUDE.md 수정 → coding 서브에이전트에 변경 지시
                    └── 기각: coding 서브에이전트에 원래 설계 유지 지시
```

coding 서브에이전트는 [설계]의 지시 없이 리뷰 제안을 직접 반영하지 않습니다.

## 파일 구성

| 파일 | 설명 |
|------|------|
| `CLAUDE.md` | 설계 명세 — single source of truth, [설계]만 수정 |
| `.claude/agents/coding.md` | coding 서브에이전트 정의, git 커밋 대상 |
| `.claude/agents/review.md` | review 서브에이전트 정의, git 커밋 대상 |
| `CLAUDE.local.md` | coding 서브에이전트 전용 진행상황·질문 기록, git 제외 |
| `REVIEW.md` | review 서브에이전트 전용 리뷰 결과, git 제외 |
