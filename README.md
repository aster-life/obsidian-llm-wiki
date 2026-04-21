# Obsidian LLM Wiki 懶人包

受 [Andrej Karpathy 的 LLM Wiki 模式](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f) 啟發，打造一套可以讓 AI 幫你蒐集、整理、累積知識的 Obsidian 工作流。

把原始文章丟進去 → AI 自動摘要建檔 → 跨來源合成分析 → 直接用中文問知識庫。

---

## 架構概念

```
raw/          ← 你負責丟，AI 只讀不動
wiki/         ← AI 完全擁有，自動維護
.claude/      ← AI 行為設定與技能包
```

**兩個層次，分工明確：**
- **Raw 層**：你的原始素材（文章、PDF、筆記）
- **Wiki 層**：AI 自動整理的知識庫（來源摘要、概念、實體、跨來源分析）

---

## 包含內容

| 項目 | 說明 |
|---|---|
| `CLAUDE.md` | AI 行為契約，定義 6 種操作模式 |
| `.claude/skills/obsidian-markdown/` | 教 AI 正確使用 Obsidian Markdown |
| `.claude/skills/obsidian-bases/` | 教 AI 操作 Obsidian Bases |
| `.claude/skills/obsidian-cli/` | 教 AI 使用 Obsidian CLI |
| `.claude/skills/json-canvas/` | 教 AI 建立 Canvas 圖 |
| `.claude/skills/defuddle/` | 網頁內容清理工具 |
| `wiki/` | 空白知識庫模板 |
| `raw/` | 素材資料夾（空） |

---

## 安裝步驟

### 1. 下載懶人包

```bash
git clone https://github.com/aster-life/obsidian-llm-wiki.git
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

### 跨來源分析
```
reflect
```

### 知識庫健康檢查
```
lint
```

---

## 6 種操作模式

| 操作 | 觸發詞 | 說明 |
|---|---|---|
| INGEST | `ingest`、`攝入` | 將 raw/ 文章摘要進 wiki/ |
| QUERY | 直接提問 | 從知識庫查詢並合成答案 |
| SCAN | `掃描 raw` | 列出未處理的 raw/ 檔案 |
| REFLECT | `reflect` | 跨來源二階合成分析 |
| LINT | `lint` | 知識庫健康檢查 |
| ADD QUESTION | `我想搞清楚` | 記錄開放問題到佇列 |

---

## 致謝

- [Andrej Karpathy](https://github.com/karpathy) — LLM Wiki 概念發想
- [kepano](https://github.com/kepano/obsidian-skills) — obsidian-skills 技能包
- [YishenTu](https://github.com/YishenTu) — Claudian 插件
- [LLM Wiki 搭建教程](https://hcn9zwu8a0fz.feishu.cn/wiki/AM3ewXySViopPdkE8Gic90BDnRb)
- [丹妮tech](https://www.youtube.com/watch?v=ohPBYGN8a9Y)
- [Blink 的 AI 笔记](https://www.youtube.com/watch?v=eUaL48Ph6DM&t=304s)
- [白白说大模型](https://www.youtube.com/watch?v=csGRKlNnnpU&t=384s)
- [PAPAYA 電腦教室](https://www.youtube.com/watch?v=Egj_DdGUIDI&t=85s)
- [三師爸Sense Bar](https://www.youtube.com/watch?v=b21TUOL1JdI)

---

## License

MIT
