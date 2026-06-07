# multi-lens-thinking

> A personal-context-aware multi-perspective analysis skill for Claude Code / Cowork.
> Run a question through up to four independent lenses — **Macro · Personal · Local · Historical** — then synthesize a focused answer tailored to you, and learn from each session via memory patches you confirm.

---

## English  // Other Language：[Chinese](README.CN.md)

### Why this exists

There's a version of AI assistance that feels impressive the first time and hollow the tenth. You ask about gold, geopolitical risk, or whether a career move makes sense — and the model returns a polished, confident, completely generic answer. Calibrated for the median reader.

**The median reader is not you.**

The model doesn't know you're based in Sydney, that your returns are AUD-denominated, that the relevant instruments for you are ASX-listed miners — not London spot. It doesn't know how you've thought about similar situations before. It doesn't know which parts of its answer you already know and which you actually needed to hear.

The naive fix is to stuff more context into the prompt: *"I'm a Sydney-based investor, AUD-paid, 5-year horizon, here's what I already know..."*. But this introduces a second, subtler failure: **context bleeding**. Ask a single prompt to simultaneously run macro analysis, historical analogies, and local market dynamics while holding all your personal constraints — and the frames bleed into each other. The macro section starts thinking in AUD when it should stay in USD. The historical analogy drifts toward your risk tolerance instead of staying disciplined on the mechanism. The answer *feels* personalized but is actually a compromise of every frame at once.

The root cause is not model capability — it's **architecture**. A single prompt asks the model to be all things at once.

### How it works

Run each analytical dimension as its own sub-agent, in its own context. Synthesize at the end. That's the entire premise.

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

The **Router** reads your `persona.md` + `memory.md` once and decides:

- which lenses to activate — *skip aggressively; most questions don't need all four*
- which `answer_mode` to emit — `analytical` / `personal_decision` / `framework` / `meta`

Four lenses then run **in parallel** (latency = slowest lens, not the sum):

| Lens | What it sees | Searches? |
|------|--------------|-----------|
| **Macro** | Geopolitics, capital flows, central-bank behavior, monetary regime | Yes (web) |
| **Personal** | Your background, constraints, prior judgments | No — reads you, not the world |
| **Local** | On-the-ground reality in your geography: prices, regulations, instruments, tax | Yes (web / anysearch) |
| **Historical** | The analog from the past that explains the present *mechanism* — not just surface similarity | Yes (web + LLM) |

The **Synthesizer** receives all four outputs plus the Router's mode and resolves conflicts explicitly. Where Macro and Historical disagree, it says so and explains its weighting — instead of papering them into a "balanced" sentence.

### The "writer trap" (why mode matters)

A failure mode worth naming. An analyst asks *"analyze gold"* — and the model, having read their profession, returns a writing brief instead of analysis. The fix: `answer_mode` is determined by the **question verb**, not your identity. *"Analyze gold"* always emits `analytical` mode, even if you're a professional writer. Personal lens in `analytical` mode shrinks to voice-calibration only; the Synthesizer is explicitly forbidden from "you should write a piece on this" meta-commentary.

### A memory loop that stays honest

After each session, the Memory Updater writes a **candidate patch** — proposed additions to `memory.md`. You review at the start of the next session: approve / reject / edit per item. Rejected items are logged so the system stops proposing them.

This is **deliberately slow**. Auto-updating memory drifts; a few weeks of AI sessions can quietly poison what the model thinks it knows about you. The patch-and-confirm loop keeps you in control of what the system learns — without forcing you to maintain a context file after every conversation.

Your `persona.md` and `memory.md` stay on your machine. The public repo ships only the pipeline logic.

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

