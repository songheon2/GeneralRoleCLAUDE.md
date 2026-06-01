# GeneralRoleCLAUDE.md

Claude Code 세션을 **설계 / 코딩 / 리뷰** 3개 역할로 분리해 운영하기 위한 범용 CLAUDE.md 템플릿입니다.

## 사용 방법

1. 이 레포를 clone합니다.
2. `GeneralRoleCLAUDE.md`를 새 프로젝트 루트에 `CLAUDE.md`로 복사합니다.
3. `[설계]` 세션을 시작합니다 — 플레이스홀더 채우기 질문부터 자동으로 진행됩니다.

```bash
git clone <this-repo>
cp GeneralRoleCLAUDE.md /your/new/project/CLAUDE.md
cd /your/new/project
claude "[설계] CLAUDE.md를 읽고 설계 담당으로 시작해."
```

## 역할 구성

| 역할 | 목적 | 시작 명령 |
|------|------|-----------|
| [설계] | 아키텍처 설계, 모듈 인터페이스 정의 | `claude "[설계] CLAUDE.md를 읽고 설계 담당으로 시작해."` |
| [코딩] | CLAUDE.md 기반 실제 구현 | `claude "[코딩] CLAUDE.md와 CLAUDE.local.md를 읽고 코딩 담당으로 시작해."` |
| [리뷰] | 구현 코드와 설계 명세 대조 검토 | `claude "[리뷰] CLAUDE.md를 읽고 현재 구현된 파일을 리뷰해."` |

## 의사결정 흐름

```
[리뷰] → REVIEW.md에 제안 기록 → [설계] → 판단
                                      ├── 승인: CLAUDE.md 수정 → [코딩]에 변경 지시
                                      └── 기각: [코딩]에 원래 설계 유지 지시
```

[코딩]은 [설계]의 지시 없이 리뷰 제안을 직접 반영하지 않습니다.

## 파일 구성

| 파일 | 설명 |
|------|------|
| `CLAUDE.md` | 설계 명세 — single source of truth, [설계]만 수정 |
| `CLAUDE.local.md` | [코딩] 전용 진행상황·질문 기록, git 제외 |
| `REVIEW.md` | [리뷰] 전용 리뷰 결과, git 제외 |
