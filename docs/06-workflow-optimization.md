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

### 互補方案：使用 RTK 進行工具輸出壓縮

CodeAct 減少了工具調用的「次數」。[**RTK (Rust Token Killer)**](https://github.com/rtk-ai/rtk) 則減少了每次工具調用結果的「大小」。它們解決了同一個問題的不同面向，並且可以結合使用。

RTK 是一個 CLI 代理，它會攔截 `git`、`cargo test`、`grep`、`ls` 以及其他 100 多種開發指令，並在輸出到達代理之前進行壓縮——每次指令可節省 60–90% 的 Token。與 CodeAct 不同，RTK 不限於 Copilot CLI；只要 shell hook 可靠，它可以在各種 Copilot 介面中提供幫助。請將 Windows 設定視為試點，而非預設部署。有關設定和完整指令列表，請參閱 [MCP 與工具成本 §2.7.7](08-mcp-tool-costs.md#277-compress-tool-output-at-the-source-rtk)。

## 2.5.4 預設使用自動模型選擇 (Auto Model Selection)

模型選擇器是 Copilot 中成本最高的控制介面之一。為了「以防萬一」而鎖定高算力模型，會將該模型的每 Token 費率應用於會話中的每一次互動——包括那些瑣碎的互動。

**正確的預設值是 Auto。** 根據 GitHub 的官方文件，Auto 會根據即時系統健康狀況和模型效能，從支援的自動選擇池中進行選擇，並且在付費方案中，其計費費率比手動鎖定相同模型更優惠。請將其視為最佳的預設基準，而不是自動升級到每個高階模型。高成本模型仍需謹慎鎖定。僅在您更了解情況時才進行覆蓋：

- 當您「知道」任務很瑣碎（自動完成風格、語法查詢、單行編輯）時，鎖定到便宜/快速的模型。
- 當您「知道」任務需要深度推理（架構、安全性審查、新穎的分解）時，鎖定到高算力模型。
- 否則，**讓 Auto 選擇。** 它會自動捕捉便宜/預設的路徑，而無需您對選擇器進行微觀管理。如果您想要更高成本的高階模型，請明確鎖定它。請參閱 [模型選擇與定價](11-models-and-pricing.md)。

將預設值從「總是 Sonnet」或「總是 Opus」切換為「Auto，必要時覆蓋」的團隊通常會減少支出，因為他們不再預設將每次互動都放入高成本通道。

### 快取感知模型工作流

模型路由和快取必須協同工作。在長期昂貴的會話中，避免在執行緒中間更改成本/控制介面：

- 除非任務明顯改變，否則不要切換模型
- 除非任務確實需要不同的工具，否則不要切換 MCP 伺服器
- 不要在同一個長執行緒中切換代理/設定檔模式

原因：這些控制項位於上下文的高穩定前綴中。更改它們可能會使快取的前綴失效，並強制重新處理大量的輸入區塊。

實踐模式：

1. 在會話開始時選擇通道：`{模型, 代理/設定檔, MCP 集合}`。
2. 在處理該執行緒時保持通道穩定。
3. 如果必須更改通道，請以簡潔的交接摘要開始一個新的聊天。

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

## 2.5.7 使用 `/chronicle` 閉環回饋

Token 浪費不僅僅存在於單個提示詞中，還存在於你沒注意到的**模式**中。同樣的意圖誤讀在每個對話中都會額外消耗 5K Token。Copilot 為此提供了一個內建的問饋機制：[`/chronicle`](https://docs.github.com/en/copilot/concepts/agents/copilot-cli/chronicle) 斜槓指令會分析你的本地對話歷史，並告訴你 Copilot 在哪裡感到困惑、你在哪裡繞圈子，以及如何修復它。

> **範圍：** `/chronicle` 是 **Copilot CLI** 的功能，由 `~/.copilot/session-state/` 中的本地對話歷史記錄支援。它在 Copilot CLI 互動式對話中運行，也可以在 **JetBrains IDE** 中透過互動式 Copilot CLI 對話運行。它**不適用於** **VS Code**；有關 VS Code 的使用分析，請參閱下方的 [AI Engineering Coach](#258-vs-code-usage-analytics-ai-engineering-coach)。
>
> **可用性：** `/chronicle` 目前處於實驗階段。在互動式 Copilot CLI 對話中使用 `/experimental on` 啟用它，或在命令列中傳遞 `--experimental`。

完整的子指令集包括 `standup`、`tips`、`cost tips`、`search`、`improve` 和 `reindex`。其中對節省 Token 影響最大的三個是：

| 指令 | 功能 | 節省 Token 的效益 |
|---------|--------------|---------------------|
| **`/chronicle cost tips`** | 分析你最近對話中的 Token 消耗（提示詞長度、工具調用頻率、接續步驟），並提出具體的降本建議 | 最高，且與本指南最相關。直接針對 Token 消耗。 |
| **`/chronicle improve`** | 掃描對話歷史中的來回溝通、誤解意圖和重複修正，然後**生成自定義指令片段**以防止下次出現同樣模式 | 高。從源頭切斷重複的浪費。每次修復都會在該儲存庫未來的每個對話中產生複利效應。 |
| **`/chronicle tips`** | 根據你實際使用 Copilot 的方式提供個性化指導，發掘你遺漏的功能和工作流改進 | 中。通常會建議 Ask 模式、模型路由或上下文範圍更改，這些都值得投入 Token。 |
| **`/chronicle standup`** | 從你的對話數據（分支、PR、狀態）生成站立會議摘要 | 間接 —— 節省你花在回顧昨天工作上的 10 分鐘，而非直接節省 Token。 |

### `cost tips` 工作流

與本指南最直接相關。每週執行一次，看看你的 Token 到底花在哪裡。

```text
/chronicle cost tips
```

Copilot CLI 會分析你最近對話中的 Token 使用情況（查看提示詞長度、工具調用頻率和接續步驟等模式），並提出具體的、基於實際使用的降本方法。與通用建議不同，這些建議與你的真實對話數據掛鉤，因此它們往往能指出消耗你最多成本的少數習慣。

### `improve` 工作流

這是生成持久修復的方法。每週執行一次，或者當你發現自己在想「為什麼它老是搞錯？」的時候執行。

```text
/chronicle improve
```

Copilot CLI 會讀取你最近的 CLI 對話，識別重複出現的困惑（例如：它一直假設錯誤的測試框架，或者一直詢問哪個目錄包含 API 代碼），並建議在你的自定義指令中添加內容。審查這些建議 —— 接受那些符合實際專案特定注意事項的建議，跳過那些只是 LLM 生成的樣板內容（參見 [第 7 部分：AGENTS.md 問題](07-agents-md-problem.md) 了解為什麼修剪很重要）。

**為什麼這符合 Token 優化指南：** 每次來回溝通都是完整的輸入 Token（你的後續跟進 + 累積的歷史記錄）**加上**完整的輸出 Token（修正後的內容）。一個導致額外三次來回的意圖誤讀很容易就消耗 10K-30K Token。透過 `/chronicle improve` 捕捉到一個模式 → 在 `copilot-instructions.md` 中添加一兩行內容 → 該模式就不再產生重複的 Token 成本。

### `tips` 工作流

每隔一兩週執行一次：

```text
/chronicle tips
```

將這些建議視為代碼審查 —— 並非所有建議都值得採納，但那些符合你實際工作流的建議通常具有很高的投資報酬率 (ROI)。與本指南重疊的常見建議包括：將解釋請求切換到 Ask 模式、使用 `applyTo` 限定指令文件的範圍、禁用未使用的 MCP 伺服器。

### 在工作流中的位置

- **每週：** `/chronicle cost tips` —— 查看 Token 去向，以及 `/chronicle tips` —— 捕捉遺漏的習慣。
- **當感覺事情重複時：** `/chronicle improve` —— 將摩擦轉化為一次性修復。
- **每日站會（可選）：** `/chronicle standup last 24 hours` —— 為了人類的儀式感，而非為了 Token。

所有對話數據都保存在本地的 `~/.copilot/session-state/` 中，且僅存在於你的機器上。這是 **Copilot CLI 對話數據**，而非通用的 Copilot Chat 歷史存儲。當你執行 `/chronicle` 指令時，標準的模型互動仍然適用（數據會發送到模型以生成摘要），但不會上傳任何內容進行存儲。

## 2.5.8 VS Code 使用分析：AI Engineering Coach

[`/chronicle`](#257-close-the-loop-with-chronicle) 涵蓋了 Copilot CLI 對話。對於 VS Code 的使用，[**AI Engineering Coach**](https://github.com/microsoft/AI-Engineering-Coach) 是對應的工具 —— 這是一個本地 VS Code 擴充功能，它會讀取你的 VS Code AI 對話日誌，並呈現同類型的洞察：反模式、Token 模式、上下文健康狀況和技能發現。

> **隱私：** 所有分析都在本地運行。數據不會離開你的機器。該擴充功能是唯讀的 —— 它絕不會修改你的對話文件。可選的 AI 功能（規則編譯器、上下文審查）僅在你明確調用時才使用 VS Code 內建的 Copilot 模型 API。

與 Token 效率相關的關鍵功能：

| 功能 | 呈現內容 |
|---------|-----------------|
| **反模式 (Anti-Patterns)** | 涵蓋提示詞質量、對話衛生、代碼審查、工具掌握和上下文管理的 45 條可編輯規則 —— 附帶嚴重程度評級和具體的修復行動 |
| **上下文健康狀況 (Context Health)** | 代理就緒檢查清單、工作區上下文地圖和指令文件審計 |
| **技能查找器 (Skill Finder)** | 檢測歷史記錄中重複的提示詞模式，並將其與開源目錄中的可重用技能進行匹配 |
| **輸出 / 燃盡圖 (Output / Burndown)** | 按語言和模型分類的 AI 生成代碼量；Token 預算進度及預測 |

**快速開始：**

```bash
git clone https://github.com/microsoft/ai-engineering-coach.git
cd ai-engineering-coach
npm install && npm run package
code --install-extension ai-engineer-coach-*.vsix
```

然後 `Cmd+Shift+P` → **AI Engineer Coach: Open Dashboard**。

**它如何與 `/chronicle` 互補：** `/chronicle` 作用於 CLI 對話歷史以生成指令修復。AI Engineering Coach 作用於 VS Code 對話歷史，為你的實踐評分並標記結構性問題（上下文膨脹、未使用的 MCP、指令文件缺失）。兩者結合使用：用 chronicle 修復重複出現的提示詞失敗；用 AI Engineering Coach 審計更廣泛的 VS Code 設置並追蹤趨勢。

## 2.5.9 先規劃，後執行（並為階段路由模型）

最昂貴的 Token 是花在達成*錯誤*結果上的：一個代理朝著錯誤方向編寫了二十個步驟，然後被撤銷並重做。將**規劃**與**執行**分離是減少這種浪費的最高槓桿習慣之一。

**兩階段模式：**

1. **先在規劃模式（或 Ask 模式）中進行規劃。** 在編寫任何代碼*之前*，使用 Copilot CLI 的規劃模式（或 VS Code 的 Ask 模式）思考方法 —— 要改動的文件、更改順序、邊緣情況、驗收標準。規劃是廉價的：它主要是推理，沒有大型 Diff，沒有重複的工具循環。這正是強大模型發揮價值的地方，因為一個好的計劃可以防止下游昂貴的重做。
2. **保存計劃，然後執行。** 將商定的計劃寫入文件（例如 `plan.md`）或追蹤的 Issue 中，然後啟動一個**全新的對話**，並根據該保存的計劃提示執行。乾淨的對話可以保持可快取的字首穩定（參見 [快取 §2.3.5](04-context-management.md#235-caching-store-and-reuse-context-within-prompts)），並避免在每次執行輪次中將整個規劃對話作為輸入 Token 拖入。

**為什麼這能節省 Token：**

- **減少浪費的步驟。** 一個具體的、預先商定的計劃意味著代理不會去探索、猜測需求或回溯。每個避免的代理步驟都節省了一次完整的上下文重新加載（參見 [最小化代理步驟 §4.5.3](10-practical-setup.md#453-minimizing-agent-steps)）。
- **更便宜的執行路徑。** 一旦完成深度思考並將其捕獲為明確的步驟，執行通常是機械性的 —— 較便宜的模型（Auto 或包含的模型）即可完成。將高級模型保留給推理質量決定結果的規劃階段。參見 [模型路由 §4.5](10-practical-setup.md#step-5-mix-models-by-task-model-routing)。
- **乾淨的執行上下文。** 從保存的計劃開始執行，而不是從一個漫長的「先規劃後構建」的超長對話開始，可以保持歷史記錄簡短且對快取友好 —— 每輪的輸入成本保持在低位。

**經驗法則：** 用強大的模型進行規劃，用便宜的模型進行執行，並在兩者之間將計劃保存在磁碟上。這樣可以用更少的總 Token 達成結果，*而且*通常質量更高，因為計劃在編寫任何代碼之前就已經過審查。

---

**下一頁：** [AGENTS.md 問題 →](07-agents-md-problem.md)
