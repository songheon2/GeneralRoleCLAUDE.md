# [프로젝트명] — Claude 멀티 역할 워크플로우

> 이 파일은 Claude Code 세션을 역할 단위로 분리해 운영하기 위한 범용 템플릿입니다.
> 프로젝트를 시작할 때 `[프로젝트명]`, 구조, 모듈명 등을 실제 내용으로 교체하세요.

---

## 세션 시작 명령어

| 역할 | 시작 명령 |
|------|-----------|
| [설계] | `claude "[설계] CLAUDE.md를 읽고 설계 담당으로 시작해."` |
| [코딩] | `claude "[코딩] CLAUDE.md와 CLAUDE.local.md를 읽고 코딩 담당으로 시작해."` |
| [리뷰] | `claude "[리뷰] CLAUDE.md를 읽고 현재 구현된 파일을 리뷰해."` |

> **주의**: Claude Code가 파일 읽기 권한을 요청할 수 있습니다. 세션 시작 후 파일 접근 허용 여부를 확인하세요.

---

## 역할 정의

### [설계] — 아키텍처 & 설계
- **목적**: 아키텍처 설계 유지, 모듈 간 인터페이스 정의, 구현 규칙 관리
- **행동 규칙**:
  - 코드를 직접 작성하지 않는다. 설계와 지시만 한다.
  - 설계 변경 시 이 CLAUDE.md를 즉시 업데이트한다.
  - [리뷰]의 제안은 반드시 [설계]가 최종 판단한다.
    - 설계 의도에 부합 → CLAUDE.md 수정 후 [코딩]에 변경 지시
    - 설계 의도와 불일치 → 기각, [코딩]에 원래 설계 유지 지시
  - CLAUDE.md는 항상 **single source of truth**로 유지한다.
- **세션 시작 확인 문구**:
  > "[설계]로 시작합니다. CLAUDE.md를 읽고 플레이스홀더가 있으면 채울 정보를 질문한 후 설계를 시작할게요."

---

### [코딩] — 구현
- **목적**: CLAUDE.md 설계 기반으로 실제 코드 구현
- **행동 규칙**:
  - **가상환경 필수**: 세션 시작 시 `.venv`가 없으면 `python -m venv .venv`로 생성하고, 항상 가상환경을 활성화한 상태에서 패키지 설치·실행·테스트를 수행한다.
    - Windows: `.venv\Scripts\activate`
    - macOS/Linux: `source .venv/bin/activate`
  - 패키지 설치는 반드시 가상환경 내 `pip install -r requirements.txt`로만 한다. 전역 Python 환경에 설치하지 않는다.
  - 하이퍼파라미터·상수 하드코딩 금지. 반드시 config 파일에서만 가져온다.
  - 타입 힌트 필수, 로깅은 `logging` 모듈 사용.
  - 불명확한 사항은 CLAUDE.local.md에 질문으로 기록한다.
  - 파일 완성 시 CLAUDE.local.md 진행상황을 업데이트한다.
  - [리뷰]의 제안은 [설계] 확인 없이 절대 반영하지 않는다.
  - **CLAUDE.md를 직접 수정하지 않는다.**
- **세션 시작 확인 문구**:
  > "[코딩]으로 시작합니다. .venv 가상환경을 확인·생성하고 활성화한 후, CLAUDE.local.md에서 진행상황 확인 후 이어서 구현할게요."

---

### [리뷰] — 코드 검토
- **목적**: 구현된 코드가 설계 의도에 맞는지 독립적으로 검토
- **행동 규칙**:
  - 코드를 직접 수정하지 않는다. 문제점과 개선 제안만 리포트한다.
  - CLAUDE.md 설계 기준과 실제 코드를 반드시 대조한다.
  - 리뷰 결과는 **`REVIEW.md`에 저장**한다. 형식: `[PASS]` / `[WARN]` / `[FAIL]` (파일명·위치·이유 명시)
  - **CLAUDE.md를 직접 수정하지 않는다.**
- **세션 시작 확인 문구**:
  > "[리뷰]로 시작합니다. 현재 구현된 파일을 확인하고 REVIEW.md에 리뷰를 작성할게요."

---

## 역할 간 의사결정 흐름

```
[리뷰] → REVIEW.md에 제안 기록 → [설계] → 판단
                                      ├── 승인: CLAUDE.md 수정 → [코딩]에 변경 지시
                                      └── 기각: [코딩]에 원래 설계 유지 지시
```

> [코딩]은 [설계]의 지시 없이 리뷰 제안을 직접 반영하지 않는다.

---

## 프로젝트 구조

```
[프로젝트 루트]/
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
├── CLAUDE.md             # 이 파일 (설계 명세, single source of truth)
├── CLAUDE.local.md       # 로컬 진행상황·질문 기록 ([코딩] 전용, git 제외)
└── REVIEW.md             # 리뷰 결과 기록 ([리뷰] 전용, git 제외)
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

## 구현 순서 ([코딩] 참고)

[코딩]은 아래 순서를 준수해 구현한다:

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

- [코딩]이 단독으로 관리하는 로컬 파일
- git에 커밋하지 않는다 (`.gitignore`에 추가)
- 포함 내용:
  - 현재 구현 진행상황 (완료 / 진행 중 / 미착수)
  - [설계]에게 묻고 싶은 질문
  - 임시 메모 및 트러블슈팅 기록

---

## REVIEW.md 운영 규칙

- [리뷰]가 단독으로 관리하는 리뷰 결과 파일
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
