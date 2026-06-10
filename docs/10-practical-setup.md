# 第 4 部分：實踐設置

[← 返回指南](index.md)

---

## 4.1 為 Token 效率配置 GitHub Copilot

### 第 1 步：建立 `copilot-instructions.md`

在存儲庫根目錄下建立 `.github/copilot-instructions.md`。此文件在專案中的每次 Copilot 互動時都會被加載。

```bash
mkdir -p .github
touch .github/copilot-instructions.md
```

**入門模板 (Token 優化版)：**

```markdown
Terse like caveman. Technical substance exact. Only fluff die.
Drop: articles, filler (just/really/basically), pleasantries, hedging.
Fragments OK. Short synonyms. Code unchanged.
Pattern: [thing] [action] [reason]. [next step].
ACTIVE EVERY RESPONSE. No revert after many turns. No filler drift.
Code/commits/PRs: normal. Off: "stop caveman" / "normal mode".
```

這大約是 50 個 Token。自然語言的對等版本將超過 120 個 Token。你每次互動都能節省 70 多個 Token。

### 第 2 步：添加專案特定指令 (壓縮後)

以相同的壓縮風格添加你的專案上下文：

```markdown
Stack: Node.js 20, TypeScript 5.4, PostgreSQL 16, Redis.
Test: Vitest. Lint: ESLint flat config.
Style: functional core, imperative shell. No classes.
Naming: camelCase vars/fns, PascalCase types, UPPER_SNAKE constants.
Errors: Result<T,E> pattern, no thrown exceptions in business logic.
```

對比自然語言版本：

```markdown
This project uses Node.js version 20 with TypeScript 5.4. We use PostgreSQL 16
as our primary database and Redis for caching. For testing, we use Vitest, and
for linting, we use ESLint with the new flat configuration format.

We follow a functional core, imperative shell architecture. Please don't use
classes. For variable and function naming, use camelCase. Types should be in
PascalCase, and constants should be in UPPER_SNAKE_CASE.

For error handling, we use the Result<T,E> pattern. Don't throw exceptions in
business logic code.
```

兩者傳達了完全相同的資訊。壓縮版本約 40 個 Token。冗長版本約 110 個 Token。**節省了 64% 的成本，套用於每次互動。**

### 第 3 步：選擇你的預設模式

在 VS Code 中，Copilot Chat 提供模式選擇。預設策略如下：

| 任務類型 | 模式 | 為什麼 |
|-----------|------|-----|
| 快速提問 | Ask | 單次 LLM 調用，無工具開銷 |
| 代碼解釋 | Ask | 無需修改文件 |
| 錯誤診斷 | Ask (通常) | 由你提供上下文 |
| 單個文件更改 | Edit | 有針對性，開銷極小 |
| 跨文件重構 | Agent | 需要跨文件讀/寫代碼 |
| 新功能實現 | Agent | 多步驟建立 |
| 問題到 PR 的自動化 | Coding Agent | 全自動工作流 |

### 第 4 步：戰略性選擇模型

GitHub Copilot 的定價取決於模型選擇和計費模式。選擇其成本與任務實際需要的*努力程度*相匹配的模型。有關定價細節和計費時間表，請參閱 [模型選擇與定價](11-models-and-pricing.md)。

| 模型層級 | 相對 Token 成本 | 適用於 |
|------------|:------------------:|---------|
| 輕量級 (GPT-4.1 mini, Haiku) | 最低 | 自動補全、簡單語法、查詢類問題 |
| 標準 (GPT-4.1, Sonnet) | 中 | 大多數編碼任務 —— 實現、重構、修復 |
| 高努力度 (Claude Opus, o-系列推理) | 最高 | 架構設計、深度推理、新型問題分解 |
| **自動 (Auto)** | 預設較低 | 預設情況下：Copilot 會從支持的自動池中選擇，並在符合條件的情況下套用付費計劃折扣 |

**預設使用 Auto。** Auto 是最佳的通用基準，因為它減少了選擇器的疲勞，並且在符合條件的付費計劃使用中，會套用 GitHub 文檔中說明的折扣。將 Auto 視為預設通道，而非自動升級到每個高努力度模型。如果你需要頂級的高努力度模型，請手動固定。參見 [模型選擇與定價](11-models-and-pricing.md)。

**永遠不要把高努力度模型浪費在「X 的語法是什麼」這類問題上** —— 你會為一個最便宜的模型就能正確回答的問題支付更高的 Token 費率。

### 第 5 步：按任務混合模型 (模型路由)

一個有用的成本槓桿是：**在同一個工作流中針對不同的子任務使用不同的模型**。詳細的定價上下文、歷史倍率參考、計劃可用性和官方 GitHub 文檔連結現在都在 [模型選擇與定價](11-models-and-pricing.md) 中。本節將重點放在實踐中的路由習慣。

#### 模型混合策略

將模型與任務的認知需求相匹配：

| 任務類型 | 推薦模型 | 相對成本 | 為什麼 |
|-----------|:-----------------:|:-------------:|-----|
| "這個函數是做什麼的？" | GPT-4.1 / GPT-5 mini | **已包含** | 知識檢索，無需推理 |
| "X 的語法是什麼？" | GPT-4.1 / GPT-5 mini | **已包含** | 已記憶的知識 |
| 快速解釋、總結 | Claude Haiku 4.5 | **0.33x** | 快速、便宜、夠好 |
| 代碼審查、語法建議 | Claude Haiku 4.5 | **0.33x** | 模式匹配，而非深度推理 |
| 實現功能、修復錯誤 | Claude Sonnet 4.5 | **1x** | SWE-bench 表明這是實際的預設選擇 |
| 跨文件重構 | Claude Sonnet 4.5 | **1x** | 在真實編碼任務中與 Opus 持平 |
| 架構決策、系統設計 | Claude Opus 4.6 | **3x** | 深度推理證明了其成本的合理性 |
| 根據規範進行複雜的多步計劃 | Claude Opus 4.6 | **3x** | 新型問題分解 |
| 安全審計、威脅建模 | Claude Opus 4.6 | **3x** | 細微差別和徹底性很重要 |

#### 真實世界的節省範例

每日典型工作流 (30 次互動)，以標準層級等效的 Token 成本表示：

| 不混合模型 (全用 Sonnet) | 混合模型 | 節省比例 |
|:---------------------------:|:-----------:|:-------:|
| 30 × 1x = **30 個成本單位** | 10 × 已包含 + 8 × 0.33x + 10 × 1x + 2 × 3x = **18.6 個成本單位** | **38%** |

如果你所有的任務都使用 Opus：30 × 3x = 90 個成本單位。混合使用則降至 18.6 —— 相對模型成本**降低了 79%**。

#### 自動模型選擇應成為你的預設設置

Copilot 的 **Auto** 模式會根據即時系統健康狀況和模型性能，從支持的自動選擇池中進行選擇。在付費計劃中，GitHub 的文檔說明在 Copilot Chat 中使用符合條件的 Auto 會獲得 **10% 的折扣**。將 Auto 視為預設的低摩擦通道；高成本的頂級模型仍需手動固定。

**預設使用 Auto，僅在需要時覆蓋。** 對於團隊來說，這是一個高效的預設設置，因為它能將日常預設保持在較低成本的通道。除非你有特定原因要固定模型，否則請使用 Auto —— 例如，你*明確知道*任務很簡單 (強制使用最便宜的層級) 或者你*明確知道*它需要深度推理 (手動固定到頂級模型)。有關具體的權衡，請參閱 [模型選擇與定價](11-models-and-pricing.md)。

#### 反模式：所有任務都使用高努力度模型

一個代價高昂的習慣：對每次互動都預設使用 Opus 或其他高努力度模型。人們這樣做是因為「更好的模型 = 更好的結果」。對於大多數日常編碼任務，額外的模型成本很難證明其合理性。

將高努力度模型留給它們真正的強項：新型推理、架構判斷，以及 1-2% 的品質差異足以支撐 3-5 倍成本增加的任務。

#### 推理努力 (Reasoning Effort)：另一個撥盤

除了模型選擇之外，在具備推理能力的模型上還存在第二個成本撥盤：**思考努力 (thinking effort)** (或稱 **推理努力 (reasoning effort)**)。這控制了模型在回應之前花費多少 Token 進行思考 —— 同時影響文本、工具調用和擴展思考。

| 努力等級 | 行為 | Anthropic 推薦的用途 |
|:------------:|----------|----------------------------|
| `max` | 不限制 Token 支出 | 最深度的推理，徹底的分析 |
| `high` (預設) | 始終進行深度思考 | 複雜推理、困難編碼、代理任務 |
| `medium` | 適度的 Token 節省，可能跳過思考 | **Anthropic 推薦的 Sonnet 4.6 預設值** —— 代理編碼、工具密集型工作流、代碼生成 |
| `low` | 顯著的 Token 節省，對簡單任務跳過思考 | 高通量、延遲敏感、聊天、簡單分類 |

來源：[Anthropic Effort Parameter Docs](https://platform.claude.com/docs/en/build-with-claude/effort), 2026 年 4 月; [VS Code Language Models Docs](https://code.visualstudio.com/docs/copilot/concepts/language-models), 2026 年 4 月; [GitHub Copilot CLI programmatic reference](https://docs.github.com/en/copilot/reference/copilot-cli-reference/cli-programmatic-reference), 2026 年 4 月。

來自 Anthropic 文檔的關鍵事實：

- **努力程度會影響所有內容**，而不僅僅是思考 Token。較低的努力 = 更短的文本回應、更少的工具調用、更少的操作前言。這是一個比 `budget_tokens` 更廣泛的槓桿。
- **Anthropic 推薦將 `medium` 作為 Sonnet 4.6 的預設值**，而非 `high`。他們的文檔明確指出，`medium` 是包括代理編碼在內「大多數應用程式的速度、成本和性能的最佳平衡點」。
- **已在多個具備推理能力模型系列的 Copilot 中公開。** 在 VS Code 中，推理模型（如 Claude Sonnet/Opus 推理變體以及受支持的 GPT 推理模型）會顯示思考努力子選單供選擇。GPT-4.1 和 GPT-4o 等非推理模型則不會顯示該控制。在 Copilot CLI 中，某些模型也支持在配置中設置 `reasoning_effort`。
- **無需啟用擴展思考即可工作。** 你不需要開啟單獨的可見思考模式即可獲益 —— 努力程度會控制總體的 Token 支出，無論是否可見。
- **尚無發布的基準測試。** Anthropic 提供定性指導（如上表），但尚未發布關於「品質 vs 努力」權衡的具體數據。這是廠商推薦的，尚未經過獨立基準測試。

**Copilot 確實在許多具備推理能力的模型上公開了此功能。** 在 VS Code 中，於模型選擇器中選擇一個推理模型，開啟其思考努力子選單並選擇等級。這適用於推理導向的模型系列，包括 Claude Sonnet/Opus 推理模型和受支持的 GPT 推理模型；GPT-4.1 和 GPT-4o 等非推理模型不會顯示該子選單。在 Copilot CLI 中，某些模型也允許在配置中設置 `reasoning_effort`，GitHub 文檔中以 `gpt-5.3-codex` 為例。同樣的槓桿在 Claude API 和相關工具中也可以直接使用。對於對等的編碼任務，將 Sonnet 設置在 `medium` 努力與將 Opus 設置在 `high` 努力相比，其總成本差異仍可能達到 3-5 倍以上。

### 第 6 步：針對你實際使用的模型調優指令

當你更換模型時，不要假設舊的提示詞堆疊仍是最佳的。提供商的提示指南是針對特定版本的，並且經常解釋行為變化：冗長度、工具積極性、結構偏好、推理努力和停止條件。

快速工作流：

```text
將官方指南 URL 粘貼到 Copilot 中。
指名目標模型和文件。
要求 Copilot 在保持行為不變的情況下調整提示詞/指令。
審閱 diff；僅保留能減少錯誤跳轉的具體更改。
```

範例：

```text
Target model: Claude Sonnet 4.6.
Guide: https://platform.claude.com/docs/en/docs/build-with-claude/prompt-engineering/claude-4-best-practices
Files: .github/copilot-instructions.md, .github/instructions/*.instructions.md
Adapt instructions for this model. Preserve repo behavior. Reduce rework. Keep terse.
```

參見 [工作流優化 §2.5.5](06-workflow-optimization.md#255-針對目標模型重新調整提示詞) 以獲取提供商指南 URL 以及 Sonnet、GPT-5.5 和 Gemini 的並排比較範例。

### 第 7 步：在提示詞之外添加組織護欄

不要試圖僅靠提示詞文本來解決治理問題。提示詞文件用於塑造行為。計費控制則位於別處。

如果你正在引導組織或企業範圍的推廣，請在此處停止並閱讀 [企業治理](12-enterprise-governance.md)。該章節包含管理員指導：AI 額度預算、每使用者收緊、模型訪問政策、組織指令和分開組織的權衡。

本頁面用於從業者設置。使用企業章節進行客戶治理決策。

### 第 8 步：在 AI 工作前轉換非文本輸入

當工作流從 `.docx`、`.pdf`、`.pptx`、`.xlsx`、HTML 導出、圖像、音頻、視頻或 ZIP 存檔開始時，請在內容到達 Copilot 或 RAG 管道之前添加轉換步驟。豐富格式攜帶的佈局和元數據會膨脹輸入 Token，但不會提升模型的理解能力。

[Marc Bara 的「格式稅」文章](https://medium.com/@marc.bara.iniesta/your-docx-is-wasting-33-of-your-ai-budget-86a3d229d042) 給出了操作原則：Markdown 應作為 AI 的工作格式，而 Word/PDF 僅在人類流程需要時作為輸出格式。文章引用了一個 10 頁 PDF 的範例，在進行乾淨的 Markdown 轉換後，Token 從大約 12,400 個降至 8,350 個 —— 對於相同的內容，輸入減少了約 33%。

[Microsoft MarkItDown](https://github.com/microsoft/markitdown) 是首選的工具。它可以將 PDF、Word、PowerPoint、Excel、圖像、音頻、HTML、CSV/JSON/XML、ZIP 內容、YouTube URL、EPUB 等轉換為 Markdown，供 LLM 和文本分析工作流使用。

```bash
pip install 'markitdown[all]'

markitdown report.docx -o report.md
markitdown slides.pptx -o slides.md
markitdown spreadsheet.xlsx -o spreadsheet.md
markitdown source.pdf > source.md
```

對於生產環境的管道，請儘可能僅安裝必要的 extras：

```bash
pip install 'markitdown[pdf,docx,pptx,xlsx]'
```

然後將 `.md` 文件發送給模型，對 `.md` 文件進行分塊/索引以便檢索，並僅在最終交付步驟重新生成 `.docx` 或 `.pdf`。對於不可信的載入，請先驗證路徑和 URL；MarkItDown 以執行進程的權限進行 I/O 操作。

## 4.2 將可重用的引導保留在始終開啟的上下文之外

本存儲庫不再附帶可安裝的工作流包。請保持相同的習慣：將偶爾使用的工作流引導放在始終開啟的提示詞之外，僅在任務需要時才提取。

好的候選者包括：

- PR 審查檢查清單
- 發布或回滾模板
- 調試手冊
- 單個子系統的遷移筆記

將它們存儲在你的團隊存放可重用提示詞或操作筆記的任何地方。Token 規則保持不變：如果一條規則在大多數互動中都不需要，就不要在每次互動中都支付費用。

### 對於 CLI 密集型使用者可選：CodeAct

如果你的大部分長時間工作都在 **Copilot CLI** 中進行，那麼可選的外部插件 [`copilot-codeact-plugin`](https://github.com/jsturtevant/copilot-codeact-plugin) 值得評估。它不是本存儲庫的一部分，也不是通用的 Copilot Chat 功能。其價值主張在於工作流形狀：將許多 `grep` / `view` / `bash` / MCP 跳轉合併為一次沙盒執行，從而減少完整上下文和工具目錄重複播放的次數。將其用於 CLI 密集型會話；如果你的工作主要是 IDE 聊天/編輯，或者你不希望在路徑中使用外部插件，則可以跳過它。

### MCP vs. 技能：渴望式載入 vs. 延遲式上下文載入

MCP (模型上下文協議伺服器) 在每次互動中都會將其**完整的工具架構**注入到上下文中 —— 無論這些工具是否被使用。一個擁有 20 個工具的伺服器可能會在你的會話中為每次請求增加數千個 Token。

技能 (Skills) 的行為則不同：預先僅載入**標題和描述**。完整的技能內容僅在該技能與當前任務實際相關時，才會按需提取。

| 機制 | 每次對話載入內容 | 完整內容何時載入 |
|-----------|--------------------|--------------------------|
| **MCP** | 完整的工具架構 (始終) | N/A —— 始終存在 |
| **技能 (Skill)** | 僅標題 + 描述 | 按需載入，當被調用時 |

**規則：** 對於大多數互動中都需要的功用，使用 MCP。對於偶爾使用的功用，使用技能 —— 對於 MCP，你每輪對話都要支付完整架構的費用，而對於技能，你只需為每次調用支付費用。如果一個工具在 10 次對話中僅被使用 1 次，那麼技能的上下文開銷大約便宜 10 倍。

## 4.3 GitHub Coding Agent 考量

Coding Agent 執行自動化會話，可能會持續幾分鐘到幾小時。Token 的節省會在這些長會話中複合。

### 4.3.1 壓縮 `copilot-instructions.md`

代理會讀取此文件。壓縮後的指令文件能在每個內部規劃步驟中節省 Token —— 而代理在每個會話中會執行許多步驟。

### 4.3.2 使用 `copilot-setup-steps.yml`

預先以確定性的方式安裝依賴項：

```yaml
# .github/copilot-setup-steps.yml
steps:
  - name: Install dependencies
    run: npm ci
  - name: Build
    run: npm run build
```

如果沒有這個，代理會通過試錯來發現並安裝依賴項 —— 每次嘗試都會消耗 LLM 調用和 Token。**節省：總會話 Token 的 10-30%。**

### 4.3.3 編寫精確的問題描述

模糊的問題會導致代理進行廣泛的代碼庫探索（閱讀許多文件 = 許多 Token），並可能誤解需求（返工 = 更多 Token）。

**模糊的問題：**

```text
Fix the login bug
```

**精確的問題：**

```text
Bug: login fails when email contains '+' character.
File: src/auth/login.ts, validateEmail() on L42.
Fix: URL-encode the email before passing to the OAuth provider.
Test: add case for "user+tag@example.com" in login.test.ts.
```

**節省：通過減少探索和返工，可節省總會話 Token 的 20-50%。**

### 4.3.4 精簡 PR 評論和提交消息

代理會讀取 PR 審查評論和 Git 歷史作為上下文。當代理攝取這些內容時，冗長的提交消息或審查評論都會消耗 Token。請保持提交消息和審查評論精簡。

### 4.3.5 自定義代理配置文件

為不同的任務類型建立集中的指令，而不是單個巨大的指令文件：

```text
# For test-writing tasks
Stack: Vitest + Testing Library. AAA pattern.
Mock: external services only. No impl mocking.
Coverage: branch coverage ≥80%.
```

集中的代理比通用指令集攜帶更少的指令開銷。

### 4.3.6 使用 RTK 壓縮 Shell 命令輸出

Coding Agent 在每個會話中都會運行許多 Shell 命令 —— `git diff`、測試運行、`grep`、`ls`。每個命令的原始輸出都會作為下一步代理的輸入 Token 流回。大型失敗的測試套件或冗長的 Git diff 可能會返回數萬個 Token。

[**RTK (Rust Token Killer)**](https://github.com/rtk-ai/rtk) 是一個過濾這些輸出的 CLI 代理。它運行原始命令，移除噪音（通過的測試、未更改的 diff 行、構建產出），並返回壓縮後的結果。代理行為不變，但能看到信號集中的更小輸出。

**為 VS Code Copilot 設置 —— 按存儲庫：**

```bash
brew install rtk   # 或: curl -fsSL https://raw.githubusercontent.com/rtk-ai/rtk/refs/heads/master/install.sh | sh

cd your-repo
rtk init --copilot
# 重啟 VS Code
```

RTK 會在當前存儲庫中安裝一個 PreToolUse 鉤子。請針對每個存儲庫重複此操作 —— 沒有全局的 VS Code Copilot 安裝。一旦啟用，該鉤子是透明的：你的終端機保持不變；僅代理的 Bash 工具調用會被攔截。

具有冗長輸出的命令（測試失敗、大型 diff）獲得的縮減量最大。短輸出命令獲得的收益較小。實際節省取決於你的專案輸出量。

與 `copilot-setup-steps.yml` (§4.3.2) 和精確問題描述 (§4.3.3) 配合使用，以獲得最大的會話效率。完整設置、命令列表和對其他 AI 工具的支持見：[MCP 和工具成本 §2.7.7](08-mcp-tool-costs.md#277-在源頭壓縮工具輸出-rtk)。

## 4.4 建立習慣

### 從小處著手

1. **第 1 週：** 在你的主專案中添加壓縮過的 `copilot-instructions.md`。對簡單問題使用 Ask Mode
2. **第 2 週：** 在提示詞中練習輕量原始人式 (caveman-lite)。去掉填充詞，保持精確
3. **第 3 週：** 晉升到全面原始人式 (caveman-full)。丟掉冠詞，使用碎片化語句
4. **第 4 週：** 在代碼生成提示詞中添加 "code only"。將可重用的精簡模板保留在始終開啟的上下文之外

### 每月維護

- 審查你的 `copilot-instructions.md` —— 它是否變長了？將其壓縮回去
- 檢查是否有任何記憶文件變得冗長 —— 將它們壓縮回去
- 審核編輯器中習慣開啟的文件 —— 關閉你不在處理的文件（開啟的分頁會自動餵入上下文）
- (Business/Enterprise) 為新的敏感路徑審查存儲庫 / 組織的**內容排除**設置
- 檢查你的模型使用情況 —— 你是否在固定那些 Auto 可以路由到更便宜層級的高努力度模型？
- 在進一步擴大高級訪問權限之前，審查預算、使用者級別上限和模型政策
- 當預設模型更改時，針對該提供商當前的提示指南重新調優提示詞/指令
- 按使用者/團隊檢查 Token 使用量 —— 是否有代理和高級使用者推高了過度消耗？參見 [企業治理](12-enterprise-governance.md)

### 何時調整

| 信號 | 動作 |
|--------|--------|
| 得到錯誤結果 | 往回退一個壓縮級別 |
| 頻繁重新解釋 | 指令可能太精簡了 —— 添加一行澄清 |
| 達到速率限制 | 套用矩陣中更多的技術 |
| 新團隊成員感到困惑 | 在代碼中添加完整的英文註釋，保持指令壓縮 |
| 長時間代理會話失敗 | 檢查問題描述的精確度，添加 `copilot-setup-steps.yml` |

## 4.5 為效率配置代理模式 (Agent Mode)

### 4.5.1 代理模式 vs. 詢問模式 vs. 編輯模式

每種模式都有根本不同的 Token 成本特徵：

| 模式 | 每次操作的 LLM 調用次數 | 工具使用 | 加載的上下文 | 最適合 |
|------|:--------------------:|:--------:|:--------------:|----------|
| **Ask** | 1 | 無 | 對話 + 指令 | 問題、解釋 |
| **Edit** | 1-2 | 文件讀/寫 | 目標文件 + 指令 | 單個文件更改 |
| **Agent** | 5-25 | 完整工具集 | 所有的內容 + 工具架構 | 多步、跨文件任務 |

**成本倍數：** 對於相同的提示詞，代理模式的成本是詢問模式的 5-25 倍。代理模式下的簡單問題會觸發文件讀取、工具評估和多步推理 —— 這些對於「此函數是做什麼的？」都是不必要的。

### 4.5.2 代理模式內部迴圈

了解迴圈有助於你最小化步驟：

```text
第 1 步：加載上下文
  ├── 系統提示詞 (~500 Token)
  ├── copilot-instructions.md (~50-1500 Token)
  ├── 工具定義 (~2,000-20,000 Token)
  ├── 對話歷史 (不斷增長)
  └── 你的提示詞
  → 發送給 LLM → 獲取回應

第 2 步：LLM 決定調用工具
  ├── 工具調用 (函數 + 參數) → 輸出 Token
  ├── 工具結果 → 輸入 Token (下一步)
  └── 關於結果的推理 → 輸出 Token

第 3 步：另一次工具調用 (或生成回應)
  ├── 第 1 步的所有上下文重新載入
  ├── + 第 2 步的工具調用和結果
  └── + 不斷增長的對話
  → 再次發送給 LLM

... 重複 5-25 次
```

**關鍵洞察：** 上下文隨每一步增長。第 15 步攜帶了第 1-14 步的所有上下文加上原始提示。這就是為什麼長時間的代理會話會迅速變得昂貴。

### 4.5.3 最小化代理步驟

避開的每一步都能節省一次完整的上下文重新載入。技術如下：

**帶有驗收標準的精確提示詞：**

```text
# 差 —— 代理會進行探索、閱讀文件、猜測需求
"Fix the user registration"

# 好 —— 代理清楚地知道該做什麼
"File: src/auth/register.ts L42.
 Bug: email validation rejects valid '+' chars.
 Fix: use RFC 5322 regex.
 Test: add 'user+tag@example.com' case in register.test.ts.
 Done when: test passes, no other tests break."
```

精確版本可能在 3-5 步內完成。模糊版本：10-20 步的探索。

**為複雜任務編寫計劃文件：**

在調用代理模式之前先制定計劃：

```markdown
# plan.md
1. Add `validateEmail()` to src/utils/validation.ts
2. Import and use in src/auth/register.ts L42
3. Add test cases in tests/auth/register.test.ts
4. Run `npm test` — expect all pass
```

然後發出提示："Execute plan.md." 代理會遵循計劃，而不是自己去尋找路徑。更少的探索步驟 = 更少的 Token。

**對確定性操作使用 CLI 組合而非代理工具迴圈：**

通過代理髮出的多步驟瀏覽器或數據操作，每一步都會觸發一次 LLM 調用 —— 且每一步都會重新載入所有累積的上下文。單個生成的 CLI 命令可以在一次 Shell 調用中執行相同的工作：

```bash
# 瀏覽器自動化 —— 一次 LLM 調用生成此內容；一次 Shell 調用運行它
playwright goto https://example.com && wait 1000 && click '#submit-btn' && screenshot out.png

# 鏈接過濾 —— 無需代理迴圈
gh issue list --json number,title,labels | jq '.[] | select(.title | test("bug"; "i"))'

# 管道數據轉換
cat logs/app.log | grep ERROR | awk '{print $1, $5}' | sort | uniq -c | sort -rn | head -20
```

CLI 命令是可組合的、可檢查的、可重複運行的且可受版本控制的。更改選擇器、過濾器或 URL 意味著編輯一行文本 —— 而不是通過另一個多步迴圈重新提示代理。將代理工具使用保留給真正需要動態決策的任務；將確定性序列轉移到 Shell。

### 4.5.4 VS Code Token 效率設置

影響代理 Token 使用的相關設置：

```json
{
  // 代理可以發出的最大請求數 (預設：25)
  "chat.agent.maxRequests": 10,

  // 使用自動模型選擇 —— 為簡單子任務選擇更便宜的模型
  "github.copilot.chat.agent.model": "auto"
}
```

**`maxRequests`** 限制了代理可以發出的工具調用請求數。越低 = Token 越少，但代理可能無法完成複雜任務。從 10-15 開始，僅在需要時增加。

### 4.5.5 用於代理效率的自定義指令

添加到 `.github/copilot-instructions.md`：

```text
Minimize tool calls. Read files only when necessary.
Batch related changes. Don't read-modify-read-modify when read-modify-modify works.
Prefer grep_search over sequential read_file for discovery.
```

這些指令減少了不必要的工具調用。跳過的每個工具調用都能節省 100-2,000+ 個工具輸入/輸出的 Token。

### 4.5.6 決策框架：何時使用每種模式

```text
關於代碼/語法/概念的問題？
  → Ask Mode (1 次調用, ~500-2,000 Token)

對單個文件的更改？
  → Edit Mode (1-2 次調用, ~1,000-4,000 Token)

範圍明確的跨文件更改？
  → Agent Mode 並配合精確提示詞 (~5-10 步, ~15,000-50,000 Token)

模糊的 "fix this" 或 "improve that"？
  → 先不要使用 Agent Mode。先在 Ask Mode 中釐清範圍。
  → 然後切換到 Agent 並配合精確提示詞。
```

**一個代價高昂的模式：** 對模糊的提示詞使用代理模式，看著它探索 20 步，然後意識到它誤解了並重新開始。這可能會使 Token 使用量翻倍，卻沒有改善結果。

## 4.6 管理員護欄位於別處

本頁面用於從業者設置。如果你正在做出客戶或企業推廣決策，請改用 [企業治理](12-enterprise-governance.md)。

該章節包含：

- 使用者級別 AI 額度預算
- 重度使用監控
- 模型訪問政策
- 組織級別自定義指令
- 6 月 1 日切換指南

---

**下一頁：** [企業治理 →](12-enterprise-governance.md)
