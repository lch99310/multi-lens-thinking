# multi-lens-thinking

> A personal-context-aware multi-perspective analysis skill for Claude Code / Cowork.
> Run a question through up to four independent lenses — **Macro · Personal · Local · Historical** — then synthesize a focused answer tailored to you, and learn from each session via memory patches you confirm.

---

## English  // Other Language：[Chinese](README.CN.md)

### What this skill does

`multi-lens-thinking` is a six-stage pipeline that:

1. Decomposes a decision-style or thesis-style question into the perspectives that materially affect the answer.
2. Runs each perspective as an isolated sub-agent, with its own context and search budget.
3. Synthesizes the outputs into a single answer **tailored to who you actually are** (via `persona.md` + `memory.md`).
4. Writes a candidate memory patch you confirm next session — so the system learns your views, your prior judgments, and the patterns it gets wrong, slowly and deliberately.

It is **not** a generic "give me four perspectives" prompt. The Router can **skip lenses** when they don't help, and the Synthesizer is mode-aware: an analytical question gets analysis (not a writing brief), a personal-decision question gets advice (not market commentary).

### Why a pipeline (vs. one big prompt)

- **Context isolation**: each lens runs in its own sub-agent, so a long Macro search doesn't pollute the Personal lens's read of your persona.
- **Parallel execution**: Macro / Personal / Local / Historical run concurrently. Latency ≈ slowest lens, not sum.
- **Mode-aware synthesis**: the Router emits an `answer_mode` (`analytical | personal_decision | framework | meta`) determined by the question verb, NOT your professional identity. This is the fix for the "writer trap" — where an analyst asking "analyze gold" used to get a writing brief instead of the analysis.
- **Auditable memory**: the Memory Updater writes patches, never edits `memory.md` directly. You approve each patch before it lands.

### Architecture

```
Question
  ↓
[Router]  ← reads persona.md + memory.md once
  output: {
      question_type,
      answer_mode,         ← controls Personal-lens depth and Synthesizer style
      user_snapshot,       ← shared digest
      search_hints,        ← hints for Macro / Local / Historical
      active_nodes,        ← which parallel nodes to fire
      skip_log             ← why each skipped lens was skipped
  }
  ↓
  ├─ Macro       (geopolitics & capital flow)   ← user_snapshot   → WebSearch
  ├─ Personal    (your background)              ← user_snapshot     (no search)
  ├─ Local       (on-the-ground reality)        ← search_hints    → WebSearch / anysearch
  └─ Historical  (analog from the past)         ← search_hints    → WebSearch + LLM
  ↓
[Synthesizer]  ← four lens outputs + answer_mode + user_snapshot + skip_log
  ↓
Reply to user
  ↓
[Memory Updater] → patches/YYYYMMDD-HHMMSS.md   (confirmed next session, then merged)
```

The four lens nodes run **in parallel** via the Task tool. Each is a separate sub-agent so contexts stay isolated.

### Install

```bash
git clone https://github.com/<your-username>/multi-lens-thinking ~/.claude/skills/multi-lens-thinking
cd ~/.claude/skills/multi-lens-thinking
cp templates/persona.md persona.md
cp templates/memory.md  memory.md
# Now edit persona.md to reflect who you actually are.
```

`persona.md` and `memory.md` are gitignored — only the templates ship in the public repo, your real content stays local.

**Optional**: install [anysearch-skill](https://github.com/anysearch-ai/anysearch-skill) as the Local lens's fallback for vertical search (jobs, real estate, local news). WebSearch is the default; anysearch only kicks in when WebSearch returns thin results.

### First run

Ask any decision-style or thesis-style question. The skill auto-triggers via its description:

```
"基於當下的地緣風險與宏觀經濟，分析未來三年黃金最有可能的走勢"
```

To invoke explicitly: prefix with `用多維度分析` or `/multi-lens-thinking`.

### Worked example — gold 2026-2029 outlook

**Question** *(analytical mode — subject is gold, not the user)*:

> "基於當下(2026年五月)的地緣風險與宏觀經濟，分析未來三年黃金最有可能的走勢。要基於事實信息與數據的推導與分析。"

**Router decision** (internal):

```json
{
  "question_type": "thesis",
  "answer_mode": "analytical",
  "active_nodes": ["macro", "personal", "local", "historical"],
  "skip_log": []
}
```

**Lens outputs** (verbatim, compressed):

| Lens | Key insight |
|------|-------------|
| **Macro** | Three forces stack: non-price-sensitive central bank buying (WGC Q1'26 net 244t) + unsustainable US fiscal path (CBO FY26 deficit $1.9T / 5.8% GDP) + BRICS Unit settlement layer going gold-backed → base case USD 5,500–7,000 / oz, CAGR 12–18%. |
| **Personal** | Voice tuning only (analytical mode). Register: formal-analytical, mechanism-over-narrative. Forbidden: hedge-everything filler, vague trend-words. |
| **Local** | AUD/USD 0.69–0.73 (RBA 4.35% hawkish hold) → AUD-gold return is 20–30% lower than USD-gold return. Cleanest ASX leverage: EVN (net cash position) > NST (production cut + Elliott activist pressure). |
| **Historical** | Strongest analog is **1971–1980** (Bretton Woods break + reserve weaponization), **NOT** 2001–2011 QE bull. Implication: structural uptrend, but expect at least one **25–35% mid-cycle drawdown** (cf. 1974–76 −40%). |

**Synthesizer output** (final answer the user sees):

> **Base case**: structural bull market through 2029, USD target range **5,500–7,000 / oz** (CAGR 12–18%). Path is not monotonic — expect at least one 25–35% mid-cycle drawdown. For AUD-denominated investors, RBA's hawkish hold compresses the actual return by 20–30%; the cleanest leverage is ASX gold miners with cost-discipline (EVN > NST).
>
> *Three falsification signals to watch:*
> 1. PBOC reported holdings jump to 5,000+ tonnes (de-dollarization acceleration)
> 2. Western ETF positioning returns to 2020 highs (second-wave private demand)
> 3. 30-year TIPS yield breaks 2.5% while gold still prints highs (gold-real-yield decoupling confirmed)
>
> *I would be wrong if*: Fed forced into YCC + orderly USD devaluation + geopolitical de-escalation (Taiwan + Middle East cooling + BRICS fracturing) — central bank pace slows 30–40%, the move compresses back to a cyclical bounce.

**Memory patch written** to `patches/YYYYMMDD-HHMMSS.md`, status `PENDING USER REVIEW`. Next session's Step 0 will show this patch and ask: approve / reject / edit.

### File layout

```
multi-lens-thinking/
├── SKILL.md              ← skill manifest + execution procedure
├── README.md             ← this file
├── LICENSE
├── .gitignore            ← excludes persona.md, memory.md, patches/
├── persona.md            ← YOUR persona — gitignored, copied from templates/
├── memory.md             ← YOUR memory — gitignored, copied from templates/
├── prompts/
│   ├── 01-router.md
│   ├── 02-macro.md
│   ├── 03-personal.md
│   ├── 04-local.md
│   ├── 05-historical.md
│   ├── 06-synthesizer.md
│   └── 07-memory-updater.md
├── templates/
│   ├── persona.md        ← generic structural template
│   └── memory.md         ← generic structural template
└── patches/              ← generated patches awaiting your confirmation
    └── .gitkeep
```

### Memory loop

```
session N         → Memory Updater → patches/YYYYMMDD-HHMMSS.md  (candidate)
                                              ↓
session N+1 Step 0 → "approve / reject / edit?" → memory.md (merged on approve)
```

You approve **per item**, not per file. A rejected entry is logged so the system stops proposing it.

This is the slow, deliberate feedback loop — better than auto-updating memory and slowly poisoning it.

### Cost / latency expectations

| Scenario | Latency | Tokens |
|----------|---------|--------|
| Full 4-lens run with search | ~30–60s | ~8–20k |
| Skip-heavy (1–2 lenses active) | ~10–20s | ~2–6k |
| Router-only (no lens worth running) | ~3s | ~1k |

The Router's skip-aggressively rule is intentional: most "is X a good idea?" questions don't need a historical analog, and most how-to questions don't need any lens at all.

### Customizing

- **Change a lens prompt**: edit `prompts/02-*.md` through `prompts/05-*.md`. Each is self-contained.
- **Add a new lens**: add `prompts/0N-<name>.md`, update `SKILL.md`'s pipeline overview and the Router's `active_nodes` enum, then add the new node to `06-synthesizer.md`'s input list.
- **Tighten / loosen skipping**: edit the "Decision rules for active_nodes" section of `prompts/01-router.md`.
- **Change the voice of the final answer**: edit `prompts/06-synthesizer.md` → "Language" section.

### Publishing your fork

This skill is designed to be safe to fork. Your real `persona.md` / `memory.md` stay on your machine — only the generic templates ship.

```bash
cd ~/.claude/skills/multi-lens-thinking
git init
git add .
git status                                   # verify persona.md / memory.md are NOT staged
git commit -m "Initial commit"
git remote add origin git@github.com:<you>/multi-lens-thinking.git
git push -u origin main
```

If you ever accidentally commit `persona.md`, treat it as a credential leak — rewrite history with `git filter-repo` or delete the repo and start fresh.

### Known limits

- Sub-agents have isolated context; they don't see your prior conversation turns. If a question depends on history, include it in the question.
- WebSearch quality varies by topic. Macro is the most search-dependent; for very specialized topics, anysearch's vertical search helps.
- The Memory Updater is conservative by design. If you want a fact remembered immediately, edit `memory.md` directly — no patch dance.
- Cost scales linearly with lens count. Specific questions trigger more aggressive Router skipping.

### License

MIT. See `LICENSE`.

