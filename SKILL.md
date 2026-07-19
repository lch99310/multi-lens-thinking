---
name: multi-lens-thinking
description: Analyze a question through up to four independent lenses — macro/geopolitical, personal background, local knowledge, historical analog — then synthesize a focused answer tailored to the user. Use when the user asks a decision-style question ("should I", "is it worth", "future of", "risk of", "opportunity in"), a career/relocation/investment question, or any question where multiple perspectives would materially improve the answer. Do NOT use for simple factual lookups, code/operational how-tos, or casual conversation.
---

# Multi-Lens Thinking

**Version 2.2** — multi-perspective analysis with a cross-cutting **Analytical Rigor Protocol** (`prompts/00-rigor.md`) and optional private `lch-llm-wiki` integration (`references/lch-wiki-integration.md`). The rigor layer is prepended to every lens and enforced by the Synthesizer as a pre-send gate so the answer stays balanced, sourced, privacy-aware, and complete.

A six-stage pipeline that runs a question through multiple independent perspectives, synthesizes them, and writes a memory patch for the next session to confirm.

## When to invoke

Invoke when the user's question matches any of these patterns:

- Decision framing: "should I", "is it worth", "好不好", "值不值得", "適不適合"
- Future/risk framing: "未來", "趨勢", "風險", "機會", "前景"
- Career / relocation / investment / major life choice
- Geopolitics, market thesis, industry analysis
- Explicit request: "用多維度分析", "幫我從幾個角度看", "lens"

Do NOT invoke for:

- Pure factual lookup (definitions, conversion, current price)
- Code or how-to ("how to install X", "syntax of Y")
- Casual conversation
- Tasks already covered by a more specific skill (the user explicitly invoked another skill)

## Pipeline overview

```
Question
  ↓
[1. Router]      — reads lch-llm-wiki when available, else persona.md + memory.md (no search)
  ↓ produces: answer_mode, user_snapshot, search_hints, active_nodes, skip_log
  ↓                     ↑
  ↓             (answer_mode is THE most important field; it controls
  ↓              Personal lens depth and Synthesizer output style)
  ↓
  ├─ [2a. Macro]      — WebSearch
  ├─ [2b. Personal]   — no web search; prefers lch-llm-wiki; behavior depends on answer_mode
  ├─ [2c. Local]      — WebSearch → anysearch fallback
  └─ [2d. Historical] — hybrid (LLM + targeted WebSearch)
  ↓
[3. Synthesizer] — integrates outputs; output style depends on answer_mode
  ↓
Final answer to user
  ↓
[4. Memory Updater] — writes a patch file (no search)
```

Steps 2a–2d run in parallel. Each is a separate Task tool sub-agent so contexts stay isolated.

## Private wiki and user data location

### Preferred personal context: `lch-llm-wiki`

When the user's private `lch99310/lch-llm-wiki` repository is available, treat it as the preferred personal knowledge base and source of truth for the Router, Personal lens, and Synthesizer privacy gate.

Before using wiki context, read `references/lch-wiki-integration.md`. It defines how to resolve `LCH_WIKI_DIR`, which wiki pages to read, and how to apply the P/R/C disclosure gate.

Use the wiki for:

- Personal background, career facts, positioning, goals, constraints, and prior decisions.
- Writing voice, depth appetite, analytical style, and product-thinking patterns.
- Privacy boundaries that keep sensitive details out of user-facing answers.

The wiki is **not** a public quotation source. Do not dump raw wiki contents into the answer.

### Legacy user data

This skill has two kinds of files:

- **Skill files** (this directory): `SKILL.md`, `prompts/`, `templates/`, `agents/`.
- **User data files**: `persona.md`, `memory.md`, `patches/`. Must live somewhere the agent can both READ and WRITE.

The legacy user data location (`USER_DATA_DIR`) is resolved at runtime in this priority order:

1. **Connected Cowork folder + `multi-lens/` subdir** — if the user has selected (mounted) a folder in Cowork, use `<mounted_folder>/multi-lens/`. This is the recommended setup.
2. **Skill install dir** — fallback for read-only access to `persona.md` and `memory.md` if the user copied them there during install. Patches CANNOT be written here; they fall through to (3).
3. **Outputs folder** — last-resort fallback for writing patches. The agent must tell the user the path and instruct them to move the patch into their actual data dir manually.

If neither (1) nor (2) yields a readable `persona.md`, but `lch-llm-wiki` is available, continue with wiki context and create/use `memory.md` only if writable. If neither wiki context nor `persona.md` is available, stop and ask the user to provide one of them.

## Execution procedure

The main agent (you) executes this skill by following these steps in order.

### Step −1 — Resolve LCH_WIKI_DIR and USER_DATA_DIR

First read `references/lch-wiki-integration.md` and try to resolve `LCH_WIKI_DIR`.

If `LCH_WIKI_DIR` is available:

- Read `<LCH_WIKI_DIR>/AGENTS.md`.
- Read `<LCH_WIKI_DIR>/wiki/index.md`.
- For the user's question, read only the relevant linked wiki pages.
- For personal facts, verify against `<LCH_WIKI_DIR>/wiki/profile/source-of-truth.md`.
- Record `WIKI_CONTEXT_AVAILABLE = true`.

If unavailable, record `WIKI_CONTEXT_AVAILABLE = false` and use legacy `persona.md` / `memory.md`.

Determine where `persona.md`, `memory.md`, `patches/` live:

1. Check whether a Cowork folder is mounted (env shows a selected folder). If yes, `USER_DATA_DIR = <mounted>/multi-lens/`. Create the subdir if missing.
2. Else, check whether `persona.md` exists alongside this SKILL.md. If yes, `USER_DATA_DIR = <skill-dir>` (read-only — patches will fall through to outputs).
3. Else, if `WIKI_CONTEXT_AVAILABLE == true`, continue without `persona.md`.
4. Else, stop and tell the user: "I need either `lch-llm-wiki` or a `persona.md` to run. Please either provide a local `lch-llm-wiki` checkout, select/connect a folder via Cowork with `persona.md` inside `<folder>/multi-lens/`, or place `persona.md` alongside SKILL.md."

Internally remember:
- `PERSONA_PATH = <USER_DATA_DIR>/persona.md`
- `MEMORY_PATH = <USER_DATA_DIR>/memory.md` (create empty from template if missing and writable)
- `PATCHES_DIR = <USER_DATA_DIR>/patches/` if writable; else `<outputs>/patches/`
- `PATCHES_WRITABLE_AT_DATA_DIR = true/false`

### Step 0 — Check for pending memory patches

List files in `<USER_DATA_DIR>/patches/`. If any patch files exist that haven't been merged, show them to the user one at a time and ask "approve / reject / edit" before merging into `memory.md`. Skip this step silently if patches dir is empty or unreachable.

### Step 1 — Run Router

Read these files:

- `references/lch-wiki-integration.md`
- If wiki context is available: relevant pages from `LCH_WIKI_DIR`, starting at `wiki/index.md`
- `<PERSONA_PATH>` if present
- `<MEMORY_PATH>` if present
- `<skill-dir>/prompts/01-router.md`

Apply the router prompt to the user's question. Prefer wiki-backed facts over legacy persona facts when they conflict. The router output is a structured JSON-like block — keep it in your working memory; do NOT show it to the user verbatim.

The v2 router also emits a `coverage_map` (actors / domains / threads). Keep it: the Synthesizer cross-checks against it so no actor, dimension, or sub-thread is silently dropped (rigor rule 12).

Router output schema:

```
{
  "question_type": "decision | thesis | comparison | exploration | other",
  "answer_mode": "analytical | personal_decision | framework | meta",
  "user_snapshot": "<2–4 sentence digest of which parts of persona/memory matter for this question>",
  "search_hints": {
    "macro": "<keywords / regions / actors to focus on>",
    "local": "<location + vertical hint, e.g. 'Sydney + jobs+infrastructure'>",
    "historical": "<comparable era / event keywords>"
  },
  "active_nodes": ["macro", "personal", "local", "historical"],
  "skip_log": [
    {"node": "<name>", "reason": "<one-sentence justification>"}
  ]
}
```

**`answer_mode` is the most important field.** It is set by the question verb, NOT by the user's professional identity. A writer asking "analyze X" still gets `analytical` — Personal lens then contributes only voice; Synthesizer delivers the analysis itself, not a writing brief.

If `active_nodes` is empty (router decided no lens helps), tell the user and answer directly without the pipeline.

### Step 2 — Spawn parallel sub-agents

For each node in `active_nodes`, launch a sub-agent via the Task tool **in the same message** (parallel execution). Use the `general-purpose` agent type.

Each sub-agent prompt is built as:

```
<CONTENTS OF prompts/00-rigor.md — the Analytical Rigor Protocol, prepended verbatim to EVERY node>

<role prompt from prompts/02-{node}.md>

<if node == personal and WIKI_CONTEXT_AVAILABLE: concise excerpts / page-path citations from relevant lch-llm-wiki pages, plus disclosure level notes from source-of-truth.md>

USER QUESTION:
<original question>

ANSWER_MODE: <one of: analytical | personal_decision | framework | meta>

USER SNAPSHOT (from router):
<user_snapshot>

SEARCH HINT (from router):
<search_hints[node]>     // omit if node doesn't search

OUTPUT FORMAT:
<for Macro/Local/Historical: existing 400-word format>
<for Personal: behavior depends on answer_mode — see prompts/03-personal.md>
```

Every node must obey rules 1–11 of the rigor protocol within its slice (balance the actors it discusses, cite events as events not as borrowed conclusions, tag fact vs inference, triangulate sources, surface the counter-case to its own key claim).

Search-enabled nodes (macro, local, historical) must cite at least one URL. Personal node must cite at least one specific element of lch-llm-wiki, persona.md, or memory.md it relied on (and in analytical mode, output drops to ~250 words and excludes any topic-redirection).

### Step 3 — Synthesizer

After all sub-agents return, read `prompts/06-synthesizer.md` and apply it yourself. Before sending, the synthesizer MUST run the **Completeness & Objectivity Gate** (10 checks appended to that prompt in v2) and cross-check against the router's `coverage_map`. The bar: the user should not be able to name a major actor, indicator, counter-case, or thread that was missed. The synthesizer input is:

- Original question
- `answer_mode` from router (controls structure and forbidden patterns)
- `user_snapshot` from router
- `skip_log` from router
- All node outputs (verbatim)

Before producing the final answer, apply the disclosure gate from `references/lch-wiki-integration.md` if wiki context was used.

The synthesizer produces the **final user-facing answer**. Behavior depends on `answer_mode`:

- **analytical**: deliver the analysis itself. Do NOT write a writing brief, article title, or "you should engage with this topic" meta-commentary. Personal lens contributes voice only.
- **personal_decision**: lead with what user actually cares about for their own decision; surface tradeoffs; end with concrete next step.
- **framework**: lead with the framework; lenses fill it.
- **meta**: minimal direct answer; skip lens scaffolding.

In all modes: surface real disagreements between lenses (don't paper them over); mention `skip_log` briefly at end; use the user's preferred language (Traditional Chinese unless the question was in English).

### Step 4 — Memory Updater

After delivering the final answer, read `prompts/07-memory-updater.md` and apply it to produce a patch file at `<PATCHES_DIR>/YYYYMMDD-HHMMSS.md`. Do NOT modify `memory.md` directly — the user confirms next session.

- If `PATCHES_WRITABLE_AT_DATA_DIR == true`: silently write the patch and tell user: "已寫入候選 memory patch，下次開啟會請你確認。"
- If patches had to fall through to outputs (skill install dir is read-only): write to `<outputs>/patches/` and tell user: "已將候選 memory patch 寫到 outputs/patches/YYYYMMDD-HHMMSS.md。因 skill 安裝目錄不可寫，請手動移到你的資料夾中、或下次跑時連接一個可寫資料夾。"

If the session produced nothing memory-worthy, write a "NO PATCHES PROPOSED" file (still useful as audit trail) — but don't surface it to the user beyond one quiet line.

## Failure modes & guardrails

- **Router output malformed**: fall back to running all four nodes with empty search_hints; default `answer_mode` to `analytical` (the safer fallback — won't accidentally personalize the answer).
- **Router puts a writer/analyst into `personal_decision` mode for an external question**: this is the known failure mode (see Bug fix v1.1). The question verb determines `answer_mode`, not the user's profession.
- **WebSearch fails / returns empty**: that node should explicitly say "搜尋失敗" rather than hallucinate; synthesizer should down-weight it. If macro or local fails twice, try anysearch via `scripts/anysearch_cli.py search "<query>"`.
- **Sub-agent times out**: skip it in synthesis and note in the skip_log.
- **persona.md missing**: continue if `lch-llm-wiki` is available; otherwise tell the user this skill needs either wiki context or persona.md to work and stop.
- **memory.md missing**: create an empty one with the template header and continue.
- **`PATCHES_DIR` not writable**: write to outputs/patches/ and notify the user explicitly.
- **Incomplete / one-sided / inherited analysis (v2 — the class this version targets)**: the analysis characterizes an actor with only one side of the ledger, leans on a think-tank/wargame/model conclusion instead of primary facts and revealed behavior, rests on one bloc's source framing, keeps a comforting assumption without stress-testing it, treats a changing fact as fixed, or silently drops a thread. **Guardrail**: the rigor protocol (`prompts/00-rigor.md`) is prepended to every node, the router emits a `coverage_map`, and the synthesizer runs the 10-point Completeness & Objectivity Gate before sending. If the user can name a missing actor/indicator/counter-case/thread, the gate failed.
- **Private wiki leakage (v2.2)**: `lch-llm-wiki` contains private personal and career material. **Guardrail**: read `references/lch-wiki-integration.md`, obey the P/R/C disclosure gate, and never surface confidential wiki details as user-facing facts unless the user explicitly asks for that exact detail.

## Files in this skill

- `SKILL.md` — this file
- `prompts/00-rigor.md` — **Analytical Rigor Protocol (v2)**; prepended to every node, enforced by the synthesizer
- `prompts/01-router.md`
- `prompts/02-macro.md`
- `prompts/03-personal.md`
- `prompts/04-local.md`
- `prompts/05-historical.md`
- `prompts/06-synthesizer.md`
- `prompts/07-memory-updater.md`
- `references/lch-wiki-integration.md` — private wiki lookup and disclosure gate
- `templates/persona.md` — copy to skill root on first install
- `templates/memory.md` — copy to skill root on first install
- `patches/` — generated patch files awaiting user confirmation
- `agents/openai.yaml` — UI metadata for skill lists and chips
