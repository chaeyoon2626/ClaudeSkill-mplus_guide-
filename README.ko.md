# mplus-manual-guide

*[View in English →](README.md)*

Mplus 통계 소프트웨어의 **공식 User's Guide(v8, 전체 20개 챕터)** 에 기반해서, Mplus 문법(`.inp`)을 안전하게 작성하도록 도와주는 Claude Skill입니다.

일반 LLM에게 Mplus 코드를 물어보면 그럴듯하지만 실제로는 존재하지 않는 옵션이나 잘못된 TECH 번호를 만들어내는 경우가 있습니다. 이 스킬은 매뉴얼에 실제로 있는 내용만 사용하고, 확인되지 않은 내용은 "모른다"고 답하도록 설계되었습니다.

## 이 스킬이 하는 일

- **매뉴얼 기반**: `references/` 안의 79개 파일이 공식 Mplus User's Guide 20개 챕터의 모든 예제(Ex 1.x ~ 20.x)를 다룹니다. 회귀분석, 매개/조절효과, EFA, CFA/SEM, LCA/LPA(mixture), 성장모형, 다층모형(multilevel), 결측치/베이지안 분석, 몬테카를로 시뮬레이션까지 포함됩니다.
- **모델 확인 후 코드 작성**: 사용자가 분석하고 싶은 모델을 설명하면, 코드를 바로 주지 않고 먼저 **사각형(관측변수)/원(잠재변수)/화살표**로 이루어진 SEM 스타일 경로도(path diagram)를 그려서 "이 모델이 맞나요?"라고 확인부터 합니다. 확인 전에는 절대 코드를 생성하지 않습니다.
- **최소 코드 원칙**: 확정된 모델을 표현하는 데 구조적으로 꼭 필요한 옵션만 포함합니다. `ESTIMATOR`, `BOOTSTRAP`, TECH 옵션 등은 사용자가 명시적으로 요청하기 전에는 절대 먼저 추가하지 않습니다.
- **점진적 옵션 추가**: "몇 개 계층이 적절한지 어디서 봐?", "정규분포 적합도 확인하고 싶어" 같은 일상 언어 요청도 정확한 Mplus 옵션(TECH11, TECH14 등)으로 번역해서 기존 코드에 추가해줍니다.
- **결측치 처리는 항상 물어봄**: 모든 분석에서 결측치 코딩 방식(-999, 999 등)을 표준 체크리스트 항목으로 항상 확인합니다.
- **변수명 규칙**: Mplus는 8자 이하의 영문/숫자 변수명만 허용하므로, 사용자가 "이직의도", "학업성취도" 같은 한글/긴 이름으로 변수를 설명해도 코드에는 짧은 영문 축약어(`turnover`, `achieve` 등)를 쓰고, 매핑을 코드 옆에 표시합니다.

## 이 스킬이 하지 않는 일

- Mplus 문법과 무관한 일반 통계 이론 설명
- 매뉴얼에 없는 옵션을 그럴듯하게 지어내는 것 (모르면 모른다고 말합니다)
- 다른 통계 소프트웨어(R, SPSS, lavaan 등) 문법

## 저작권 관련 안내

이 스킬의 `references/*.md` 파일들은 공식 Mplus User's Guide를 읽고 **직접 새로 작성한 요약/가이드**입니다. 매뉴얼 원문이나 PDF, 그림을 그대로 복사하거나 재배포하지 않습니다. Mplus 자체는 Muthén & Muthén 사의 상용 소프트웨어이며, 이 저장소는 해당 회사와 무관한 개인 프로젝트입니다. 실제 분석에 사용하기 전에는 반드시 [공식 매뉴얼](https://www.statmodel.com/html_ug.shtml)과 실제 Mplus 실행 결과로 교차 검증하시기 바랍니다.

## 저장소 구성

```
mplus-manual-guide/
├── README.md              # 영어 버전
├── README.ko.md            # 이 문서 (한국어)
├── mplus-skill.skill       # 바로 업로드 가능한 패키지 파일 (실제로는 zip)
└── mplus-skill/             # 압축 풀린 원본 폴더 (Claude Code용)
    ├── SKILL.md
    └── references/
        ├── index.md         # 20개 챕터 전체 커버리지 인덱스
        └── (79개의 절차별/문법별 참조 파일)
```

## 설치 방법

`.skill` 파일은 확장자만 다를 뿐 **일반 zip 파일과 동일한 형식**입니다. 어떤 방식으로 쓰든 별도의 압축 해제 없이 zip 그대로 쓰거나(claude.ai / Desktop / Cowork), 압축을 풀어서 폴더째로 배치(Claude Code)하면 됩니다.

### 1) claude.ai / Claude Desktop / Cowork에서 쓰는 경우

1. GitHub 저장소에서 `mplus-skill.skill` 파일을 다운로드합니다. (저장소 전체를 "Code → Download ZIP"으로 받아도 되고, 이 파일 하나만 받아도 됩니다.)
2. Claude 앱에서 **설정(Settings) → Capabilities**에서 "코드 실행(Code execution)"이 꺼져 있다면 켭니다. (조직/팀 계정이라면 관리자가 조직 설정에서 Skills 기능을 먼저 켜야 할 수 있습니다.)
3. **Customize → Skills**로 이동해 **"+" → "Create skill" → "Upload a skill"**을 선택합니다.
4. 다운로드한 `mplus-skill.skill` 파일을 그대로 업로드합니다. (확장자 때문에 업로드가 안 된다면 파일명을 `mplus-skill.zip`으로 바꿔서 올리면 됩니다 — 내용물은 동일합니다.)
5. 업로드 후 스킬을 켜면(toggle on) 바로 사용할 수 있습니다.

### 2) Claude Code에서 쓰는 경우

1. `mplus-skill.skill` (또는 저장소 전체)을 다운로드해 압축을 풉니다. `mplus-skill/` 폴더 안에 `SKILL.md`가 바로 보이는지 확인하세요.
2. 이 폴더를 원하는 위치에 복사합니다.
   - **개인용(모든 프로젝트에서 사용)**: `~/.claude/skills/mplus-manual-guide/`
   - **프로젝트 전용**: `<프로젝트 루트>/.claude/skills/mplus-manual-guide/`
   
   즉, 압축을 푼 `mplus-skill` 폴더의 내용물(`SKILL.md`, `references/`)을 폴더명이 `mplus-manual-guide`인 디렉토리 아래로 옮기면 됩니다:
   ```bash
   mkdir -p ~/.claude/skills/mplus-manual-guide
   cp -r mplus-skill/* ~/.claude/skills/mplus-manual-guide/
   ```
3. Claude Code를 재시작하거나 새 세션을 시작하면 자동으로 인식됩니다. `/mplus-manual-guide`로 직접 호출하거나, "Mplus로 조절된 매개효과 분석하고 싶어" 같은 질문을 하면 자동으로 트리거됩니다.

## 사용 예시

아래는 실제 대화가 어떻게 진행되는지 보여주는 3가지 시나리오입니다. (실제로는 영어로 응답하도록 만들어져 있지만, 한국어로 질문해도 잘 이해합니다.)

### 예시 1 — 조절된 매개효과 회귀분석 (moderated mediation)

> **사용자**: 스트레스가 수면의 질을 통해 우울에 영향을 주는데, 이 매개효과가 사회적지지 수준에 따라 달라지는지 보고 싶어.

**스킬의 진행 순서**
1. 변수 유형 확인 (연속형/범주형), 결측치 코딩 방식 확인
2. 경로도 확인: `stress`(사각형) → `sleep`(사각형) → `depress`(사각형), `support`가 `stress→sleep` 경로를 조절(interaction term)
3. "이 모델이 맞나요?" 확인 후에만 최소 코드 제공:

```
VARIABLE:
  NAMES ARE stress sleep depress support;
  MISSING ARE ALL (-999);

DEFINE:
  stsup = stress*support;

MODEL:
  sleep ON stress support stsup;
  depress ON sleep stress;

MODEL INDIRECT:
  depress IND sleep stress;
```

4. 후속 요청 예: "부트스트랩 신뢰구간도 보고 싶어" → `ANALYSIS: BOOTSTRAP = 1000;` + `OUTPUT: CINTERVAL (BOOTSTRAP);` 만 추가해서 다시 보여줌 (다른 옵션은 임의로 추가하지 않음)

### 예시 2 — 확인적 요인분석 (CFA)

> **사용자**: 우울 척도 6문항이 하나의 요인으로 묶이는지 확인하고 싶어. 문항은 리커트 5점 척도야.

**스킬의 진행 순서**
1. 문항 6개가 순서형(categorical)인지 확인, 결측치 코딩 확인
2. 경로도 확인: 원(`dep`) → 사각형 6개(`d1`~`d6`), 각 문항에 잔차 화살표
3. 확인 후 최소 코드:

```
VARIABLE:
  NAMES ARE d1-d6;
  CATEGORICAL ARE d1-d6;
  MISSING ARE ALL (-999);

MODEL:
  dep BY d1-d6;
```

4. 후속 요청 예: "적합도 지수 확인하고 싶어" → `OUTPUT: STDYX;` 추가 / "수정지수도 보고 싶어" → `OUTPUT: MODINDICES;` 추가

### 예시 3 — 다층모형 / 잠재프로파일분석 (LPA + multilevel)

> **사용자**: 학생들이 학교에 nested 되어 있는 데이터인데, 학생 수준에서 학습동기 프로파일(잠재계층)을 나누고 싶어. 학교 간 차이도 고려해야 해.

**스킬의 진행 순서**
1. 군집변수(학교 ID), 몇 개의 지표 변수로 프로파일을 나눌지, 결측치 확인
2. 경로도 확인: `WITHIN`/`BETWEEN`을 점선으로 구분, Within 수준에 원(`c`, 잠재계층) → 지표 변수 사각형들, Between 수준에 학교 군집 표시
3. 확인 후 최소 코드:

```
VARIABLE:
  NAMES ARE school m1-m4;
  CLASSES = c(3);
  CLUSTER = school;
  MISSING ARE ALL (-999);

ANALYSIS:
  TYPE = TWOLEVEL MIXTURE;

MODEL:
  %WITHIN%
  %OVERALL%
  c ON m1-m4;
```

(참고: 프로파일 지표를 잠재계층 지표로 쓸지, 공변량으로 쓸지에 따라 실제 `MODEL` 구문은 달라지며, 스킬이 대화 중 이 부분을 먼저 확인합니다.)

4. 후속 요청 예: "계층 수 몇 개가 적절한지 어떻게 봐?" → 매뉴얼 근거로 `TECH11`, `TECH14` (LMR/BLRT 검정)를 설명하고 코드에 추가

## 한계 및 주의사항

- 이 스킬은 **매뉴얼에 실제로 있는 문법**만 다루므로, Mplus 최신 버전에서 추가된 기능이나 이 저장소 이후에 나온 매뉴얼 개정판 내용은 반영되어 있지 않을 수 있습니다. (그런 경우 스킬이 자체적으로 statmodel.com을 실시간 조회하도록 설계되어 있습니다.)
- 통계적 판단(어떤 모형이 이론적으로 타당한지, 결과를 어떻게 해석할지)은 사용자의 몫입니다. 이 스킬은 "문법을 정확하게 쓰도록" 돕는 도구이지, 통계 자문 서비스가 아닙니다.
- 생성된 코드는 실제 Mplus에서 실행하기 전에 반드시 검토하시기 바랍니다.

## 라이선스

이 저장소(README, SKILL.md, references/*.md 등 직접 작성한 콘텐츠)는 [MIT License](https://opensource.org/licenses/MIT)로 배포됩니다. Mplus 소프트웨어 자체 및 공식 User's Guide의 저작권은 Muthén & Muthén 사에 있습니다.
