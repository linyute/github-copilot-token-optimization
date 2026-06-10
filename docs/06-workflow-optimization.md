# 2.5 特定工作流優化

[← 返回指南](index.md)

---

## 2.5.1 提交消息 (Commit Messages)

採用 Conventional Commits 格式。標題 ≤50 個字元。僅在 "為什麼 (why)" 不明顯時才編寫本文。

**冗長提交 (~25 個 Token)：**

```text
feat: Added a new feature to allow users to reset their passwords through
the settings page, which also sends a confirmation email
```

**精簡提交 (~10 個 Token)：**

```text
feat: add password reset via settings page

Sends confirmation email on reset.
```

雖然單個提交節省的量看起來很少，但 Coding Agent 會讀取 Git 歷史作為上下文。整個存儲庫歷史中的精簡提交會產生複合效果。

## 2.5.2 PR 審查 (PR Reviews)

與其編寫長達一段的審查評論，不如使用單行格式：

**冗長審查評論 (~40 個 Token)：**

```text
I noticed that on line 42, the user variable could potentially be null at this
point in the code, which would cause a NullPointerException when you try to
access the user's email property. You should add a null check before accessing
this property to handle this edge case properly.
```

**精簡審查 (~12 個 Token)：**

```text
L42: 🔴 bug: user can be null here. Add null guard before .email access.
```

**節省：~70%。** 相同的資訊，相同的可操作性。僅消耗一小部分 Token。

嚴重性前綴用 1-2 個 Token 編碼優先級：

- 🔴 Bug / 安全問題 —— 必須修復
- 🟡 建議 —— 應修復
- 🔵 瑣碎事項 (Nit) —— 選項優化
- ❓ 問題 —— 需要澄清

## 2.5.3 詢問模式 (Ask Mode) vs. 代理模式 (Agent Mode)

這是本指南中槓桿較高的節省機會之一。

**代理模式 (Agent Mode)** 在每個可見操作中可能會觸發 3-10 次內部模型調用。它讀取文件、制定計劃、執行、驗證。每一步都會消耗 Token。

**詢問模式 (Ask Mode)** 只有單次調用。一個問題，一個答案。

| 任務 | 正確模式 | 為什麼 |
|------|-----------|-----|
| "這個函數是做什麼的？" | Ask | 單次回答。無需使用工具 |
| "TypeScript 泛型的語法是什麼？" | Ask | 知識性問題 |
| "重構此模組以使用依賴注入" | Agent | 跨文件更改，需要讀/寫代碼 |
| "建立一個帶有測試和文件的 REST API" | Agent | 多步驟建立任務 |
| "為什麼這個測試失敗了？" | Ask (通常) | 通常只需要你提供的錯誤資訊 + 上下文 |

**節省：對於簡單問題，通過使用 Ask 而非 Agent，可節省 60-90%。**

### 進階 Copilot CLI 策略：CodeAct

對於 **工具密集型 Copilot CLI 會話**，有一個有用的例外。[`copilot-codeact-plugin`](https://github.com/jsturtevant/copilot-codeact-plugin) 是一個可選的外部插件，它改變了執行形狀：代理不再是在多輪對話中「模型 -> 工具 -> 模型 -> 工具」，而是編寫一個 Python 程序，將工作鏈接在一起並在單次沙盒執行中運行。

為什麼這可以節省 Token：

- 更少的對話輪數意味著更少地重複播放系統提示詞、先前消息和內置工具定義。
- 如果加載了 MCP 伺服器，它們的工具目錄被重複播放的次數也會減少，因此節省效果會複合。
- 一個合併後的結果通常比敘述每個中間的 `grep` / `view` / `bash` 跳轉更短。

何時使用：

- 否則需要經過多次小型工具調用的 CLI 密集型探索或審計任務
- 載入了 MCP 且架構重複播放成本已經很高的會話
- 可重複的分析任務，如 TODO 掃描、函數索引、覆蓋率檢查或交叉引用收集

何時不使用：

- 簡單的一步式 Ask 問題
- 插件不適用的正常 IDE chat/edit 工作流
- 不希望在工作流中使用外部插件的團隊

保持此主張有界：本指南**並未**對 CodeAct 本身進行基準測試。該插件的 README 報告稱在其自己的基準提示（包括加載 MCP 的情況）中 Token 使用量較低，但那是插件報告的任務數據，而非普遍的節省基準。

### 互補：用 RTK 壓縮工具輸出

CodeAct 減少了工具調用的*數量*。[**RTK (Rust Token Killer)**](https://github.com/rtk-ai/rtk) 減少了每個工具調用結果的*大小*。它們解決的是同一問題的不同方面，可以配合使用。

RTK 是一個 CLI 代理 (Proxy)，它攔截 `git`、`cargo test`、`grep`、`ls` 和 100 多個其他開發命令，並在輸出到達代理之前進行壓縮 —— 每個命令可節省 60–90%。與 CodeAct 不同，RTK 適用於所有 Copilot 介面（VS Code、CLI 和其他 AI 工具），而不僅僅是 Copilot CLI。請參閱 [MCP 和工具成本 §2.7.7](08-mcp-tool-costs.md#277-在源頭壓縮工具輸出-rtk) 了解設置方法和完整命令列表。

## 2.5.4 預設使用自動模型選擇 (Auto Model Selection)

模型選擇器是 Copilot 中成本最高的控制平面之一。「以防萬一」地固定一個高成本模型，會使該模型的每 Token 費率套用於會話中的每一次互動 —— 包括那些微不足道的互動。

**正確的預設值是 Auto。** 根據 GitHub 的官方文檔，Auto 會根據即時系統健康狀況和模型性能，從支持的自動選擇池中進行選擇。在付費計劃中，其計費費率比手動固定同一模型更有優勢。請將其視為最佳的通用基準，而非自動升級到每個頂級模型。高成本模型仍需刻意固定。僅在你更了解情況時覆蓋：

- 當你*明確知道*任務很微不足道時（自動補全風格、語法查詢、單行編輯），固定到便宜/快速的模型。
- 當你*明確知道*任務需要深度推理時（架構設計、安全審查、新型分解），固定到高成本模型。
- 否則，**讓 Auto 選擇。** 它會自動捕獲便宜/預設的路徑，而不必讓你微管理選擇器。如果你想要更高成本的頂級模型，請明確固定它。參見 [模型選擇與定價](11-models-and-pricing.md)。

將預設值從「始終 Sonnet」或「始終 Opus」切換為「Auto，需要時覆蓋」的團隊通常會減少開支，因為他們停止了將每次互動都預設進入高成本車道的行為。

## 2.5.5 針對目標模型重新調整提示詞

這不是提示詞壓縮。它可能不會減少每次請求的 Token。但它通過提高初次嘗試的品質來減少總 Token 使用量，從而減少後續輪數、重複澄清和代理的返工。

模型提供商發布的提示指南會隨模型版本更新。像對待依賴項升級一樣對待模型升級：閱讀遷移/提示指南，然後針對該模型的當前行為調整你的提示詞和指令文件。

**工作流：**

```text
開啟目標模型的官方提示指南。
將 URL 粘貼到 Copilot chat 中。
詢問："根據此指南調整這些目標文件。保持行為不變。減少返工。"
目標文件：.github/copilot-instructions.md, .github/instructions/*.instructions.md, agents/*.md, app 提示文件。
審閱 diff。僅保留可衡量的、與模型相關的更改。
```

官方起點：

| 提供商 | 模型系列 | 提示指南 |
|---|---|---|
| Anthropic | Claude Sonnet / Opus / Haiku | [Prompt engineering overview](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/overview) 和 [Claude latest-model best practices](https://platform.claude.com/docs/en/docs/build-with-claude/prompt-engineering/claude-4-best-practices) |
| OpenAI | GPT-5.5 / GPT-5 | [GPT-5.5 prompting guide](https://developers.openai.com/api/docs/guides/prompt-guidance) 和 [GPT-5 prompting guide](https://cookbook.openai.com/examples/gpt-5/gpt-5_prompting_guide) |
| Google | Gemini | [Gemini prompt design strategies](https://ai.google.dev/gemini-api/docs/prompting-strategies) |

### 範例：一個基礎指令，三種調優方式

基礎指令：

```markdown
You are a coding assistant. Help with implementation. Be concise. Ask questions if needed. Follow repo style. Run tests.
```

針對特定模型的重寫：

| 目標模型 | 調優後的指令 | 為什麼合適 |
|---|---|---|
| **Claude Sonnet** | `Role: senior repo engineer.\nUse XML-ish sections when helpful: <task>, <constraints>, <done>.\nBefore edits: inspect relevant files only. Preserve existing style.\nFor ambiguous requests: ask only if choice changes implementation.\nDone = patch applied + existing targeted tests pass or blocker named.` | Claude 指南強調明確的成功標準、範例/結構、明確的工具/使用界限以及經過校準的努力。XML 風格的分隔符通常有助於區分任務、上下文和約束。 |
| **GPT-5.5** | `Outcome: correct repo change with minimal churn.\nSuccess: target behavior works, diff scoped, tests or exact blocker reported.\nChoose efficient path; do not over-spec process.\nStart tool-heavy work with one short progress update.\nAsk only for missing info that changes outcome or safety.` | GPT-5.5 指南偏好以結果為導向的提示、簡明的性格/協作規則、高效的解決路徑、多步驟工作的可見前言，以及避免遺留的過度規範。 |
| **Gemini** | `Task: implement requested repo change.\nContext: use referenced files, nearby tests, and repo conventions.\nConstraints: concise output, scoped diff, no unrelated rewrites.\nFormat final: changed files + behavior impact + test result/blocker.\nIf input is incomplete, state one needed detail.` | Gemini 指南強調明確的「任務/輸入/約束/回應格式」結構。明確的格式和上下文界限可減少解讀偏差。 |

### 給 Copilot 的使用者提示範例

```text
Target model: GPT-5.5.
Guide: https://developers.openai.com/api/docs/guides/prompt-guidance
Files: .github/copilot-instructions.md, agents/token-saver.agent.md
Adapt prompts to guide. Preserve behavior. Cut repeated clarification. Keep concise.
Show diff only.
```

在以下情況使用：

- 升級或更改預設模型。
- 提示詞在一個模型上有效，但在另一個模型上變得冗贅、懶散、過於積極或太過照本宣科。
- 在模型更改後，代理一直做出錯誤的假設。
- 維護應用程式提示詞、代理配置文件或可重用的指令文件。

當提示詞行為已經過測量且穩定時，請避免這樣做。在沒有失敗信號的情況下更改指令會增加噪音。

## 2.5.6 何時「不要」壓縮

壓縮是有極限的。某些情況需要完全的清晰度：

- **安全警告** —— "This will delete all user data" 絕對不能縮寫為 "del usr data"
- **不可逆操作** —— 確認提示必須是明確無誤的
- **入職上下文** —— 新團隊成員需要了解 "為什麼 (why)"，而不僅僅是 "做什麼 (what)"
- **複雜的多步驟指令** —— 碎片化的順序可能導致誤讀
- **監管/合規文本** —— 法律要求內容必須精確

一個設計良好的精簡提示模板或代理配置文件可以自動處理這一點 —— 在安全警告和不可逆操作確認時關閉精簡模式，然後在該部分結束後恢復。

## 2.5.7 使用 `/chronicle` 閉環

Token 浪費不僅存在於任何單個提示詞中 —— 它隱藏在那些你未察覺的**模式**中。同樣的誤讀意圖在每個會話中都會額外消耗 5K Token。Copilot 附帶了一個內置的反饋迴圈來解決這個問題：[`/chronicle`](https://docs.github.com/en/copilot/concepts/agents/copilot-cli/chronicle) 斜槓命令會分析你的本地會話歷史，告訴你 Copilot 在哪裡感到困惑、你在哪裡繞圈子，以及如何修復它。

> **範圍：** 完整的 `/chronicle`（所有子命令）在 **Copilot CLI** 互動式會話中可用，由 `~/.copilot/session-state/` 中的本地會話歷史支持。`tips` 子命令也在 **VS Code** 中作為 `/chronicle:tips` 可用。
>
> **可用性：** `/chronicle` 目前處於實驗階段。在互動式 Copilot CLI 會話中通過 `/experimental on` 啟用，或在命令行中傳遞 `--experimental`。

三個子命令，按 Token 節省影響力排名：

| 命令 | 它的作用 | Token 節省回報 |
|---------|--------------|---------------------|
| **`/chronicle improve`** | 掃描會話歷史以尋找來回對話、誤解意圖和重複更正 —— 然後**生成自定義指令片段**以防止下次出現同樣模式 | 最高。從源頭切斷循環性浪費。每個修復都會在該存儲庫未來的每個會話中複合。 |
| **`/chronicle tips`** (CLI) / **`/chronicle:tips`** (VS Code) | 根據你實際使用 Copilot 的方式提供個性化指導 —— 揭示你錯過的特性和工作流改進 | 中。通常會建議 Ask Mode、模型路由或上下文範圍更改，這些都值得投入 Token。 |
| **`/chronicle standup`** | 從你的會話數據（分支、PR、狀態）生成每日站報摘要 | 間接 —— 節省你重建昨天工作所需的 10 分鐘，而不是直接的 Token 支出。 |

### `improve` 工作流

這是對 Token 優化最重要的子命令。每週運行一次，或者在你發現自己思考「為什麼它一直出錯？」時運行。

```text
/chronicle improve
```

Copilot CLI 會讀取你最近的 CLI 會話，識別反覆出現的混淆（例如，它一直假設錯誤的測試框架，或者一直詢問哪個目錄包含 API 代碼），並提出添加到自定義指令的建議。審核這些建議 —— 接受那些符合真實專案特定警示的建議，跳過那些只是 LLM 生成的樣板代碼（參見 [第 7 部分：AGENTS.md 問題](07-agents-md-problem.md) 了解為什麼修剪很重要）。

**為什麼這符合 Token 優化指南：** 每一輪來回對話都是全額的輸入 Token（你的後續操作 + 累積的歷史記錄）**加上**全額的輸出 Token（修正後的回應）。一個耗費額外三輪對話的單次誤讀意圖很容易消耗 10K-30K Token。通過 `/chronicle improve` 捕捉一個模式 → 在 `copilot-instructions.md` 中添加一兩行 → 該模式停止產生重複的 Token 成本。

### `tips` 工作流

每隔一兩週運行一次。命令因介面而異：

```text
# Copilot CLI
/chronicle tips

# VS Code
/chronicle:tips
```

像對待代碼審查一樣對待這些建議 —— 並非所有建議都值得採納，但那些符合你實際工作流的建議通常具有高投資報酬率 (ROI)。與本指南重疊的常見建議包括：切換到 Ask Mode 進行解釋請求、使用 `applyTo` 劃分指令文件範圍、禁用未使用的 MCP 伺服器。

### 在工作流中的位置

- **每週：** `/chronicle tips` (CLI) 或 `/chronicle:tips` (VS Code) —— 捕捉錯過的習慣。
- **當感覺重複時：** `/chronicle improve` —— 將摩擦轉化為一次性修復。
- **每日站報 (可選)：** `/chronicle standup last 24 hours` —— 為了人類的儀式感，而非 Token。

所有會話數據都本地存儲在 `~/.copilot/session-state/` 中，且僅存在於你的機器上。這是 **Copilot CLI 會話數據**，而非通用的 Copilot Chat 歷史存儲。當你運行 `/chronicle` 命令時，仍會發生標準的模型交互（數據被發送給模型以生成摘要），但沒有任何內容被上傳存儲。

## 2.5.8 VS Code 使用分析：AI 工程教練 (AI Engineering Coach)

[`/chronicle`](#257-使用-chronicle-閉環) 涵蓋了 Copilot CLI 會話。對於 VS Code 使用情況，[**AI Engineering Coach**](https://github.com/microsoft/AI-Engineering-Coach) 是對應的工具 —— 一個本地 VS Code 擴充功能，它讀取你的 VS Code AI 會話日誌並揭示同類洞察：反模式、Token 模式、上下文健康狀況和技能發現。

> **隱私：** 所有分析都在本地運行。數據不會離開你的機器。該擴充功能是唯讀的 —— 它永遠不會修改你的會話文件。可選的 AI 特性（規則編譯器、上下文審核）僅在你明確調用時使用 VS Code 內置的 Copilot 模型 API。

與 Token 效率相關的關鍵功能：

| 功能 | 它揭示了什麼 |
|---------|-----------------|
| **反模式 (Anti-Patterns)** | 跨提示品質、會話衛生、代碼審查、工具掌握和上下文管理的 45 條可編輯規則 —— 帶有嚴重性評級和具體的修復動作 |
| **上下文健康狀況** | 代理就緒檢查清單、工作區上下文地圖和指令文件審計 |
| **技能尋找器** | 檢測你歷史記錄中重複的提示模式，並將其與開源目錄中可重用的技能進行匹配 |
| **輸出 / 消耗狀況** | 按語言和模型劃分的 AI 生成代碼量；Token 預算進度及預測 |

**快速開始：**

```bash
git clone https://github.com/microsoft/ai-engineering-coach.git
cd ai-engineering-coach
npm install && npm run package
code --install-extension ai-engineer-coach-*.vsix
```

然後按下 `Cmd+Shift+P` → **AI Engineer Coach: Open Dashboard**。

**它如何補充 `/chronicle`：** `/chronicle` 作用於 CLI 會話歷史以生成指令修復。AI Engineering Coach 作用於 VS Code 會話歷史以對你的實踐進行評分，並標記結構性問題（上下文膨脹、未使用的 MCP、指令文件缺失）。兩者配合使用：用 chronicle 修補重複發生的提示詞失敗；用 AI Engineering Coach 審核更廣泛的 VS Code 設置並追踪趨勢線。

---

**下一頁：** [AGENTS.md 問題 →](07-agents-md-problem.md)
