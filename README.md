# GeneralRoleCLAUDE.md

Claude Code를 **[설계] 메인 에이전트 하나** 중심으로 운영하기 위한 범용 CLAUDE.md 템플릿입니다.
사용자는 [설계] 세션 하나만 직접 컨트롤합니다. **평소 구현·테스트는 [설계]가 직접 수행**하며,
중요한 마일스톤(핵심 파이프라인 완성 직후, 되돌리기 어렵거나 비용이 큰 작업 직전, 최종 제출/배포
전 등)에서만 독립적인 검토를 위해 `Agent` tool로 review 서브에이전트를 호출합니다.

(과거엔 모든 구현 단계마다 coding/review 서브에이전트에 위임하는 구조였으나, 서브에이전트를 매번
호출하면 매번 컨텍스트를 새로 시작해야 해서 `CLAUDE.md`/`CLAUDE.local.md` 전체를 반복
재컨텍스트화하는 토큰 비용이 크고, 컨텍스트 단절로 중복 작업·혼선이 생기기 쉬웠음. 실제 완성도는
컨텍스트 분리 자체가 아니라 테스트·실측 검증·명세 대조 같은 "관행"에서 나온다는 것이 실전에서
확인되어, 직접 구현 + 마일스톤 리뷰 구조로 전환함.)

## 사용 방법

1. 이 레포를 clone합니다.
2. `GeneralRoleCLAUDE.md`를 새 프로젝트 루트에 `CLAUDE.md`로, `.claude/agents/`를 그대로 복사합니다.
3. `[설계]` 세션 하나만 시작합니다 — 플레이스홀더 채우기 질문부터 자동으로 진행되며, 이후 구현·테스트는 [설계]가 직접 진행하고 마일스톤마다 review 서브에이전트를 호출합니다.

```bash
git clone <this-repo>
cp GeneralRoleCLAUDE.md /your/new/project/CLAUDE.md
cp -r .claude /your/new/project/.claude
cd /your/new/project
claude "[설계] CLAUDE.md를 읽고 메인 에이전트로 시작해."
```

## 역할 구성

| 역할 | 목적 | 호출 주체 |
|------|------|-----------|
| [설계] | 아키텍처 설계, 모듈 인터페이스 정의, **직접 구현·테스트·실행**, 마일스톤 시점 독립 리뷰 호출·판단 | 사용자가 직접 대화 |
| `review` 서브에이전트 (`.claude/agents/review.md`) | 구현 코드와 설계 명세 대조 검토 (마일스톤 시점에만) | [설계]가 `Agent` tool로 호출 |
| `coding` 서브에이전트 (`.claude/agents/coding.md`) | 선택적, 기본 경로 아님 — 명확히 병렬화 가능한 예외적 상황에서만 사용 | [설계]가 드물게 `Agent` tool로 호출 |

## 의사결정 흐름

```
사용자 ↔ [설계] (유일하게 직접 대화하는 세션)
              ├─ (평소) [설계]가 직접 구현 + 테스트/스모크 테스트로 검증, CLAUDE.local.md에 기록
              ├─ (마일스톤 시점만) review 서브에이전트 호출 → REVIEW.md에 독립 검토 결과 기록
              └─ [설계]가 REVIEW.md 판단
                    ├── 승인: [설계]가 직접 CLAUDE.md/코드 수정
                    └── 기각: 판단 근거를 CLAUDE.local.md 또는 설계 변경 이력에 기록하고 기존 구현 유지
```

review 서브에이전트는 [설계]가 부른 마일스톤에서만 등장하며, 그 외 대부분의 작업은 [설계] 혼자 구현·검증합니다.

## 파일 구성

| 파일 | 설명 |
|------|------|
| `CLAUDE.md` | 설계 명세 — single source of truth, [설계]만 수정 |
| `.claude/agents/review.md` | review 서브에이전트 정의, git 커밋 대상 |
| `.claude/agents/coding.md` | coding 서브에이전트 정의(예외적 사용을 위한 참고용), git 커밋 대상 |
| `CLAUDE.local.md` | [설계]가 직접 작업하며 기록하는 진행상황·질문, git 제외 |
| `REVIEW.md` | review 서브에이전트 전용 리뷰 결과(마일스톤 시점에만 갱신), git 제외 |
