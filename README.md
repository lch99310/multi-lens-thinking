# multi-lens-thinking

> A personal-context-aware multi-perspective analysis skill for Claude Code / Cowork.
> Run a question through up to four independent lenses — **Macro · Personal · Local · Historical** — then synthesize a focused answer tailored to you, and learn from each session via memory patches you confirm.

---

## English

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
"我在考慮要不要去悉尼做數據中心相關的工作"
"2026 川習會對台海局勢的影響怎麼看"
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

---

## 繁體中文

### 這個 skill 在幹嘛

`multi-lens-thinking` 是一條六階段的 pipeline：

1. 把一個決策型或論點型的問題，分解成幾個會實質影響答案的視角。
2. 每個視角各用一個獨立的 sub-agent 跑，context 與搜尋預算彼此隔離。
3. 把這些輸出整合成一份**為「你這個人」量身的答案**（透過 `persona.md` + `memory.md`）。
4. 寫一個候選 memory patch，下次 session 由你親自確認——這條學習迴路慢、刻意、可審計，避免自動更新 memory 慢慢污染信號。

它**不是**「幫我給四個觀點」這種泛用 prompt。Router 在沒幫助時會**主動跳過 lens**；Synthesizer 是 mode-aware 的：分析題給你分析（不是寫作大綱），個人決策題給你建議（不是市場評論）。

### 為什麼用 pipeline（而不是一個大 prompt）

- **Context 隔離**：每個 lens 在獨立 sub-agent 中跑，避免 Macro 的長搜尋結果污染 Personal lens 對你 persona 的閱讀。
- **平行執行**：Macro / Personal / Local / Historical 同時跑。延遲 ≈ 最慢的 lens，而非總和。
- **Mode-aware 整合**：Router 輸出一個 `answer_mode`（`analytical | personal_decision | framework | meta`），由**問題動詞**決定，**不**由你的職業身份決定。這修掉了「writer trap」——以前一位分析師問「分析黃金」會拿到一份寫作大綱，而非分析本身。
- **可審計的 memory**：Memory Updater 只寫 patch、絕不直接改 `memory.md`。每筆 patch 由你 approve 後才落地。

### 架構圖

```
問題
  ↓
[Router]  ← 一次性讀 persona.md + memory.md
  輸出: {
      question_type,
      answer_mode,         ← 控制 Personal lens 深度與 Synthesizer 風格
      user_snapshot,       ← 共用快照
      search_hints,        ← 給 Macro / Local / Historical 的搜尋提示
      active_nodes,        ← 要啟動哪些並行節點
      skip_log             ← 跳過的 lens 各自為何被跳過
  }
  ↓
  ├─ Macro       (地緣與資本流)        ← user_snapshot   → WebSearch
  ├─ Personal    (你的背景)            ← user_snapshot     (不搜尋)
  ├─ Local       (在地現實)            ← search_hints    → WebSearch / anysearch
  └─ Historical  (歷史類比)            ← search_hints    → WebSearch + LLM
  ↓
[Synthesizer]  ← 四個 lens 輸出 + answer_mode + user_snapshot + skip_log
  ↓
回覆給使用者
  ↓
[Memory Updater] → patches/YYYYMMDD-HHMMSS.md   （下次 session 確認後合入 memory.md）
```

四個 lens 節點透過 Task tool **平行**啟動，各自獨立 sub-agent，context 互不干擾。

### 安裝

```bash
git clone https://github.com/<你的帳號>/multi-lens-thinking ~/.claude/skills/multi-lens-thinking
cd ~/.claude/skills/multi-lens-thinking
cp templates/persona.md persona.md
cp templates/memory.md  memory.md
# 編輯 persona.md，寫成你自己真實的樣子
```

`persona.md` 與 `memory.md` 已被 gitignore——公開 repo 只有 templates，你的真實內容只留在本機。

**選配**：安裝 [anysearch-skill](https://github.com/anysearch-ai/anysearch-skill) 作為 Local lens 在 WebSearch 結果稀薄時的垂直搜尋備援（求職、房地產、地方新聞）。預設用 WebSearch，anysearch 只在需要時介入。

### 第一次使用

直接問任何決策型或論點型問題，skill 會依 description 自動觸發：

```
"我在考慮要不要去悉尼做數據中心相關的工作"
"2026 川習會對台海局勢的影響怎麼看"
"基於當下的地緣風險與宏觀經濟，分析未來三年黃金最有可能的走勢"
```

顯式呼叫：在問題前加 `用多維度分析` 或 `/multi-lens-thinking`。

### 完整範例 — 2026-2029 黃金走勢

**問題** *(analytical 模式——主體是黃金，不是使用者)*：

> 「基於當下(2026年五月)的地緣風險與宏觀經濟，分析未來三年黃金最有可能的走勢。要基於事實信息與數據的推導與分析。」

**Router 決策**（內部）：

```json
{
  "question_type": "thesis",
  "answer_mode": "analytical",
  "active_nodes": ["macro", "personal", "local", "historical"],
  "skip_log": []
}
```

**四個 lens 輸出**（壓縮重點）：

| Lens | 關鍵洞察 |
|------|---------|
| **Macro** | 三力疊加：央行非價格敏感買盤（WGC Q1'26 淨買 244t）+ 美財政無解（CBO FY26 赤字 1.9T / 5.8% GDP）+ BRICS Unit 結算層黃金化 → 基準目標 USD 5,500–7,000 / oz，年化 12–18%。 |
| **Personal** | 僅做 voice 校準（analytical 模式）。語氣：formal-analytical、mechanism-over-narrative。禁區：兩面下注、空泛趨勢詞。 |
| **Local** | AUD/USD 0.69–0.73（RBA 4.35% 鷹派 hold）→ AUD 計價漲幅相對 USD 漲幅打折 20–30%。ASX 槓桿路徑：EVN（淨現金）> NST（產量下修 + Elliott 壓力）。 |
| **Historical** | 最強類比是 **1971–1980**（布列敦森林崩解 + 儲備武器化），**不是** 2001–2011 QE 多頭。推論：結構性向上，但路徑中段預期至少一次 **25–35% 回檔**（類比 1974–76 的 −40%）。 |

**Synthesizer 最終輸出**（給使用者看的答案）：

> **基準情境**：結構性多頭延續到 2029，USD 目標區間 **5,500–7,000 / oz**（年化 12–18%）。路徑非單調，預期至少一次 25–35% 中段回檔。對 AUD 計價投資者而言，RBA 鷹派 hold 會把實質報酬吃掉 20–30%；最乾淨的槓桿路徑是有成本紀律的 ASX 金礦商（EVN > NST）。
>
> *三個破局訊號（觀察清單）*：
> 1. PBOC 公布持倉跳升至 5,000+ 噸（去美元化加速）
> 2. 西方 ETF 部位回到 2020 高點（第二波私人需求啟動）
> 3. 30Y TIPS 殖利率突破 2.5% 同時金價仍創高（黃金-實質利率脫鉤確認）
>
> *我會錯在哪*：Fed 被迫 YCC + 美元有序貶值 + 地緣緩和（台海 + 中東降溫 + BRICS 內部分裂）→ 央行買盤節奏放緩 30-40%，行情壓回循環性。

**Memory patch** 已寫入 `patches/YYYYMMDD-HHMMSS.md`，狀態 `PENDING USER REVIEW`。下次 session 的 Step 0 會顯示這份 patch 並請你 approve / reject / edit。

### 檔案結構

```
multi-lens-thinking/
├── SKILL.md              ← skill manifest 與執行程序
├── README.md             ← 本檔
├── LICENSE
├── .gitignore            ← 排除 persona.md / memory.md / patches/
├── persona.md            ← 你的 persona——gitignored，從 templates/ 複製
├── memory.md             ← 你的 memory——gitignored，從 templates/ 複製
├── prompts/
│   ├── 01-router.md
│   ├── 02-macro.md
│   ├── 03-personal.md
│   ├── 04-local.md
│   ├── 05-historical.md
│   ├── 06-synthesizer.md
│   └── 07-memory-updater.md
├── templates/
│   ├── persona.md        ← 通用結構模板
│   └── memory.md         ← 通用結構模板
└── patches/              ← 候選 patches，等你確認
    └── .gitkeep
```

### Memory 迴路

```
Session N         → Memory Updater → patches/YYYYMMDD-HHMMSS.md  （候選）
                                              ↓
Session N+1 Step 0 → 「approve / reject / edit?」 → memory.md（批准後合入）
```

你**逐筆**核可，不是整檔接受。被拒絕的條目會記錄起來，系統不會再提同樣建議。

這條「慢、刻意、可審計」的迴路比自動更新 memory 健康——後者會慢慢污染信號。

### 成本與延遲

| 情境 | 延遲 | Tokens |
|------|------|--------|
| 四個 lens 全跑 + 搜尋 | ~30–60s | ~8–20k |
| 跳過比例高（只啟動 1–2 個 lens） | ~10–20s | ~2–6k |
| Router 判定無 lens 值得跑 | ~3s | ~1k |

Router 的「積極跳過」規則是刻意的：大多數「X 是不是好主意」的問題不需要歷史類比，多數 how-to 問題不需要任何 lens。

### 客製化

- **改某個 lens 的 prompt**：編輯 `prompts/02-*.md` 到 `prompts/05-*.md`，各自獨立。
- **新增一個 lens**：新增 `prompts/0N-<name>.md`，更新 `SKILL.md` 的 pipeline 流程圖與 Router 的 `active_nodes` enum，並在 `06-synthesizer.md` 的輸入清單加入新節點。
- **跳過規則的鬆緊**：編輯 `prompts/01-router.md` 的 "Decision rules for active_nodes" 段落。
- **最終答案的聲音**：編輯 `prompts/06-synthesizer.md` 的 "Language" 段落。

### Fork 並發布到自己 GitHub

這個 skill 設計成可安全 fork。你的真實 `persona.md` / `memory.md` 只在本機，公開 repo 只有 templates。

```bash
cd ~/.claude/skills/multi-lens-thinking
git init
git add .
git status                                   # 確認 persona.md / memory.md 沒被 staged
git commit -m "Initial commit"
git remote add origin git@github.com:<你>/multi-lens-thinking.git
git push -u origin main
```

如果不慎把 `persona.md` commit 上去，當作 credential leak 處理——用 `git filter-repo` 重寫歷史，或乾脆刪除 repo 重來。

### 已知限制

- Sub-agent 有獨立 context，看不到你前面的對話。如果問題依賴對話歷史，請在問題中帶入。
- WebSearch 品質依主題而異。Macro 最依賴搜尋，冷門主題可用 anysearch 的垂直搜尋補強。
- Memory Updater 故意保守。想立刻記住某個事實，直接編輯 `memory.md` 即可，不必走 patch 流程。
- 成本隨啟動的 lens 數量線性成長。問題越具體，Router 越積極跳過、整體越省。

### 授權

MIT。詳見 `LICENSE`。
