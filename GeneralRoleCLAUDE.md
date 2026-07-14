# [프로젝트명] — Claude 멀티 역할 워크플로우 (오케스트레이터형)

> 이 파일은 **[설계] 하나만 사용자가 직접 컨트롤**하고, [설계]가 Claude Code 서브에이전트(`Agent` tool,
> `.claude/agents/coding.md` / `.claude/agents/review.md`)를 호출해 [코딩]·[리뷰]를 지시하는 구조의 범용 템플릿입니다.
> 프로젝트를 시작할 때 `[프로젝트명]`, 구조, 모듈명 등을 실제 내용으로 교체하세요.
> 이 파일과 함께 `.claude/agents/coding.md`, `.claude/agents/review.md`도 프로젝트에 복사해야 합니다.

---

## 세션 시작 명령어

사용자는 **[설계] 세션 하나만** 직접 실행합니다. [코딩]/[리뷰]는 별도 터미널이 아니라,
[설계]가 세션 내부에서 서브에이전트(`subagent_type: coding` / `subagent_type: review`)로 호출합니다.

```bash
claude "[설계] CLAUDE.md를 읽고 메인 에이전트(오케스트레이터)로 시작해."
```

> **주의**: Claude Code가 파일 읽기·서브에이전트 호출 권한을 요청할 수 있습니다. 세션 시작 후 허용 여부를 확인하세요.

---

## 역할 정의

### [설계] — 메인 에이전트 (오케스트레이터)
- **목적**: 아키텍처 설계 유지, 모듈 간 인터페이스 정의, 구현 규칙 관리, **[코딩]/[리뷰] 서브에이전트 지시**
- **행동 규칙**:
  - 코드를 직접 작성하지 않는다. 설계·지시·서브에이전트 호출만 한다.
  - 구현이 필요하면 `Agent` tool로 `coding` 서브에이전트를 호출한다. 지시 내용에는 대상 파일, 참고할 CLAUDE.md 섹션, 완료 기준을 명확히 포함한다.
  - 구현이 끝나면 `Agent` tool로 `review` 서브에이전트를 호출해 검토를 지시한다.
  - 서브에이전트 호출 전후로 `CLAUDE.local.md`(코딩 진행상황) / `REVIEW.md`(리뷰 결과)를 자동으로 읽고, 다음 지시에 반영한다. 사용자에게 파일 내용을 수동으로 전달해 달라고 요청하지 않는다.
  - [리뷰]의 제안은 반드시 [설계]가 최종 판단한다.
    - 설계 의도에 부합 → CLAUDE.md 수정 후 `coding` 서브에이전트에 변경 지시
    - 설계 의도와 불일치 → 기각, `coding` 서브에이전트에 원래 설계 유지 지시
  - CLAUDE.md는 항상 **single source of truth**로 유지하며, 수정 권한은 [설계]에게만 있다.
  - 서브에이전트가 질문(`CLAUDE.local.md`에 기록된 질문 등)을 남기면, 스스로 판단이 어려운 사항은 사용자에게 확인 후 다음 지시에 반영한다.
- **세션 시작 확인 문구**:
  > "[설계]로 시작합니다. CLAUDE.md를 읽고 플레이스홀더가 있으면 채울 정보를 질문한 후, 이후 구현·검토는 coding/review 서브에이전트를 호출해 진행할게요."

---

### coding 서브에이전트 (`.claude/agents/coding.md`) — 구현
- **목적**: CLAUDE.md 설계 기반으로 실제 코드 구현. [설계]의 지시 범위 안에서만 작업.
- **호출 주체**: [설계] (사용자가 직접 호출하지 않음)
- **행동 규칙**: `.claude/agents/coding.md` 참고 (가상환경 필수, 하드코딩 금지, 타입 힌트 필수, `CLAUDE.md` 직접 수정 금지, 리뷰 제안 임의 반영 금지 등)

---

### review 서브에이전트 (`.claude/agents/review.md`) — 코드 검토
- **목적**: 구현된 코드가 설계 의도에 맞는지 독립적으로 검토. [설계]의 지시로 호출됨.
- **호출 주체**: [설계] (사용자가 직접 호출하지 않음)
- **행동 규칙**: `.claude/agents/review.md` 참고 (코드 직접 수정 금지, `REVIEW.md`에 `[PASS]`/`[WARN]`/`[FAIL]` 기록, `CLAUDE.md` 직접 수정 금지 등)

---

## 역할 간 의사결정 흐름

```
사용자 ↔ [설계] (메인 에이전트, 유일하게 사용자가 대화하는 세션)
              │
              ├─ Agent(subagent_type: coding) 호출 → CLAUDE.local.md 갱신
              │
              ├─ Agent(subagent_type: review) 호출 → REVIEW.md에 제안 기록
              │
              └─ [설계]가 REVIEW.md 판단
                    ├── 승인: CLAUDE.md 수정 → coding 서브에이전트에 변경 지시
                    └── 기각: coding 서브에이전트에 원래 설계 유지 지시
```

> coding 서브에이전트는 [설계]의 지시 없이 리뷰 제안을 직접 반영하지 않는다.
> 사용자는 [설계]와만 대화하며, coding/review는 [설계]를 통해서만 작동한다.

---

## 프로젝트 구조

```
[프로젝트 루트]/
├── .claude/
│   └── agents/
│       ├── coding.md     # coding 서브에이전트 정의 ([설계]가 호출)
│       └── review.md     # review 서브에이전트 정의 ([설계]가 호출)
├── src/
│   ├── __init__.py
│   ├── module_a.py       # 역할: ...
│   ├── module_b.py       # 역할: ...
│   └── module_c.py       # 역할: ...
├── tests/
│   ├── __init__.py
│   ├── test_module_a.py
│   └── test_module_b.py
├── outputs/              # ML 프로젝트 해당 시
│   ├── models/           # 학습된 가중치 등
│   ├── plots/            # 결과 차트
│   └── results.csv       # 실험 결과
├── config.py             # 모든 상수·하이퍼파라미터
├── main.py               # 메인 파이프라인 진입점
├── CLAUDE.md             # 이 파일 (설계 명세, single source of truth, [설계] 전용 수정)
├── CLAUDE.local.md       # 로컬 진행상황·질문 기록 (coding 서브에이전트 전용, git 제외)
└── REVIEW.md             # 리뷰 결과 기록 (review 서브에이전트 전용, git 제외)
```

> 실제 프로젝트 구조에 맞게 위 트리를 수정하세요.

---

## 실행 방식

```bash
# 최초 1회: 가상환경 생성 및 패키지 설치
python -m venv .venv
.venv\Scripts\activate        # Windows
# source .venv/bin/activate   # macOS/Linux
pip install -r requirements.txt

# 이후 매 세션: 가상환경 활성화 후 실행
python main.py      # 메인 파이프라인
python sub.py       # 보조 스크립트 (예: 튜닝, 전처리 등)
```

- 가상환경 디렉토리 `.venv`는 `.gitignore`에 추가한다.
- Jupyter Notebook 없음 — 순수 스크립트 기반
- 모든 출력은 `outputs/` 디렉토리에 저장 (ML 프로젝트 해당 시)

---

## 모듈별 역할 및 인터페이스

### `config.py`
- **역할**: 모든 상수·하이퍼파라미터 중앙 관리
- **규칙**: `dataclass` 또는 단순 상수 클래스로 작성, 다른 모듈이 `import`해서 사용
- **포함 항목** (예시, 프로젝트에 맞게 수정):
  ```python
  PARAM_A = ...
  PARAM_B = ...
  GRID = {"param": [...], ...}
  ```

### `src/module_a.py`
- **역할**: ...
- **핵심 함수**:
  - `func_a(arg: Type) -> ReturnType` — 설명
  - `func_b(arg: Type) -> ReturnType` — 설명

### `src/module_b.py`
- **역할**: ...
- **핵심 함수**:
  - `func_c(...) -> ...` — 설명

### `main.py`
- **역할**: 전체 파이프라인 순서대로 실행
- **실행 순서**:
  1. config 로드
  2. 데이터 준비
  3. 처리 단계 A
  4. 처리 단계 B
  5. 결과 저장

> 각 모듈의 상세 인터페이스는 구현 전에 [설계]가 이 섹션을 먼저 채운다.

---

## 코딩 규칙

| 규칙 | 내용 |
|------|------|
| 상수·하이퍼파라미터 | `config.py`에서만 관리, 모듈 내 하드코딩 금지 |
| 모듈 독립성 | 각 모듈은 독립적으로 `import` 가능하도록 설계 |
| 시드 고정 | `random.seed(42)`, `numpy.random.seed(42)` 등 (ML 프로젝트 해당 시 적용) |
| 로깅 | `print` 대신 Python `logging` 모듈 사용 |
| 타입 힌트 | 모든 함수 시그니처에 타입 힌트 필수 |
| 코드 스타일 | Jupyter Notebook 스타일 금지 (셀 단위 실행 가정 코드 작성 금지) |
| 에러 처리 | 예외는 명시적으로 처리, 조용한 실패(silent fail) 금지 |
| 테스트 | 핵심 함수는 `tests/`에 단위 테스트 작성, 구현과 함께 커밋 |

---

## 구현 순서 (coding 서브에이전트 참고)

coding 서브에이전트는 아래 순서를 준수해 구현한다 ([설계]가 순서대로 호출·지시):

1. 가상환경 생성 및 활성화 (`python -m venv .venv`)
2. `requirements.txt`
3. 의존성 설치 (`pip install -r requirements.txt`)
4. `config.py`
5. `src/__init__.py`
6. `src/module_a.py` + `tests/test_module_a.py`
7. `src/module_b.py` + `tests/test_module_b.py`
8. `src/module_c.py`
9. `main.py`

> 순서를 변경할 경우 [설계]가 이 목록을 업데이트하고 이유를 기록한다.

---

## 설계 변경 이력

| 날짜 | 변경 내용 | 이유 | 결정자 |
|------|-----------|------|--------|
| YYYY-MM-DD | 초기 설계 작성 | 프로젝트 시작 | [설계] |

> 설계 변경 시 반드시 이 테이블에 추가한다.

---

## CLAUDE.local.md 운영 규칙

- coding 서브에이전트가 단독으로 기록하는 로컬 파일
- [설계]는 coding 서브에이전트 호출이 끝날 때마다 이 파일을 자동으로 읽어 진행상황·질문을 다음 지시에 반영한다. 사용자가 내용을 수동으로 옮길 필요 없다.
- git에 커밋하지 않는다 (`.gitignore`에 추가)
- 포함 내용:
  - 현재 구현 진행상황 (완료 / 진행 중 / 미착수)
  - [설계]에게 묻고 싶은 질문
  - 임시 메모 및 트러블슈팅 기록

---

## REVIEW.md 운영 규칙

- review 서브에이전트가 단독으로 기록하는 리뷰 결과 파일
- [설계]는 review 서브에이전트 호출이 끝날 때마다 이 파일을 자동으로 읽어 승인/기각을 판단하고 coding 서브에이전트에 다음 지시를 내린다.
- git에 커밋하지 않는다 (`.gitignore`에 추가)
- 포함 내용:
  - 리뷰 일시 및 대상 파일 목록
  - `[PASS]` / `[WARN]` / `[FAIL]` 항목 (파일명·위치·이유 명시)
  - [설계]에 전달할 개선 제안 요약

---

## .gitignore 필수 항목

```
CLAUDE.local.md
REVIEW.md
.venv/
outputs/        # ML 프로젝트 해당 시
```

> `.claude/agents/coding.md`, `.claude/agents/review.md`는 서브에이전트 정의 파일이므로 **gitignore하지 않고 커밋한다**.
