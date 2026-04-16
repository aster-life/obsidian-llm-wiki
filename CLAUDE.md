# CLAUDE.md — LLM Wiki 行為契約

> 這是 Claude 的行為規範。每次對話開始時自動讀取。
> Vault 路徑：請將此檔案放在 Obsidian vault 根目錄

---

## 系統架構

```
raw/              ← 人類擁有，LLM 只讀不改
  assets/         ← 原始文章、圖片、PDF 等素材
  personal/       ← 自己寫的文章（ingest 行為不同）
wiki/             ← LLM 完全擁有，負責維護
  sources/        ← 每篇來源的摘要頁
  concepts/       ← 核心概念頁（方法論、技術名詞，含 Evolution Log）
  entities/       ← 實體頁（人物、工具、公司、論文）
  synthesis/      ← 跨來源合成分析（含升級的高品質 Query 答案）
  index.md        ← 總索引
  log.md          ← 操作日誌（只追加）
  QUESTIONS.md    ← 開放問題佇列
.claude/
  skills/         ← AI 技能包（obsidian-markdown 等）
```

**核心原則：**
- Raw 層不可變 — 原始來源絕對不修改
- Wiki 層 LLM 完全擁有 — 你只瀏覽，不編輯
- 寫入操作都記日誌 — INGEST、REFLECT 等寫入操作追加到 wiki/log.md；QUERY 為讀取操作，不強制記錄
- 矛盾必須顯式標注 — 來源分歧明確記錄

---

## 使用者資訊

- 主題：（填入你主要蒐集的知識領域，例如：AI 工具、行銷科技、產品設計）
- 語言：繁體中文為主，技術術語保留英文
- 標籤（tags）：英文小寫，如 `ai-agent`、`llm`、`rag`

---

## 操作一：INGEST（攝入）

**觸發詞：** `ingest`、`攝入`、`處理這個`

### 來源類型判斷

| 條件 | 流程 |
|---|---|
| 路徑含 `raw/personal/` | 個人寫作流程 |
| 其他 raw/ 檔案 | 標準外部來源流程 |

### 查重規則（執行前必做）

執行 INGEST 前，先比對以下兩處：
- `wiki/index.md` Raw 清單是否標記「✅ 已 ingest」
- `wiki/sources/` 是否已有 `raw_file` 欄位指向同一檔案

| 情況 | 處理方式 |
|---|---|
| 確認重複 | 提示「已在 wiki/sources/xxx」，詢問是否要更新，不自動擋住 |
| 確認未處理 | 直接進入 INGEST 流程 |

### 標準外部來源流程（8 步）

1. 讀取 raw/ 中的目標檔案（只讀）
2. 一次列出所有核心要點，請用戶確認（不要逐一慢慢問，保持流暢）
3. 生成 slug（英文小寫連字符，如 `attention-is-all-you-need`）
4. 建立 `wiki/sources/<slug>.md`，寫入來源摘要
5. **概念與實體對齊檢查**：
   - **Concepts**（方法論、技術名詞）→ 查 wiki/concepts/ 是否已有對應頁面（含 aliases 比對）；有則更新，無則新建
   - **Entities**（人物、工具、公司、論文）→ 查 wiki/entities/ 是否已有對應頁面；有則更新，無則新建
6. 更新或新建 concept/entity 頁面，追加 Evolution Log
7. 更新 `wiki/index.md`（將來源加入 Sources 表格，更新 Raw 待處理清單）
8. 追加 `wiki/log.md`：
   `## [YYYY-MM-DD] ingest | [來源標題] | concepts: N, entities: M, links: Z`

若來源發表超過 2 年前，在 source 頁標注 `possibly_outdated: true`。

ingest 完成後，檢查 wiki/QUESTIONS.md 是否有本次來源能回答的問題，若有則提示用戶。

### 個人寫作流程（raw/personal/）

`raw/personal/` 收錄兩種內容，處理邏輯不同：

**類型一：讀後心得**（有明確對應的來源或概念）
- 不生成 Summary，跳過客觀摘要
- 將核心論點寫入相關 concept 頁的 `## My Position` 節（標注「個人觀點」）
- **不計入 source_count**（避免用自己的文章給自己背書）
- Evolution Log 記錄：「YYYY-MM-DD 個人寫作 [[slug]] 確立了對此概念的明確立場」

**類型二：靈感與想法**（獨立、模糊、或全新）
- 只存放於 `raw/personal/`，不強制對應現有 concept
- 不寫入任何 wiki 頁面，保留原始性
- 若日後靈感成熟或反覆出現，用戶主動說「ingest 這篇靈感」，再建立新 concept 或補入 My Position

### 缺少 frontmatter 時

- 從第一個 `#` 標題提取 title；若無則從檔名推斷
- 不中斷 INGEST，但在 log.md 記錄警告

---

## 操作二：QUERY（查詢）

**觸發詞：** 與知識庫主題相關的提問，或明確說「根據我的知識庫」。一般閒聊、操作指示、系統設定等不觸發 QUERY。

**執行步驟：**
1. 讀取 wiki/index.md，識別最相關的 3-5 個頁面
2. 完整讀取 concepts/ + sources/ 摘要頁
3. **自動判斷是否需要補讀 raw/：**
   - 論點需要具體數據、統計、案例 → 主動讀取對應 `raw_file` 原始檔補充
   - 摘要描述模糊或資訊不足 → 主動讀取 raw/ 原始檔
   - 純概念比較或架構說明 → 摘要頁已足夠，不需補讀
4. 合成答案，每個核心結論必須溯源到具體的 `wiki/sources/<slug>.md`（不允許只引用 concept 頁）
5. 標注各來源的 confidence 等級；來源相互矛盾時顯式標注分歧
6. 輸出末尾包含「⚠ Confidence Notes」節
7. **升級判斷**：若答案涉及 3 個以上來源、具備跨來源合成價值 → 詢問用戶是否升級為 `wiki/synthesis/<topic>-synthesis.md`
8. **選用存檔**：若用戶說「存起來」或「記錄這個答案」，才追加 wiki/log.md；否則只在對話中回答

**輸出格式根據問題類型：**
- 普通問題 → Markdown 正文
- 比較類 → Markdown 表格
- 清單類 → 結構化 bullet list

---

## 操作三：SCAN（掃描未處理檔案）

**觸發詞：** `掃描 raw`、`有哪些還沒 ingest`、`raw 裡有什麼`

**執行步驟：**
1. 列出 `raw/assets/` 與 `raw/personal/` 的所有檔案
2. 與 `wiki/index.md` Raw 清單比對
3. 已 ingest 的自動略過，不暫停、不詢問
4. 輸出兩份清單：
   - ✅ 已處理（略過）
   - 📋 待處理（列出檔名，供用戶選擇下一步）

---

## 操作四：REFLECT（二階合成）

**觸發詞：** `reflect`、`綜合分析`、`發現規律`

**四階段執行：**

- **Stage 0（反向檢驗）：** 生成任何合成結論前，主動搜尋反駁證據。若無，在 Limitations 節標注「⚠ 回音室風險」
- **Stage 1（模式掃描）：** 讀取 wiki/index.md，識別跨來源模式、隱性關聯、內容空白、矛盾對
- **Stage 2（深度合成）：** 對有證據支撐的候選項，寫入 `wiki/synthesis/<topic>-synthesis.md`
- **Stage 3（Gap Analysis）：** 找出 source_count = 1 且建立超過 30 天的孤立概念；找出多篇 sources 提及但無獨立頁面的概念；直接在對話中輸出報告

完成後：更新 wiki/index.md 的 Synthesis 表格，追加 wiki/log.md。

**建議頻率：** 每月一次，或每積累 10 篇新來源後一次。

---

## 操作五：LINT（健康檢查）

**觸發詞：** `lint`、`健康檢查`

**LLM 直接執行以下檢查（不需要腳本）：**
1. Broken wikilinks：`[[xxx]]` 引用了不存在的頁面
2. Stub 頁面：正文少於 100 字的空殼頁面
3. Index 一致性：wiki/index.md 中標記的檔案是否都實際存在
4. Stale 概念：超過 domain_volatility 時效未更新（high=90天, medium=180天, low=365天）
5. 矛盾未標注：多個 sources 立場相反但 Contradictions 節為空

輸出結果後詢問用戶是否要修復。

**建議頻率：** 每兩週一次。

---

## 操作六：ADD QUESTION（記錄問題）

**觸發詞：** `我想搞清楚`、`add question`、`記錄一個問題`

**執行步驟：**
1. 將問題規範化（提取核心疑問）
2. 追加到 `wiki/QUESTIONS.md`（格式：`- [ ] 問題內容（opened YYYY-MM-DD）`）
3. 追加 wiki/log.md

---

## Wikilink 規範

**格式鐵律：** 所有 wikilink 目標使用英文小寫連字符
- ✅ `[[ai-agent]]` `[[chain-of-thought]]`
- ❌ `[[AI Agent]]` `[[鏈式思考]]`

**連結必須附上關係說明：**
- ✅ `[[chain-of-thought]] — CoT 是 AI Agent 決策時最常用的推理框架`
- ❌ `[[chain-of-thought]]`（裸連結）

**禁止用 wikilink 引用系統文件：** log.md、index.md、QUESTIONS.md

---

## Confidence 規則

| 等級 | 條件 | 說明 |
|---|---|---|
| `low` | 1-2 個來源 | 線索，標注不確定性 |
| `medium` | 3+ 個來源，觀點一致 | 可引用，建議標注來源數 |
| `high` | **用戶親自確認** | 你的主動背書，不靠計數器 |

個人寫作（raw/personal/）不計入 source_count。

---

## 系統文件隔離規則

以下文件的 frontmatter 必須含 `graph-excluded: true`，不出現在 Obsidian 圖譜：
- wiki/log.md
- wiki/index.md
- wiki/QUESTIONS.md

---

## Concept 頁 Evolution Log 格式

```
## Evolution Log

- YYYY-MM-DD（N sources）：強化：來源 [[sources/slug]] 觀點一致
- YYYY-MM-DD（N sources）：修正：新增 [[sources/slug]] 指出 X 維度被忽略
- YYYY-MM-DD（N sources）：新增分歧：[[sources/slug]] 立場相反，見 Contradictions 節
```

---

## Source 頁 Frontmatter 範例

```yaml
---
type: source-summary
title: "文章標題"
date: YYYY-MM-DD
source_url: "https://..."
author: "作者"
tags: [wiki, wiki/source]
processed: true
raw_file: "raw/filename.md"
possibly_outdated: false
---
```

## Concept 頁 Frontmatter 範例

```yaml
---
type: concept
title: "概念中文名"
date: YYYY-MM-DD
updated: YYYY-MM-DD
tags: [wiki, wiki/concept]
aliases: ["中文名", "English Name", "縮寫"]
source_count: 0
confidence: low
domain_volatility: medium
last_reviewed: YYYY-MM-DD
---
```
