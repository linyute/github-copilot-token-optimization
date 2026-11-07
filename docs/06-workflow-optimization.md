# 2.5 針對特定工作流程的最佳化

[← 回到指南](index.md)

---

## 2.5.1 提交訊息

使用 Conventional Commits，主旨長度不超過 50 個字元。若變更原因不明顯，請加上內文說明。

**冗長的提交訊息：**

```text
feat: 新增一項功能，讓使用者可以透過設定頁面重設密碼，
同時也會寄送一封確認電子郵件
```

**簡潔的提交訊息：**

```text
feat: 新增透過設定頁面重設密碼的功能

重設後會寄送確認電子郵件。
```

簡潔的提交訊息更容易瀏覽，也讓代理程式載入 git 歷史記錄時使用較少的 token。若原因重要，請保留。

## 2.5.2 PR 審查

指出問題、位置、後果與修正方式。若一行就能保留這些細節，一行即可。

**冗長的審查意見：**

```text
我注意到在第 42 行，此處的 user 變數可能為 null，
當您嘗試存取使用者的 email 屬性時，
會導致 NullPointerException。您應該在存取此屬性之前
加上 null 檢查，以妥善處理這個邊界情況。
```

**簡潔的審查意見：**

```text
L42: 🔴 bug: user 此處可能為 null。存取 .email 前加上 null 防護。
```

這組手寫範例展示的是較短的意見，並非實測的節省比例。

嚴重性標籤能讓優先順序更容易掃讀：

- 🔴 錯誤／安全性問題 — 必須修正
- 🟡 建議 — 應該修正
- 🔵 細節 — 可選的改進
- ❓ 問題 — 需要釐清

## 2.5.3 Ask 模式與 Agent 模式

**Agent 模式**可以進行多次模型與工具呼叫，以讀取、規劃、編輯與驗證。每個步驟都會增加 token 用量。

**Ask 模式**適合可由所提供的內容直接回答、不需要實作迴圈的問題。

![Copilot 模式決策樹：問題使用 Ask，單一檔案變更使用 Edit，明確的多檔案工作使用 Agent，模糊的工作則先釐清再執行。](../../source/docs/assets/diagrams/mode-decision-tree.svg)

| 任務 | 適合的模式 | 原因 |
|------|-----------|-----|
| 「這個函式在做什麼？」 | Ask | 單次回答。不需要使用工具 |
| 「TypeScript 泛型的語法是什麼？」 | Ask | 知識性問題 |
| 「更新這些已知檔案中的驗證邏輯」 | Edit（若可用） | 範圍明確且有明確內容的變更 |
| 「重構此模組以使用相依性注入」 | Agent | 多檔案變更，需要讀寫程式碼 |
| 「建立含測試與文件的 REST API」 | Agent | 多步驟的建立任務 |
| 「這個測試為什麼失敗？」 | Ask（通常） | 通常只需要錯誤訊息加上您提供的內容 |

Ask 可以避免不必要的工具迴圈。差異取決於 Agent 實際會載入與執行的內容；每個問題並沒有固定的節省比例。

### 進階 Copilot CLI 策略：CodeAct

對於**大量使用工具的 Copilot CLI 工作階段**，選用的外部外掛程式 [`copilot-codeact-plugin`](https://github.com/jsturtevant/copilot-codeact-plugin) 可以合併工具操作。代理程式不再於多個回合中交替進行模型與工具呼叫，而是撰寫一個 Python 程式，並在一次沙箱化的執行中跑完整個流程。

這可能節省 token 的原因：

- 回合減少，可能意味著提示詞、歷史記錄與已載入的工具定義（包括 MCP 目錄）的重複回放減少。
- 一份整合後的結果，通常比逐一敘述每次中間的 `grep` / `view` / `bash` 步驟更短。

適用時機：

- 以 CLI 為主的探索或稽核任務，否則會在許多小型工具呼叫之間來回
- 已載入 MCP、重複回放結構描述的成本已經很高的工作階段
- 可重複執行的分析任務，例如 TODO 掃描、函式索引、涵蓋率檢查或交叉參照蒐集

不適用時機：

- 簡單的單次 Ask 問題
- 該外掛程式不適用的一般 IDE 聊天／編輯工作流程
- 不希望在工作流程中使用外部外掛程式的團隊

該外掛程式的 README 回報了其自身基準測試提示詞的節省情形，包括已載入 MCP 的案例。本指南並未獨立對 CodeAct 進行基準測試；這些結果無法證明每項任務都能節省。

### 互補方案：使用 RTK 或 snip 壓縮工具輸出

CodeAct 減少的是工具呼叫的*次數*。[**RTK (Rust Token Killer)**](https://github.com/rtk-ai/rtk) 與 [`snip`](https://github.com/edouard-claude/snip) 減少的則是每個 shell 工具結果的*大小*。它們處理的是同一問題的不同面向，可以搭配使用。

這些工具會在 `git`、測試執行器、`grep`、`ls` 與其他命令的結果送達代理程式之前先行過濾。縮減幅度取決於命令與輸出內容。與 CodeAct 不同，這並不僅限於 Copilot CLI；只要 shell hook 可靠，就能發揮作用。請將 Windows 與預覽版的 hook 路徑視為試行項目。設定方式請參閱 [MCP 與工具成本 §2.7.7](08-mcp-tool-costs.md#277-compress-tool-output-at-the-source-rtk) 與 [§2.7.8](08-mcp-tool-costs.md#278-rtk-alternative-snip)。

## 2.5.4 預設使用自動模型選擇

模型與推理強度的選擇，請參閱[模型選擇與定價](11-models-and-pricing.md#good-default-heuristics)。快取穩定性與交接，請遵循[上下文管理](04-context-management.md#235-caching-store-and-reuse-context-within-prompts)。

## 2.5.5 依目標模型重新調校提示詞

重新調校提示詞不一定會縮短每次請求。其目的是改善第一次的結果，並避免後續回合、重複釐清與重工。

模型供應商會發布隨模型版本變動的提示詞指南。當升級改變了提示詞的行為時，請參考供應商的遷移或提示詞指南，只調整受影響的指示。

**工作流程：**

```text
開啟目標模型的官方提示詞指南。
將 URL 貼到 Copilot 聊天中。
詢問：「依照此指南調整這些目標檔案。保持行為不變。減少重工。」
目標檔案：.github/copilot-instructions.md, .github/instructions/*.instructions.md, agents/*.md, app prompt files。
檢視差異。只保留可衡量且與模型相關的變更。
```

官方起點：

| 供應商 | 模型系列 | 提示詞指南 |
|---|---|---|
| Anthropic | Claude Sonnet / Opus / Haiku | [提示工程概觀](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/overview)與 [Claude 最新模型最佳做法](https://platform.claude.com/docs/en/docs/build-with-claude/prompt-engineering/claude-4-best-practices) |
| OpenAI | GPT-5.5 / GPT-5 | [GPT-5.5 提示詞指南](https://developers.openai.com/api/docs/guides/prompt-guidance)與 [GPT-5 提示詞指南](https://cookbook.openai.com/examples/gpt-5/gpt-5_prompting_guide) |
| Google | Gemini | [Gemini 提示詞設計策略](https://ai.google.dev/gemini-api/docs/prompting-strategies) |

### 範例：一個基礎指示，三種調校方式

基礎指示：

```markdown
你是一位程式碼助理。協助完成實作。回答簡潔。必要時提問。遵循儲存庫風格。執行測試。
```

針對特定模型的改寫：

| 目標模型 | 調校後的指示 | 適用原因 |
|---|---|---|
| **Claude Sonnet** | `角色：資深儲存庫工程師。\n有幫助時使用類 XML 區段：<task>、<constraints>、<done>。\n編輯前：只檢查相關檔案。保留既有風格。\n對於模糊的請求：只有在選擇會改變實作時才提問。\n完成 = 已套用修補 + 既有的目標測試通過，或已說明阻礙。` | Claude 指南強調明確的成功標準、範例／結構、明確的工具／使用界線，以及經過校準的投入程度。XML 風格的分隔符號通常有助於區隔任務、上下文與限制。 |
| **GPT-5.5** | `成果：以最小變動完成正確的儲存庫變更。\n成功：目標行為可運作、差異範圍受控、回報測試結果或確切的阻礙。\n選擇有效率的路徑；不要過度規範流程。\n以一則簡短的進度更新開始大量使用工具的工作。\n只針對會影響成果或安全性的缺漏資訊提問。` | GPT-5.5 指南偏好以成果為導向的提示詞、簡潔的個性／協作規則、有效率的解決路徑、多步驟工作的可見前言，並避免舊式的過度規範。 |
| **Gemini** | `任務：實作所要求的儲存庫變更。\n上下文：使用所參照的檔案、鄰近的測試與儲存庫慣例。\n限制：輸出簡潔、差異範圍受控、不進行無關的重寫。\n最終格式：變更的檔案 + 行為影響 + 測試結果／阻礙。\n若輸入不完整，請說明一項所需的細節。` | Gemini 指南強調清楚的任務／輸入／限制／回應格式結構。明確的格式與上下文界線可減少詮釋上的偏差。 |

### 給 Copilot 的使用者提示詞範例

```text
目標模型：GPT-5.5。
指南：https://developers.openai.com/api/docs/guides/prompt-guidance
檔案：.github/copilot-instructions.md, agents/token-saver.agent.md
依指南調整提示詞。保持行為不變。減少重複釐清。保持簡潔。
只顯示差異。
```

適用於以下情況：

- 您升級或變更了預設模型。
- 某個提示詞在一個模型上運作良好，但換到另一個模型後變得冗長、怠惰、過度積極或太拘泥字面。
- 模型變更後，代理程式一再做出相同的錯誤假設。
- 您維護應用程式提示詞、代理程式設定檔或可重複使用的指示檔案。

當提示詞行為已經過衡量且穩定時，請避免這麼做。在沒有失敗訊號的情況下變更指示，可能只會增加雜訊。

## 2.5.6 何時不該壓縮

壓縮有其限度。某些情況需要完整清楚的表達：

- **安全性警告** — 「這將刪除所有使用者資料」不得縮寫成「del usr data」
- **不可逆的操作** — 確認提示必須明確無誤
- **新進人員引導情境** — 新進團隊成員需要知道「為什麼」，而不只是「是什麼」
- **複雜的多步驟指示** — 片段的順序可能導致誤讀時
- **法規／合規文字** — 法律要求必須精確

請告知精簡範本或代理程式設定檔，在安全性警告與不可逆操作的確認中使用完整且明確的措辭。

## 2.5.7 使用 `/chronicle` 形成回饋迴圈

重複的誤解所浪費的 token，可能比一個冗長的提示詞更多。Copilot CLI 的 [`/chronicle`](https://docs.github.com/en/copilot/concepts/agents/copilot-cli/chronicle) 命令會分析本機工作階段歷史記錄，找出混淆、重複修正與其他值得改善的模式。

> **範圍：** `/chronicle` 是 **Copilot CLI** 的功能，以 `~/.copilot/session-state/` 中的本機工作階段歷史記錄為基礎。它可在 Copilot CLI 互動式工作階段中執行，也能透過互動式 Copilot CLI 工作階段在 **JetBrains IDE** 內執行。它**不**適用於 **VS Code**；若要取得 VS Code 的使用情況分析，請參閱下方的 [AI Engineering Coach](#258-vs-code-usage-analytics-ai-engineering-coach)。
>
> **可用性：** `/chronicle` 目前為實驗性功能。請在 Copilot CLI 互動式工作階段中使用 `/experimental on` 啟用，或在命令列傳入 `--experimental`。

子命令包括 `standup`、`tips`、`cost tips`、`search`、`improve` 與 `reindex`。針對成本與工作流程審查：

| 命令 | 功能 | 節省 token 的效益 |
|---------|--------------|---------------------|
| **`/chronicle cost tips`** | 檢視近期工作階段中的提示詞長度、工具呼叫頻率與後續步驟 | 直接的成本建議 |
| **`/chronicle improve`** | 找出重複的修正，並**產生自訂指示片段** | 可能預防反覆出現的誤解 |
| **`/chronicle tips`** | 根據您的使用情況建議功能與工作流程變更 | 可能找出更好的模式、模型或上下文選擇 |
| **`/chronicle standup`** | 從工作階段資料彙整分支、PR 與狀態 | 節省人工彙整的工作，而非直接節省 token |

### `cost tips` 工作流程

檢視 token 的去向：

```text
/chronicle cost tips
```

根據其針對各工作階段的建議，決定要改變哪些習慣。

### `improve` 工作流程

當相同的誤解反覆發生時使用：

```text
/chronicle improve
```

Copilot CLI 會根據近期工作階段中反覆出現的混淆（例如誤以為使用了錯誤的測試框架）提出指示。請接受專案特定的修正；略過樣板內容。要保留哪些內容，請參閱[第 7 部分：AGENTS.md 的問題](07-agents-md-problem.md)。

每次修正都會增加後續輸入、回放的歷史記錄與新的輸出。只有在針對反覆出現的失敗時，才加入可重複使用的指引。

### `tips` 工作流程

尋找工作流程建議：

```text
/chronicle tips
```

保留符合您工作流程的建議，例如使用 Ask 模式處理說明、以 `applyTo` 限定指示範圍，或停用未使用的 MCP 伺服器。

所有工作階段資料都儲存在本機的 `~/.copilot/session-state/`，且僅存在於您的電腦上。這是 **Copilot CLI 的工作階段資料**，並非通用的 Copilot Chat 歷史記錄儲存區。執行 `/chronicle` 命令時，仍適用標準的模型互動（資料會傳送給模型以產生摘要），但不會上傳任何內容進行儲存。

## 2.5.8 VS Code 使用情況分析：AI Engineering Coach

針對 VS Code 的歷史記錄，[**AI Engineering Coach**](https://github.com/microsoft/AI-Engineering-Coach) 會讀取本機 AI 工作階段記錄，找出反覆出現的錯誤、token 模式、上下文問題與可重複使用的技能。

> **隱私：** 本機分析為唯讀，不會將工作階段資料傳送到機器之外。選用的 AI 功能（例如規則編譯與上下文審查）在明確叫用時，會使用 VS Code 內建的 Copilot 模型 API；請勿假設這些功能可離線運作。

與 token 效率相關的主要功能：

| 功能 | 呈現的內容 |
|---------|-----------------|
| **Anti-Patterns** | 涵蓋提示詞品質、工作階段衛生、程式碼審查、工具熟練度與上下文管理的 45 條可編輯規則，並附有嚴重性等級與具體的修正動作 |
| **Context Health** | 代理程式就緒檢查清單、工作區上下文對應圖與指示檔案稽核 |
| **Skill Finder** | 偵測您歷史記錄中重複的提示詞模式，並比對開放原始碼目錄中可重複使用的技能 |
| **Output / Burndown** | 依語言與模型統計的 AI 產生程式碼量；附帶預測的 token 預算進度 |

**快速入門：**

```bash
git clone https://github.com/microsoft/ai-engineering-coach.git
cd ai-engineering-coach
npm install && npm run package
code --install-extension ai-engineer-coach-*.vsix
```

接著按 `Cmd+Shift+P` → **AI Engineer Coach: Open Dashboard**。

## 2.5.9 先規劃，再執行（並為各階段選擇路徑）

對於複雜的工作，請在實作之前先就驗收標準達成共識。完整的[規劃／執行／驗證工作流程](13-outcome-per-token.md#plan-first-then-execute-cheaply)說明了何時應分開工作階段，並使用較便宜的執行模型。

## 2.5.10 在 Copilot 基礎架構之上疊加工具

Copilot CLI 與 VS Code Copilot 已經最佳化了代理程式迴圈的部分環節。在加入外部工具之前，請先將這些視為基準：

![MCP 與工具堆疊：限定 MCP 結構描述範圍、對長工具鏈使用 CodeAct、以 RTK 或 snip 過濾命令輸出，並以 Graphify 處理重複的定向作業。](../../source/docs/assets/diagrams/mcp-tooling-stack.svg)

- **提示詞／快取層：** 保持 `{model, active MCP set, active agent/profile}` 穩定，讓快取的前綴得以持續重複使用。
- **工具結構描述層：** 優先使用內建工具與限定範圍的 MCP；Copilot 可以延後或路由部分工具定義，但額外的伺服器與擴充功能仍會增加受攻擊面與負擔。
- **傳輸／工作階段層：** WebSocket 重複使用與自動壓縮有助於長時間的代理程式執行，但壓縮是將代理程式已看過的內容做成摘要。
- **終端機輸出層：** 內建的截斷是安全網，而非語意過濾器。

只針對第三方工具實際改善的層級加入它們：

| 層級 | 工具 | 減少的項目 |
|-------|---------|-----------------|
| 工作流程回合 | CodeAct | Copilot CLI 中因許多小型工具呼叫而產生的重複回放 |
| 命令輸出 | RTK 或 snip | 冗長的 `git`、測試、grep、建構與基礎結構命令輸出 |
| 命令選擇 | minimal-context-tools | 透過引導使用 `rg`、`fd`、`jq`、`ast-grep`，減少大範圍的檔案讀取與反覆搜尋 |
| 程式碼庫定向 | Graphify | 跨工作階段重複的結構性檔案讀取 |
| 可見度／稽核 | [Tokentop](https://github.com/tokentopapp/tokentop)、Tokalator、token-optimizer | Tokentop 提供即時的本機工作階段、模型、token、成本與消耗速率可見度；監控本身並不會壓縮 |

避免重複的過濾器。CodeAct 與單一輸出過濾器處理的是不同的成本；在同一個命令路徑上同時使用 RTK 與 snip，可能造成結果被重複截斷並隱藏失敗。

請使用[實務設定](10-practical-setup.md#step-3-choose-mode-model-and-context)來套用這些選擇。工具的安裝說明位於 [MCP 與工具成本](08-mcp-tool-costs.md)，而不在這份工作流程摘要中。

---

**下一篇：** [AGENTS.md 的問題 →](07-agents-md-problem.md)
