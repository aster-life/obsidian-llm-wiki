# Obsidian LLM Wiki 懶人包

受 [Andrej Karpathy 的 LLM Wiki 模式](https://github.com/karpathy/llm.sh) 啟發，打造一套可以讓 AI 幫你蒐集、整理、產出知識的 Obsidian 工作流。

把原始文章丟進去 → AI 自動摘要建檔 → 想查什麼直接問 → 一鍵整理成社群貼文。

---

## 架構概念

```
raw/          ← 你負責丟，AI 只讀不動
wiki/         ← AI 完全擁有，自動維護
outputs/      ← 貼文草稿產出
.claude/      ← AI 行為設定與技能包
```

**三個層次，分工明確：**
- **Raw 層**：你的原始素材（文章、PDF、筆記）
- **Wiki 層**：AI 自動整理的知識庫（來源摘要、概念、實體、跨來源分析）
- **Output 層**：從知識庫產出的社群貼文

---

## 包含內容

| 項目 | 說明 |
|---|---|
| `CLAUDE.md` | AI 行為契約，定義 7 種操作模式 |
| `.claude/skills/humanizer-zh/` | 中文去 AI 腔工具，貼文自動套用 |
| `.claude/skills/obsidian-markdown/` | 教 AI 正確使用 Obsidian Markdown |
| `.claude/skills/obsidian-bases/` | 教 AI 操作 Obsidian Bases |
| `.claude/skills/obsidian-cli/` | 教 AI 使用 Obsidian CLI |
| `.claude/skills/json-canvas/` | 教 AI 建立 Canvas 圖 |
| `.claude/skills/defuddle/` | 網頁內容清理工具 |
| `wiki/` | 空白知識庫模板 |
| `raw/` | 素材資料夾（空） |
| `outputs/` | 貼文產出資料夾（空） |

---

## 安裝步驟

### 1. 下載懶人包

```bash
git clone https://github.com/YOUR_USERNAME/obsidian-llm-wiki.git
```

或直接下載 ZIP，解壓縮。

### 2. 移入 Obsidian Vault

將所有檔案放到你的 Obsidian vault 根目錄（或新建一個 vault）。

### 3. 安裝 Claude Code

前往 [claude.ai/code](https://claude.ai/code) 安裝 Claude Code CLI。

### 4. 安裝 Claudian 插件（可選，但推薦）

讓你在 Obsidian 裡直接跟 AI 對話，不用切換視窗。

在 Obsidian 設定 → Community Plugins，搜尋 **Claudian** 安裝，  
或手動下載 [最新 release](https://github.com/YishenTu/claudian/releases) 的三個檔案放到 `.obsidian/plugins/claudian/`。

### 5. 設定個人風格（重要）

打開 `CLAUDE.md`，找到「個人風格規範」節，依照你的身份定位填寫：
- 你是誰（身份定位）
- 你的受眾是誰
- 你主要用哪個平台發文
- 你想要什麼語氣風格

### 6. 安裝 NotebookLM MCP（可選，用於自動生成配圖）

需要先安裝 [uv](https://docs.astral.sh/uv/getting-started/installation/)，然後：

```bash
uv tool install notebooklm-mcp-cli
nlm login
```

---

## 使用方式

在 Claude Code 或 Claudian 插件中，直接用中文說：

### 攝入文章
```
ingest raw/assets/你的文章.md
```

### 查詢知識庫
```
根據我的知識庫，幫我整理 XXX 概念
```

### 掃描未處理檔案
```
掃描 raw
```

### 寫社群貼文
```
幫我把 XXX 寫成 FB 貼文
```

### 跨來源分析
```
reflect
```

### 知識庫健康檢查
```
lint
```

---

## 7 種操作模式

| 操作 | 觸發詞 | 說明 |
|---|---|---|
| INGEST | `ingest`、`攝入` | 將 raw/ 文章摘要進 wiki/ |
| QUERY | 直接提問 | 從知識庫查詢並合成答案 |
| SCAN | `掃描 raw` | 列出未處理的 raw/ 檔案 |
| REFLECT | `reflect` | 跨來源二階合成分析 |
| LINT | `lint` | 知識庫健康檢查 |
| ADD QUESTION | `我想搞清楚` | 記錄開放問題到佇列 |
| PUBLISH | `幫我寫貼文` | 從知識庫產出社群貼文 |

---

## 貼文產出流程

```
PUBLISH 觸發
    ↓
AI 詢問你的真實踩坑經驗
    ↓
產出草稿（PAS 結構）
    ↓
自動套用 humanizer-zh（去 AI 腔 + 品質評分）
    ↓
你確認標題 + 內容
    ↓
存檔 + 自動生成 NotebookLM 資訊圖（若有安裝）
```

---

## 致謝

- [Andrej Karpathy](https://github.com/karpathy) — LLM Wiki 概念發想
- [kepano](https://github.com/kepano) — obsidian-skills 技能包
- [YishenTu](https://github.com/YishenTu) — Claudian 插件
- [WikiProject AI Cleanup](https://en.wikipedia.org/wiki/Wikipedia:WikiProject_AI_Cleanup) — humanizer-zh 的 24 種模式研究基礎

---

## License

MIT
