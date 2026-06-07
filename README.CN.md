# multi-lens-thinking

> A personal-context-aware multi-perspective analysis skill for Claude Code / Cowork.
> Run a question through up to four independent lenses — **Macro · Personal · Local · Historical** — then synthesize a focused answer tailored to you, and learn from each session via memory patches you confirm.

## 繁體中文  // Other Language：[English](README.md)

### 為什麼有這個 skill

有一種 AI 輔助工具，第一次用覺得驚豔，用到第十次覺得空洞。你問黃金市場、地緣風險、或某個職涯決定值不值得——模型回傳一個流暢、自信、完全通用的答案。為「中位數讀者」校準的。

**中位數讀者不是你。**

它不知道你住在雪梨、報酬以澳元計價、對你而言最相關的工具是 ASX 礦商而非倫敦現貨。它不知道你過去如何思考類似情境。它也不知道哪些部分你早已了解、哪些才是你真正需要聽到的。

直覺的解法是塞更多 context 進 prompt：「我是雪梨的投資者、澳元計薪、五年期、以下是我已知的⋯⋯」但這帶來第二個、更隱性的問題：**context 污染**。要求單一 prompt 同時處理宏觀分析、歷史類比、在地市場動態，又要記住你所有個人限制——各個框架就會互相滲透。宏觀段落該以 USD 思考的地方開始講 AUD；歷史類比因你的風險偏好而偏移，無法嚴格聚焦在機制本身。答案*看起來*個人化，實際上是所有框架同時妥協的產物。

問題的根源不是模型能力，而是**架構**。單一 prompt 要求模型同時扮演所有角色。

### 它怎麼運作

讓每個分析維度在自己的 sub-agent、自己的 context 裡獨立運行，最後再綜合。整個 skill 的設計前提就是這一句。

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

**分流器（Router）** 讀取你的 `persona.md` + `memory.md` 一次，做兩個決定：

- 啟動哪些 lens——*積極跳過；多數問題不需要全部四個*
- 輸出哪個 `answer_mode`——`analytical` / `personal_decision` / `framework` / `meta`

四個 lens 接著**平行運行**（延遲 = 最慢的 lens，不是總和）：

| Lens | 在看什麼 | 是否搜尋？ |
|------|---------|-----------|
| **Macro** 宏觀 | 地緣、資本流、央行行為、貨幣秩序 | 是（網路） |
| **Personal** 個人 | 你的背景、限制、過往判斷 | 否——讀你，不讀世界 |
| **Local** 在地 | 你所在地的現實：價格、法規、可用工具、稅務 | 是（網路 / anysearch） |
| **Historical** 歷史 | 解釋當下「機制」的歷史類比——不是表面相似 | 是（網路 + LLM） |

**綜合器（Synthesizer）** 收四個 lens 輸出加 Router 的 mode，明確處理視角間的分歧。當宏觀與歷史意見不同，它會說明並解釋加權邏輯——而不是和稀泥成一句「綜合來看」。

### 「寫作陷阱」（為什麼 mode 很重要）

值得單獨命名的失效模式：分析師問「分析黃金走勢」——模型讀到 prompt 裡的職業背景，回的是一份寫作簡報而非分析本身。修法是：`answer_mode` 由**問題動詞**決定，不由你的身份決定。「分析黃金」永遠輸出 `analytical` 模式，即使你是專業寫作者。Personal lens 在 `analytical` 模式下縮為「語氣校準」；Synthesizer 被明確禁止輸出「你應該寫一篇⋯⋯」這類元評論。

### 不會說謊的記憶迴圈

每次對話結束後，記憶更新器會輸出一份**候選 patch**——擬加入 `memory.md` 的條目。你在下次對話開始時逐條審閱：approve / reject / edit。被拒絕的條目會被記錄，系統不會再提同樣建議。

這個設計**刻意慢**。自動更新的記憶會漂移；幾週的 AI 對話可能讓系統對你的認知悄悄走偏。Patch 確認機制讓你掌控系統學什麼，但又不需要在每次對話結束後手動維護背景檔案。

你的 `persona.md` 與 `memory.md` 保留在本機。公開 repo 只有 pipeline 邏輯本身。

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
