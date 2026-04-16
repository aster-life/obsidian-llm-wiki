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
outputs/
  posts/          ← 社群貼文產出
.claude/
  skills/         ← AI 技能包（humanizer-zh、obsidian-markdown 等）
```

**核心原則：**
- Raw 層不可變 — 原始來源絕對不修改
- Wiki 層 LLM 完全擁有 — 你只瀏覽，不編輯
- 寫入操作都記日誌 — INGEST、PUBLISH、REFLECT 等寫入操作追加到 wiki/log.md；QUERY 為讀取操作，不強制記錄
- 貼文輸出確認後持久化 — PUBLISH 確認後存入 outputs/posts/，不消失在對話中
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

## 操作七：PUBLISH（社群貼文產出）

**觸發詞：** 任何帶有社群發文意圖的說法，例如：
`幫我寫貼文`、`寫成社群貼文`、`發一篇關於...`、`整理成 FB 貼文`、`這篇可以發嗎`

**兩種入口：**

| 模式 | 觸發方式 | 適合情境 |
|---|---|---|
| Wiki 模式 | `幫我寫 [主題] 的貼文` | 已 ingest，從知識庫合成 |
| Raw 模式 | `幫我把 [raw檔路徑] 寫成貼文` | 單篇快速產出，尚未 ingest |

**執行步驟：**

1. **判斷入口**：
   - Wiki 模式 → 讀 wiki/index.md，找出相關 sources + concepts，自動判斷是否需補讀 raw/
   - Raw 模式 → 直接讀取指定 raw/ 檔案作為主要素材
2. **詢問用戶補充真實經驗**（問題一次問完，不要逐一慢慢問）。依內容類型調整提問方向：
   - 工具／方法類 → 當初為什麼選這個？最有感或最踩坑的是？現在怎麼用在工作流裡？
   - 觀念／洞察類 → 是什麼情境讓你有這個想法？有沒有反例或讓你猶豫的地方？
   - 案例／流程類 → 這個案例的背景是？過程中遇到最大的卡關是什麼？
3. **產出貼文草稿**，依照下方個人風格規範：
   - 先給吸睛標題（書名號包覆，如《XXX》），放在貼文最開頭
   - 完整踩坑歷程，300-600 字，PAS 結構
4. **人性化處理**：草稿產出後，自動讀取 `.claude/skills/humanizer-zh/SKILL.md` 與 `.claude/skills/humanizer-zh/references/`，套用 24 種模式掃描與重寫，輸出：
   - 人性化後的貼文版本
   - 變更摘要（說明修復了哪些 AI 模式）
   - 品質評分表（五維度 / 50 分）
5. **自行補強**：提出 1-2 個可強化的建議（例如：某段可加更具體的數字、某個轉折可更有畫面感），供用戶參考
6. **等待用戶確認**：用戶回覆「確認」或「OK」後，才執行以下步驟：
   - 將貼文存入 `outputs/posts/YYYY-MM-DD-<slug>.md`（frontmatter 含 `graph-excluded: true`）
   - 更新 wiki/index.md 的 Outputs 表格
   - 追加 wiki/log.md：`## [YYYY-MM-DD] publish | [主題] | 輸出: outputs/posts/xxx`
7. **自動生成配圖**（需安裝 NotebookLM MCP，可選）：
   - 呼叫 `notebook_create` 建立暫用筆記本，標題為貼文 slug
   - 呼叫 `source_add` 將貼文內容加入筆記本
   - 呼叫 `studio_create`（artifact_type: infographic）觸發生成
   - 輪詢 `studio_status` 直到完成
   - 呼叫 `download_artifact` 下載，儲存至 `outputs/posts/YYYY-MM-DD-<slug>-cover.png`
   - 若生成失敗，告知用戶並略過，不中斷整個流程

---

## 個人風格規範（PUBLISH 操作專用）

> 請依照你自己的身份定位與目標受眾填寫此節。以下為範例格式。

**身份定位：** （填入你的角色，例如：MarTech 顧問、產品設計師、獨立開發者）

**目標受眾：** （填入你的受眾，例如：非工程師的業主、行銷主管、創業者）

**平台：** （填入主要發文平台，例如：Facebook、Threads、LinkedIn）

**語氣與調性：**
- （描述你的寫作風格，例如：謙遜感性、像在咖啡廳跟朋友聊天）
- 有血有肉的踩坑歷程，不是概念分享
- 有情境、有走錯路、有轉折、有具體工具名稱

**鐵律（依個人需求自訂禁止事項）：**
- 禁止行銷廣告語氣：過度熱情、浮誇形容詞、任何銷售感
- 禁止說教口吻：「你應該這樣做」、「建議你」
- 禁止 AI 腔：「總之」、「深入探討」、「不得不說」

**文章結構（Facebook 版）：**
1. Hook：用真實情境或踩坑開場
2. 問題描述：具體說遇到什麼困境
3. 走錯的路：試過什麼方法、為什麼不適合
4. 轉折與發現：怎麼找到現在的解法
5. 核心洞察：1-2 個最有感的設計或功能
6. 適用情境：適合誰用、不適合誰用
7. 隨興 CTA：邀請留言或分享，語氣要輕鬆
8. Hashtag：2-4 個

**標題風格（四種可選）：**
- 痛點直說型：《每次叫 AI 改東西，都要重新解釋一遍——這不正常》
- 反問帶入型：《三個月後，你還記得自己叫 AI 做了什麼嗎？》
- 現象點破型：《你跟 AI 開發的需求，正在消失在聊天記錄裡》
- 兩難破局型：《用 Spec Kit 太重、不用又亂——我怎麼找到中間那條路》

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
- outputs/posts/ 下所有文件

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
