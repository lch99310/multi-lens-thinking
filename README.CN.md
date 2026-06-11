# multi-lens-thinking

<p align="center">
  <img src="multi-lens_thinking_workflow.png" alt="Multi-Lens Thinking Workflow" width="400">
</p>

<h2 align="center">MLT —— Multi-Lens Thinking</h2>

<p align="center">
  中文 | <a href="README.md">English</a>
</p>

<p align="center">
  <img alt="Version" src="https://img.shields.io/badge/version-v1.0.0-brightgreen">
  <img alt="License" src="https://img.shields.io/badge/licence-MIT-blue">
  <img alt="Audience" src="https://img.shields.io/badge/audience-Everyone-orange">
  <img alt="Workflow" src="https://img.shields.io/badge/what%20this%20for-Asking%20Anything-purple">
</p>

> **同一個問題、四種角度、一個只為你而寫的答案。**

把問題拆給四副獨立的鏡頭——**宏觀 · 個人 · 在地 · 歷史**——各自並行思考，再合成一份*只屬於你的*判斷。

不再是那種「第一次驚豔、第十次空洞」的通用 AI 回答。


---

### 它能幫你做什麼

當你問的是這類問題：

- 「現在該不該轉到 ___ 工作 / 城市？」
- 「未來三年 ___ 行業 / 資產 / 政策怎麼走？」
- 「___ 這個機會，對我這個情況到底適不適合？」

通用的 AI 會給你一份「為中位數讀者寫的答案」。
**multi-lens-thinking 給的是寫給你的答案。**

| 它做的 | 你拿到的 |
|--------|---------|
| **拆視角** | 四個鏡頭並行——宏觀(地緣資本) · 個人(你的背景) · 在地(你所在地的現實) · 歷史(機制類比) |
| **避污染** | 每個鏡頭跑在自己的 context，不會互相滲透成「四不像」 |
| **明衝突** | 鏡頭結論不一致時，直接攤開、解釋加權邏輯，不和稀泥 |
| **留記憶** | 每次對話結束擬一份候選 memory patch，下次開機由你逐條批准——不會自動污染你的背景檔案 |

---

### 怎麼用（三步）

**1. 安裝並告訴它你是誰**

```bash
git clone https://github.com/<你的帳號>/multi-lens-thinking ~/.claude/skills/multi-lens-thinking
cd ~/.claude/skills/multi-lens-thinking
cp templates/persona.md persona.md
cp templates/memory.md  memory.md
# 編輯 persona.md，寫成你真實的樣子：所在地、職業、決策風格、現有部位 / 限制
```

`persona.md` 與 `memory.md` 已 gitignore，公開 repo 只有 templates，真實內容只留在本機。

**2. 直接問問題**

```
基於當下地緣與宏觀，分析未來三年黃金走勢
```

skill 會依問題型態自動觸發；要顯式呼叫就在前面加 `用多維度分析` 或 `/multi-lens-thinking`。

**選配**：裝 [anysearch-skill](https://github.com/anysearch-ai/anysearch-skill) 作為 Local lens 在 WebSearch 結果稀薄時的垂直搜尋備援（求職、房地產、地方新聞）。

**3. 下次開機審記憶**

它會把上次寫好的候選 patch 拿出來請你 approve / reject / edit。**逐條**核可，不是整檔接受。被拒絕的條目會被記錄，系統不會再提同樣建議。

---

### 真實範例 — 2026-2029 黃金走勢

**你問**：

> 「基於當下(2026年五月)的地緣風險與宏觀經濟，分析未來三年黃金最有可能的走勢。要基於事實信息與數據的推導與分析。」

**四個鏡頭並行跑出**（壓縮重點）：

| Lens | 關鍵洞察 |
|------|---------|
| **Macro** | 三力疊加：央行非價格敏感買盤（WGC Q1'26 淨買 244t）+ 美財政無解（CBO FY26 赤字 1.9T / 5.8% GDP）+ BRICS Unit 結算層黃金化 → 基準目標 USD 5,500–7,000 / oz，年化 12–18%。 |
| **Personal** | 僅做 voice 校準（analytical 模式）。語氣：formal-analytical、mechanism-over-narrative。禁區：兩面下注、空泛趨勢詞。 |
| **Local** | AUD/USD 0.69–0.73（RBA 4.35% 鷹派 hold）→ AUD 計價漲幅相對 USD 漲幅打折 20–30%。ASX 槓桿路徑：EVN（淨現金）> NST（產量下修 + Elliott 壓力）。 |
| **Historical** | 最強類比是 **1971–1980**（布列敦森林崩解 + 儲備武器化），**不是** 2001–2011 QE 多頭。推論：結構性向上，但路徑中段預期至少一次 **25–35% 回檔**。 |

**你拿到的最終答案**：

> **基準情境**：結構性多頭延續到 2029，USD 目標區間 **5,500–7,000 / oz**（年化 12–18%）。路徑非單調，預期至少一次 25–35% 中段回檔。對 AUD 計價投資者而言，RBA 鷹派 hold 會把實質報酬吃掉 20–30%；最乾淨的槓桿路徑是有成本紀律的 ASX 金礦商（EVN > NST）。
>
> *三個破局訊號*：(1) PBOC 公布持倉跳升至 5,000+ 噸 (2) 西方 ETF 部位回到 2020 高點 (3) 30Y TIPS 殖利率突破 2.5% 同時金價仍創高
>
> *我會錯在哪*：Fed 被迫 YCC + 美元有序貶值 + 地緣緩和——央行買盤節奏放緩 30-40%，行情壓回循環性。

注意這份答案沒給你「為中位數讀者寫的版本」——它記得你在雪梨、用 AUD 計價、相關工具是 ASX 金礦商。這就是「為你而寫」的差別。

---

### 為什麼這樣設計

通用 AI 的問題不是「不夠聰明」，是**架構**。

你問黃金走勢，模型同時被要求：做宏觀分析、找歷史類比、抓在地現實、貼合你的個人限制——全塞在一個 prompt 裡。結果是**框架互相污染**：宏觀段落該講 USD 卻開始算 AUD；歷史類比因你的風險偏好而偏移，不再嚴格聚焦在機制本身。答案*看起來*個人化，其實是所有框架同時妥協後的「中位數」。

直覺的解法是塞更多 context——但問題只會更嚴重。

multi-lens-thinking 的設計前提就一句：**讓每個分析維度在自己的 sub-agent、自己的 context 裡獨立運行，最後再綜合。** 宏觀只想宏觀，歷史只想歷史，個人鏡頭只負責確保答案對得起你——彼此不滲透。Synthesizer 收到四份乾淨輸出後，把意見不合的地方攤開，給你看加權邏輯，而不是和稀泥成「綜合來看」一句廢話。

---

### 它怎麼運作

```
問題
  ↓
[Router]  ← 一次性讀 persona.md + memory.md
  輸出: { question_type, answer_mode, user_snapshot,
         search_hints, active_nodes, skip_log }
  ↓
  ├─ Macro       (地緣與資本流)        → WebSearch
  ├─ Personal    (你的背景)              (不搜尋)
  ├─ Local       (在地現實)            → WebSearch / anysearch
  └─ Historical  (歷史機制類比)        → WebSearch + LLM
  ↓
[Synthesizer]  ← 四個 lens 輸出 + answer_mode + user_snapshot + skip_log
  ↓
回覆給使用者
  ↓
[Memory Updater] → patches/YYYYMMDD-HHMMSS.md（下次 session 確認後合入）
```

**分流器（Router）** 讀你的 `persona.md` + `memory.md` 一次，決定兩件事：

- **啟動哪些鏡頭**——*積極跳過；多數問題不需要全部四個*
- **輸出哪個 `answer_mode`**——`analytical` / `personal_decision` / `framework` / `meta`

四個鏡頭接著**平行運行**（延遲 = 最慢的鏡頭，不是總和）：

| Lens | 在看什麼 | 是否搜尋？ |
|------|---------|-----------|
| **Macro** 宏觀 | 地緣、資本流、央行行為、貨幣秩序 | 是（網路） |
| **Personal** 個人 | 你的背景、限制、過往判斷 | 否——讀你，不讀世界 |
| **Local** 在地 | 你所在地的現實：價格、法規、可用工具、稅務 | 是（網路 / anysearch） |
| **Historical** 歷史 | 解釋當下「機制」的歷史類比——不是表面相似 | 是（網路 + LLM） |

**綜合器（Synthesizer）** 收四份輸出加 Router 的 mode，明確處理視角間的分歧。

**一個值得提防的失效模式**：分析師問「分析黃金走勢」——模型讀到 prompt 裡的職業背景，回的是一份**寫作簡報**而非分析本身。修法：`answer_mode` 由**問題動詞**決定，不由你的身份決定。「分析黃金」永遠輸出 `analytical` 模式，即使你是專業寫作者。Personal lens 在 `analytical` 模式下縮為語氣校準；Synthesizer 被明確禁止輸出「你應該寫一篇⋯⋯」這類元評論。

---

### 不會說謊的記憶迴圈

```
Session N         → Memory Updater → patches/YYYYMMDD-HHMMSS.md（候選）
                                              ↓
Session N+1 Step 0 → 「approve / reject / edit?」 → memory.md（批准後合入）
```

每次對話結束後，Memory Updater 寫一份**候選 patch**——擬加入 `memory.md` 的條目。下次對話開始時你逐條審閱。被拒絕的條目會被記錄，系統不會再提同樣建議。

這個設計**刻意慢**。自動更新的記憶會漂移；幾週的 AI 對話可能讓系統對你的認知悄悄走偏。Patch 確認機制讓你掌控系統學什麼，又不需要每次對話結束後手動維護背景檔案。

你的 `persona.md` 與 `memory.md` 只在本機。公開 repo 只有 pipeline 邏輯本身。

---

### 檔案結構

```
multi-lens-thinking/
├── SKILL.md              ← skill manifest 與執行程序
├── README.md             ← 英文版
├── README.CN.md          ← 本檔
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

---

### 成本與延遲

| 情境 | 延遲 | Tokens |
|------|------|--------|
| 四個鏡頭全跑 + 搜尋 | ~30–60s | ~8–20k |
| 跳過比例高（只啟動 1–2 個鏡頭） | ~10–20s | ~2–6k |
| Router 判定無鏡頭值得跑 | ~3s | ~1k |

Router 的「積極跳過」規則是刻意的：大多數「X 是不是好主意」的問題不需要歷史類比，多數 how-to 問題不需要任何鏡頭。

---

### 客製化

- **改某個鏡頭的 prompt**：編輯 `prompts/02-*.md` 到 `prompts/05-*.md`，各自獨立。
- **新增一個鏡頭**：新增 `prompts/0N-<name>.md`，更新 `SKILL.md` 的 pipeline 流程圖與 Router 的 `active_nodes` enum，並在 `06-synthesizer.md` 的輸入清單加入新節點。
- **跳過規則的鬆緊**：編輯 `prompts/01-router.md` 的 "Decision rules for active_nodes" 段落。
- **最終答案的聲音**：編輯 `prompts/06-synthesizer.md` 的 "Language" 段落。

---

### Fork 並發布到自己 GitHub

這個 skill 設計成可安全 fork——你的真實 `persona.md` / `memory.md` 只在本機，公開 repo 只有 templates。

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

---

### 已知限制

- Sub-agent 有獨立 context，看不到你前面的對話。如果問題依賴對話歷史，請在問題中帶入。
- WebSearch 品質依主題而異。Macro 最依賴搜尋，冷門主題可用 anysearch 的垂直搜尋補強。
- Memory Updater 故意保守。想立刻記住某個事實，直接編輯 `memory.md` 即可，不必走 patch 流程。
- 成本隨啟動的鏡頭數量線性成長。問題越具體，Router 越積極跳過、整體越省。

---

### 授權

MIT。詳見 `LICENSE`。
