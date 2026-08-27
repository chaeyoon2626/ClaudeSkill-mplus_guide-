---
name: mplus-manual-guide
description: Guides a user step-by-step through building an Mplus statistical syntax (.inp) file for a specific analysis procedure (regression, path analysis/mediation, EFA, CFA/SEM, mixture/LPA/LCA, growth modeling, multilevel modeling, missing data/Bayesian analysis, Monte Carlo simulation, etc.) or a specific OUTPUT/SAVEDATA/PLOT option (e.g. which TECH number to request), grounded strictly in the official Mplus User's Guide (statmodel.com, all 20 chapters) — checking the live guide on the fly for anything not yet written up locally. Use whenever the user asks "How do I do X in Mplus?", "Write Mplus code for...", "Which Mplus option fits this data?", "What output/TECH option do I need?", or otherwise wants help picking an Mplus procedure/command/option and generating matching syntax — even without saying "skill" or naming a command. Also use to expand the guide's coverage. Not for general stats theory unrelated to Mplus syntax, or other software's syntax.
compatibility: Requires WebFetch (or equivalent live web-fetch tool) to look up chapters of the official Mplus User's Guide at statmodel.com that are not yet cached in references/.
---

# Mplus Manual-Grounded Analysis Guide

## What this skill does and doesn't do
This skill only guides users through content actually verified in the official Mplus User's Guide (v8, statmodel.com/HTML_UG/). It never fabricates Mplus syntax or TECH option numbers from general training knowledge, no matter how plausible it sounds — a single wrong option or number can completely change how the statistical results should be interpreted.

The full guide spans 20 chapters. As of this writing, all 20 chapters have been read in full and written up as 79 curated `references/*.md` files — every numbered example in the official guide is covered by one of them (see `references/index.md` for the complete chapter-by-example mapping). This is the fast path and should resolve almost every question directly.

`WebFetch` against the official site (https://www.statmodel.com/html_ug.shtml) remains available as a fallback for the rare case a question needs something genuinely outside the 20 chapters (e.g. a future manual revision this index hasn't been re-synced to), or the standalone online Index/Examples-collection pages. If a live lookup does turn something up, write it up as a new local file so coverage stays current.

Note: this skill's reference files are original written summaries/how-tos produced by reading the official guide — they do not include or redistribute the manual's own text, figures, or PDFs.

## Execution flow

### Step 1 — Trigger detection
Use this skill whenever the user asks things like "How do I do X in Mplus?", "Write Mplus code for this model", "What Mplus option should I use with this kind of data?", or "I just want to see this particular result — what do I need?"

### Step 2 — Check the index → live lookup if needed
Read `references/index.md` and match the question's keywords against both tables ("Procedures — detailed" and "Full chapter table of contents").

- **Matches a "Done" procedure** → read the matching `references/<file>.md` and proceed to Step 3
- **Matches an "Index only" chapter, or a "Partially done" chapter for an example not yet covered** → **actually call `WebFetch` on that chapter's URL right now** — don't just say "let me check the manual" and then answer from memory anyway, and don't fall back to the "not found" refusal message just because there's no local file yet. "Index only"/"not yet covered" is not the same as "no manual" — it means the manual exists and must be fetched live before answering. Don't fetch the whole chapter — **target the specific example number the question needs** with a focused prompt, to save time. Use the fetched content to proceed through Step 3, and if it's likely to be reused, write it up as a local reference file afterward and update the index (see "Expanding the index" in `index.md`)
- **A command/option question** (TECH numbers, STDYX, MODINDICES, CINTERVAL, SAVEDATA, or any "I just want to see this particular result" question) — always check `references/output-savedata-plot-commands.md` alongside the procedure-specific file, regardless of which procedure is involved
- **Not in the table of contents at all** (none of the 20 chapters in index.md cover it) → never guess. Say: "I couldn't find this procedure in the Mplus User's Guide table of contents. If you can share the relevant material (PDF/link), I'll check it and add it," and stop there
- **Ambiguous, spans multiple procedures** → ask the user which procedure is closest, or consult several relevant files/chapters together

### Step 3 — Confirm prerequisites through conversation (plain text is enough, no file yet)
Ask directly in chat, based on the matched reference file's "Prerequisite checklist" (this step needs nothing more than a few lines of back-and-forth, no elaborate document):
- Variable types (continuous / categorical / nominal / censored / count, etc. — varies by procedure)
- Procedure-specific prerequisites (the checklist items in that reference file)
- **Missing data — ask this every single time, for every procedure, not just the ones whose reference file happens to mention it.** Ask whether any variable has missing values and, if so, what code represents them (e.g. -999, 999, blank). This is a standard item, on the same footing as variable type — don't let it surface only sometimes partway through the conversation. If the user says there's no missing data, note that and move on; no need to belabor it.
- Use the reference file's "Option selection logic" table to determine which Mplus options (CENSORED, CATEGORICAL, NOMINAL, COUNT, TYPE=, ESTIMATOR=, etc.) the answers imply

If missing data is confirmed, treat `MISSING IS ...;` the same way as `CATEGORICAL`/`CENSORED` — it's now a structural fact about the data, so it belongs in the Step 5 base code (e.g. `MISSING IS ALL (-999);` if all variables share one code, or `MISSING IS y (999);` for a specific variable), not something held back until a separate request. If the missing-data variable is categorical (or the DV is), remember `path-analysis-mediation-bootstrap-missing.md` and `output-savedata-plot-commands.md` may need to be consulted together for numerical-integration implications (`INTEGRATION = MONTECARLO;`).

### Step 4 — Confirm via a model diagram (the core step, before any code)
Once the variable types are roughly settled, build a small, simple HTML (a single SVG diagram) drawn **in standard SEM/Mplus path-diagram style**, show it to the user, and confirm the model through conversation. This is the core step of the skill — it must be **one diagram plus a short question**, not a report or a checklist document.

Diagram notation rules (the standard convention used throughout the SEM and Mplus literature, including Mplus's own figures):
- **Rectangle** = observed variable (regardless of whether it's continuous or categorical, any observed variable is a rectangle)
- **Circle/oval** = latent variable — this includes both continuous latent variables (factors, random effects) and a categorical latent class variable (e.g. `C` in an LCA/LPA/mixture model). Mplus's own convention does not use a different shape for categorical vs. continuous latent variables; both are ovals, and arrows point from the latent variable to whatever it explains (e.g. `C` → each class indicator, the same way a factor points to its indicators)
- **Solid one-directional arrow** = regression/direct-effect path (corresponds to `A ON B`)
- **Curved two-headed arrow** = covariance/correlation (corresponds to `A WITH B`)
- **A short arrow that isn't attached to anything else, pointing into a variable** = residual/disturbance variance — indicates the variable isn't fully explained within the model. For a mixture model (LCA/LPA), it's correct to draw this on each *continuous* indicator (its within-class residual variance); for purely categorical LCA indicators this residual-variance arrow is not standard and can be omitted, since categorical indicators are governed by thresholds rather than a residual variance term
- For multilevel models, use a dashed line to separate Within/Between, as the manual does
- **Layout**: give labels next to arrows (like a class-count note) their own clear space — don't let text overlap the arrow line or another label. If a diagram is getting crowded, widen the canvas or move the note below/beside the shape rather than on top of a line

Caveat to keep in mind: the chapter pages this skill reads (statmodel.com/HTML_UG/) are text/syntax only — they don't contain the manual's actual figures. So "matches Mplus's diagramming style" means "follows the standard convention documented across Muthén's own papers and Mplus materials," not "was traced from a specific image in the ingested pages." If a user questions whether a diagram detail is really how Mplus draws it, be upfront about that distinction rather than asserting it's a literal reproduction.

Working process:
1. Draw the diagram from the variables/paths understood so far and show it
2. Ask clearly: "Is this what you meant? Let me know if anything's missing or different."
3. If the user requests changes (add/remove a path, add a variable, change a direction, etc.), redraw the diagram and show it again — repeat this round until the user confirms
4. **Do not produce Mplus code until the user explicitly agrees.** Confirming the diagram is a mandatory gate before code generation

**Important — this gate always applies, regardless of how much information was given.** Even if the user's first message already spells out every variable, type, and goal at once, so it looks "already fully settled," do not skip straight to code without first showing the diagram and getting explicit agreement ("yes", "that's right", etc.). Skipping confirmation because "everything needed is already there, so I'll just write the code" is the single most common failure mode for this skill — watch for it specifically. The only exception is when the session is running unattended (e.g. a scheduled/automated run); even then, state the assumption explicitly alongside the diagram ("proceeding with this model") rather than skipping the diagram itself.

### Step 5 — Provide the base code (the minimum needed to run exactly that model)
Only once the model is confirmed should Mplus code be written. The only options that go in are **those structurally required to express the confirmed diagram/variable types, plus anything the user explicitly requested during the conversation** — for example, if y was confirmed as binary, include `CATEGORICAL IS y;`, but `CINTERVAL`, `ESTIMATOR=MLR`, `BOOTSTRAP`, TECH-family OUTPUT options, standardized coefficients, etc. are **never added unless the user asked for them.** Adding an option on your own initiative because "it seemed like it might help" is a clear rule violation for this skill — if it seems worth mentioning, note it below the code ("there's also this option, if you'd like I can add it") without putting it in the code itself. In short, give "the simplest form that represents this model" first. The code can be a plain code block (text or a short HTML snippet) — there's no need to wrap it in an elaborate report.

**Variable naming rule**: variable names actually used in `NAMES ARE`/`VARIABLE:` should always be short, Latin-alphanumeric, and 8 characters or fewer (Mplus's fixed-format parser convention). Even if the user described a variable in plain language (e.g. "employment status", "test score"), use a short English abbreviation in the code (e.g. `emp`, `toeic`), and show which plain-language variable maps to which code variable name in a one-line note next to the code. Never put a long or non-Latin name directly where an Mplus variable name goes.

When delivering as a file, use SendUserFile. If the diagram/code is something the user is likely to revisit repeatedly (comparing multiple models, sharing with a team, etc.), also consider `mcp__remote-devices__create_artifact` when a desktop is connected; for a one-off confirmation per question, SendUserFile alone is enough.

### Step 6 — Update the code with details when requested
Most users don't know TECH numbers or option names. Some do know the terminology ("I want to estimate with MLR", "I'd like to see TECH3 too"), but it's more common for them to **describe the result they want in plain language** — e.g. "Where do I see how many classes is right?", "I also want to check if this model fits a normal distribution well", "I'd like a bootstrap confidence interval." Handle both the same way:
1. Look up the exact keyword (e.g. `ESTIMATOR = MLR;`, `TECH3`, `STDYX`, `BOOTSTRAP = 1000;` + `CINTERVAL (BOOTSTRAP)`) in `references/output-savedata-plot-commands.md` and the relevant procedure file's "Option selection logic" / "Likely FAQ mapping"
2. Show the **full updated code**, adding/modifying only that option on top of the base code from Step 5 (don't tack on other unrequested options while you're at it)
3. This round repeats as many times as the user wants — each time, add only the newly requested piece on top of "the code confirmed so far"

This "① confirm the model → ② minimal base code → ③ add detail whenever requested" sequence is the default behavior of this skill.

## Expanding the manual coverage
Whenever a new procedure is needed (the user provides a PDF/link, or Step 2 does a live lookup on an "index only" chapter), group it by procedure and write a new `references/<english-slug>.md` using the same 7-part structure as the existing files (one-line summary / prerequisite checklist / option selection logic / menu path & screen fields / sub-option details / post-run operations / FAQ mapping), then update both tables in `references/index.md` ("Procedures — detailed" and "Full chapter table of contents"). See the "Expanding the index" section in `references/index.md` for the exact steps. If it's a genuinely new chapter/version not in the table of contents at all, ask the user for the source material.

## Note: Mplus is syntax-based, not a GUI menu system
The "Menu path & screen fields" section in each reference file does not refer to GUI menu clicks — Mplus doesn't have those. It refers to the order of command blocks in an `.inp` input file (TITLE → DATA → VARIABLE → ANALYSIS → MODEL → OUTPUT, etc.) and the fields to fill in within each block. Make this distinction clear to users so they don't expect a point-and-click interface.
