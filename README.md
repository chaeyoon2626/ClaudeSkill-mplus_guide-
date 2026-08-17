# mplus-manual-guide

*[한국어 버전 보기 →](README.ko.md)*

A Claude Skill that helps you write correct Mplus syntax (`.inp` files), grounded strictly in the **official Mplus User's Guide (v8, all 20 chapters)**.

Ask a general-purpose LLM for Mplus code and it will sometimes invent plausible-sounding options or TECH numbers that don't actually exist. This skill only uses content that's actually in the manual, and says "I don't know" when something isn't there rather than guessing.

## What this skill does

- **Manual-grounded**: 79 files under `references/` cover every worked example (Ex 1.x through 20.x) across all 20 chapters of the official Mplus User's Guide — regression, mediation/moderation, EFA, CFA/SEM, LCA/LPA (mixture models), growth modeling, multilevel modeling, missing data/Bayesian analysis, and Monte Carlo simulation.
- **Confirms the model before writing code**: when you describe the analysis you want, the skill doesn't jump straight to code. It first draws a simple SEM-style path diagram (rectangles = observed variables, circles = latent variables, arrows for paths) and asks "Is this what you meant?" It never generates code before you confirm the diagram.
- **Minimal base code**: only the options that are structurally required to express the confirmed model go into the code. `ESTIMATOR`, `BOOTSTRAP`, TECH options, etc. are never added on the skill's own initiative — only when you explicitly ask for them.
- **Translates plain-language follow-ups**: "Where do I see how many classes is right?" or "I want to check if this fits a normal distribution" get translated into the correct Mplus option (e.g. `TECH11`, `TECH14`) and added to the existing code.
- **Always asks about missing data**: every analysis gets a standard check for how missing values are coded (`-999`, `999`, etc.) — not just sometimes.
- **Variable naming rule**: Mplus only allows variable names up to 8 characters, Latin alphanumeric. If you describe a variable in plain language (e.g. "turnover intention", "academic achievement"), the skill uses a short English abbreviation in the code (`turnover`, `achieve`, etc.) and shows the mapping next to the code.

## What this skill doesn't do

- General statistics theory unrelated to Mplus syntax
- Inventing options that sound plausible but aren't in the manual (it says so honestly instead)
- Syntax for other statistical software (R, SPSS, lavaan, etc.)

## A note on copyright

The `references/*.md` files in this skill are **original summaries written by reading** the official Mplus User's Guide — they do not copy or redistribute the manual's own text, figures, or PDFs. Mplus itself is commercial software from Muthén & Muthén; this repository is an independent personal project, not affiliated with them. Before using this for real analysis, please cross-check against the [official manual](https://www.statmodel.com/html_ug.shtml) and actual Mplus output.

## Repository layout

```
mplus-manual-guide/
├── README.md                # this file (English)
├── README.ko.md              # Korean version
├── mplus-skill.skill          # ready-to-upload package (this is just a zip)
└── mplus-skill/                # the unpacked source folder (for Claude Code)
    ├── SKILL.md
    └── references/
        ├── index.md             # full 20-chapter coverage index
        └── (79 procedure/syntax reference files)
```

## Installation

A `.skill` file is just a **regular zip file with a different extension**. You can either use the zip as-is (claude.ai / Desktop / Cowork), or unzip it and drop the folder in place (Claude Code) — no repackaging needed either way.

### 1) claude.ai / Claude Desktop / Cowork

1. Download `mplus-skill.skill` from this repository (or download the whole repo as a ZIP via "Code → Download ZIP" — either works).
2. In the Claude app, go to **Settings → Capabilities** and turn on "Code execution" if it's off. (On a Team/Enterprise account, an admin may need to enable Skills in organization settings first.)
3. Go to **Customize → Skills**, click **"+" → "Create skill" → "Upload a skill"**.
4. Upload `mplus-skill.skill` as-is. (If the upload dialog rejects the `.skill` extension, just rename the file to `mplus-skill.zip` — the contents are identical.)
5. Toggle the skill on after upload, and it's ready to use.

### 2) Claude Code

1. Download `mplus-skill.skill` (or the whole repo) and unzip it. Confirm that `SKILL.md` sits directly inside the `mplus-skill/` folder.
2. Copy that folder to one of these locations:
   - **Personal (available in every project)**: `~/.claude/skills/mplus-manual-guide/`
   - **Project-only**: `<project root>/.claude/skills/mplus-manual-guide/`

   In other words, move the contents of the unzipped `mplus-skill` folder (`SKILL.md`, `references/`) into a directory named `mplus-manual-guide`:
   ```bash
   mkdir -p ~/.claude/skills/mplus-manual-guide
   cp -r mplus-skill/* ~/.claude/skills/mplus-manual-guide/
   ```
3. Restart Claude Code or start a new session and it will pick it up automatically. Invoke it directly with `/mplus-manual-guide`, or just ask something like "I want to run a moderated mediation analysis in Mplus" and it triggers automatically.

## Usage examples

Three scenarios showing how a real conversation plays out. (The skill is written to respond in English, but understands questions in Korean or other languages just fine.)

### Example 1 — Moderated mediation regression

> **User**: Stress affects depression through sleep quality, and I want to see whether this mediation effect depends on the level of social support.

**What the skill does**
1. Confirms variable types (continuous/categorical) and how missing data is coded
2. Confirms the path diagram: `stress` (rectangle) → `sleep` (rectangle) → `depress` (rectangle), with `support` moderating the `stress → sleep` path (interaction term)
3. Only after you confirm "yes, that's right" does it give the minimal code:

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

4. Example follow-up: "I also want a bootstrap confidence interval" → adds only `ANALYSIS: BOOTSTRAP = 1000;` and `OUTPUT: CINTERVAL (BOOTSTRAP);` on top of the existing code (nothing else gets added on its own).

### Example 2 — Confirmatory factor analysis (CFA)

> **User**: I want to check whether these 6 depression scale items load onto a single factor. The items are 5-point Likert scale.

**What the skill does**
1. Confirms the 6 items are categorical/ordinal, and how missing data is coded
2. Confirms the path diagram: a circle (`dep`) pointing to 6 rectangles (`d1`–`d6`), with a residual arrow on each item
3. Once confirmed, minimal code:

```
VARIABLE:
  NAMES ARE d1-d6;
  CATEGORICAL ARE d1-d6;
  MISSING ARE ALL (-999);

MODEL:
  dep BY d1-d6;
```

4. Example follow-ups: "I want to see fit indices" → adds `OUTPUT: STDYX;` / "I also want modification indices" → adds `OUTPUT: MODINDICES;`

### Example 3 — Multilevel modeling + latent profile analysis (LPA)

> **User**: My data has students nested in schools. I want to identify latent profiles of student motivation at the student level, while accounting for between-school differences.

**What the skill does**
1. Confirms the clustering variable (school ID), how many indicator variables define the profiles, and how missing data is coded
2. Confirms the path diagram: a dashed line separating Within/Between, a circle (`c`, the latent class variable) pointing to the indicator rectangles at the Within level, and the school clustering shown at the Between level
3. Once confirmed, minimal code:

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

(Note: the actual `MODEL` syntax depends on whether the profile indicators are class indicators or covariates — the skill confirms this with you in conversation before writing the code.)

4. Example follow-up: "How do I check how many classes/profiles is right?" → explains and adds `TECH11`/`TECH14` (LMR and bootstrap likelihood ratio tests) with the manual as the basis.

## Limitations

- This skill only covers **syntax that's actually in the manual**, so features added in newer Mplus versions or manual revisions released after this repository was built may not be reflected. (The skill is designed to fall back to a live lookup of statmodel.com in that case.)
- Statistical judgment — whether a model is theoretically sound, how to interpret results — is still up to you. This is a tool for writing correct syntax, not a statistical consulting service.
- Always review generated code before running it in actual Mplus.

## License

This repository (README, SKILL.md, references/*.md, and other original content) is distributed under the [MIT License](https://opensource.org/licenses/MIT). Mplus itself and the official User's Guide are copyrighted by Muthén & Muthén.
