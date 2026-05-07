# 如何建立自訂 Skill 並套用至全域

## 問題背景

在 Claude Code 中，自訂的斜線指令（slash command / skill）必須放在特定目錄下才能被識別。  
常見錯誤是將指令檔案放在專案根目錄的 `commands/` 資料夾，這樣 Claude Code **無法**讀取。

---

## 目錄結構規則

Claude Code 只會從以下兩個位置讀取自訂指令：

| 範圍 | 路徑 | 說明 |
|------|------|------|
| 全域（所有專案皆可用） | `~/.claude/commands/` | 存放於使用者家目錄 |
| 專案層級（僅限當前目錄） | `.claude/commands/` | 存放於專案的 `.claude/` 資料夾內 |

> **重要**：專案根目錄下的 `commands/`、`.claude-plugin/commands/` 等路徑均**不會**被自動載入。

---

## 建立全域 Skill 的步驟

### Step 1：建立全域指令目錄

```bash
mkdir -p ~/.claude/commands
```

### Step 2：建立指令檔案

檔名即為斜線指令名稱，副檔名為 `.md`。

```bash
# 範例：建立 /my-skill 指令
touch ~/.claude/commands/my-skill.md
```

### Step 3：撰寫指令內容

指令檔案為 Markdown 格式，支援 YAML frontmatter 設定描述。

```markdown
---
description: 這裡填寫指令的簡短說明（會顯示在 skill 清單中）
---

你的指令提示詞寫在這裡。

使用者的輸入會透過 $ARGUMENTS 傳入：
使用者請求：$ARGUMENTS
```

#### 欄位說明

| 欄位 | 必填 | 說明 |
|------|------|------|
| `description` | 否 | 顯示在 `/` 指令清單中的說明文字 |
| `$ARGUMENTS` | 否 | 接收使用者在斜線指令後輸入的內容 |

---

## 完整範例：/quant-researcher

以下是本次實作的量化研究員指令範例。

**檔案位置**：`~/.claude/commands/quant-researcher.md`

```markdown
---
description: 量化策略研究員：產生、設計、分析與回測量化交易策略
---

你是一位嚴謹的量化策略研究員。根據使用者的請求，依序完整執行以下四個階段。

使用者請求：$ARGUMENTS

## Phase 1：策略理論推論
...

## Phase 2：文獻佐證
...

## Phase 3：程式碼回測
...

## Phase 4：策略摘要報告
...
```

---

## 套用至全域的指令

```bash
# 1. 確認目錄存在
mkdir -p ~/.claude/commands

# 2. 複製你的指令檔案至全域目錄
cp /your/project/commands/my-skill.md ~/.claude/commands/my-skill.md

# 3. 確認檔案存在
ls ~/.claude/commands/
```

---

## 驗證是否生效

重新開啟一個 Claude Code session，輸入 `/` 後應可在清單中看到你的指令。  
或直接輸入：

```
/quant-researcher 設計一個動量策略
```

---

## 常見錯誤對照

| 錯誤做法 | 正確做法 |
|----------|----------|
| 放在 `commands/my-skill.md`（專案根目錄） | 放在 `~/.claude/commands/my-skill.md` |
| 放在 `.claude-plugin/commands/my-skill.md` | 放在 `~/.claude/commands/my-skill.md` |
| 放在 `skills/my-skill/SKILL.md` | 放在 `~/.claude/commands/my-skill.md` |
| 忘記建立 `~/.claude/commands/` 目錄 | 先執行 `mkdir -p ~/.claude/commands` |

---

## 附錄：專案層級 Skill（僅限當前專案）

若只想在特定專案中使用，改放在專案的 `.claude/commands/` 目錄：

```bash
mkdir -p .claude/commands
cp commands/my-skill.md .claude/commands/my-skill.md
```

此指令只在該專案目錄下的 Claude Code session 中可見，不影響其他專案。
