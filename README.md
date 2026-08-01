# GitHub Copilot Token 優化指南

> [!IMPORTANT]
> **此非 GitHub 或 Microsoft 的官方指南。** 本指南為社群資源，源自於真實領域的實務經驗 — 由採用 AI 進行開發的實務人員所觀察到的模式、測試過的技術以及汲取的教訓。它反映了業界的反饋：從零開始收集的實務知識，而非由上而下的產品文件。請使用它來了解優化策略，並根據您客戶的情境調整適用的內容。官方指南請參閱 [docs.github.com/copilot](https://docs.github.com/copilot)。

> 以資料為驅動的實用指南，旨在降低 Token 消耗量，同時保持程式碼品質。
> 涵蓋 Chat、Inline 以及 Coding Agent 工作流程。

---

## 簡報

- [精簡實務人員簡報 (18 張簡報)](slides/briefing.html)
- [完整客戶工作坊 (8 小時)](slides/index.html)

## 快速入門 — 現在就可以做的 15 件事

> **2026 年 6 月 1 日 — 用量計費 (UBB) 已正式上線。** GitHub Copilot 現在會根據從集中的 AI 點數（Business 每席位 $30、Enterprise 每席位 $70）中扣除的實際 Token（輸入 + 輸出 + 快取）進行計費，而非請求計數器。本指南中的每項技術都能直接轉化為點數節省 — 且對快取的友善習慣比以往任何時候都更加重要。有關客戶安全邊界請參閱 [企業治理](docs/12-enterprise-governance.md)，有關模型成本指南請參閱 [模型選擇與計價](docs/11-models-and-pricing.md)。

> **輸出 Token 的成本遠高於輸入 Token。** 這是本指南中最重要的高定價事實。Anthropic 的公開定價使這種不對稱性具體化（每百萬 Token 輸入/輸出：Haiku 為 $1/$5、Sonnet 為 $3/$15、Opus 為 $5/$25）。Copilot 的特定模型 UBB 定價表尚未公開，但 UBB 仍使冗長的輸出變得異常昂貴。大多數輸入 Token 來自檔案 context、歷史紀錄與工具 schema — 而非您輸入的內容。您輸入的提示詞僅佔總輸入的微小比例。請從輸出控制開始，然後解決結構性輸入的獲勝點。

![Token 成本剖析：輸入 Token 包含隱藏的 context，輸出 Token 是可見的答案，穩定的快取 Token 會更加便宜。](docs/assets/diagrams/token-cost-anatomy.svg)

沒有時間閱讀完整指南？今天就做這幾件事來削減您的 Token 使用量：

| # | 行動 | 主要效果 | 設定所需時間 |
|---|--------|----------------|----------------|
| 1 | **要求僅輸出程式碼的回應** — 在 `copilot-instructions.md` 中新增 `Code only, no explanation.`。最高單位 Token ROI：輸出成本比輸入貴 5 倍，且這能永久減少每個程式碼任務 40-70% 的輸出 | 縮減回應長度 | 0 分鐘 |
| 2 | **預設限制輸出格式** — 在 `copilot-instructions.md` 中新增 `Bullets over paragraphs. No explanations unless asked.` | 保持答案精簡 | 0 分鐘 |
| 3 | **縮減您的常駐 context** — 壓縮 `copilot-instructions.md` 並且修剪 `AGENTS.md` 至僅包含地雷區。這兩個檔案中的每個 Token 都會在每次互動（以及每個 Agent 步驟）中被計費。刪除無用填料，刪除 Agent 透過閱讀程式碼即可發現的任何內容，刪除 LLM 生成的 `/init` 樣板 | 減少常駐輸入/context | 15 分鐘 |
| 4 | **預設為 Auto 模型選擇 + 保護快取的穩定性** — 使用 Auto 作為基準，因為它會從支援的 Auto 池中進行選擇，並提供付費方案折扣。在昂貴的長對話串中，保持模型、推理強度、已載入的技能、MCP/工具集以及 agent/設定檔穩定。變更其中任何一項都可能丟棄快取前綴，因此請攜帶簡短的交接摘要開啟新的聊天。請參閱 [模型選擇與計價](docs/11-models-and-pricing.md) | 降低符合條件用量的計費費率並保留快取輸入折扣 | 0 分鐘 |
| 5 | **簡單問題使用 Ask 模式** — 將 Agent 模式保留給多步驟任務 | 避免 Agent 開銷 | 0 分鐘（只需選擇正確的模式） |
| 6 | **使用 `applyTo:` 路徑限定 context 範圍** — 將一個大型說明檔案拆分為多個僅在相關時載入的小型限定範圍檔案 | 減少常駐輸入/context | 15 分鐘 |
| 7 | **提示詞保持精準** — 使用 "Add null check to `getUser()`" 而非 "Can you please look at this and maybe add some error handling?" 注意：您輸入的提示詞僅佔總輸入的一小部分；精準度對品質的影響大於對原始 Token 的節省 | 提升任務標的精準度 | 0 分鐘 |
| 8 | **針對目標模型重新調整提示詞** — 提供者提示指南因模型/版本而異。將官方指南 URL 貼入 Copilot 中，並要求它為您實際使用的模型調整 `.github/copilot-instructions.md`、agent 設定檔或應用程式提示詞 | 減少重做 | 每次模型變更 10 分鐘 |
| 9 | **稽核您的 MCP 伺服器與注入的工具** — 停用未使用的 MCP 伺服器與增加技能/工具的 VS Code 擴充套件；對重複的工作流程使用乾淨的程式碼編寫設定檔或專用的自訂 agent。每個 MCP 工具在每個 agent 步驟中花費約 100-500 個 Token。若在此之後 Shell 輸出仍然很大，請評估如 [RTK](https://github.com/rtk-ai/rtk) 或 [`snip`](https://github.com/edouard-claude/snip) 等輸出篩選器 | 移除工具/schema 開銷與雜亂的命令輸出 | 5-10 分鐘 |
| 10 | **在 AI 作業前將豐富檔案轉換為 Markdown** — `.docx`、`.pdf`、`.pptx`、`.xlsx`、HTML、影像、音訊、影片和 ZIP 都帶有格式稅。[Marc Bara 的文章](https://medium.com/@marc.bara.iniesta/your-docx-is-wasting-33-of-your-ai-budget-86a3d229d042) 展示了其成本；在聊天、agent 或 RAG 攝取前請使用 [Microsoft MarkItDown](https://github.com/microsoft/markitdown) | 減少雜亂的輸入 context | 5 分鐘 |
| 11 | **每週執行 `/chronicle cost tips` 與 `/chronicle improve`**（**僅限 Copilot CLI**，實驗性）— 這些斜線命令可在互動式 Copilot CLI 工作階段中使用（非 VS Code），非一般的 Copilot Chat 功能。`cost tips` 分析您的 Token 支出並建議削減方式；`improve` 尋找您 CLI 工作階段歷史中重複出現的困惑，並生成自訂說明修正案，使相同的誤解意圖不再持續花費 Token | 削減重複重做與直接 Token 支出 | 每次執行 2 分鐘 |
| 12 | **使用 [Tokentop](https://github.com/tokentopapp/tokentop) 獲得即時 Token 可視化** — 其本地終端機儀表板可顯示 Copilot CLI 及其他受支援 agent 的工作階段、模型、Token、成本與消耗率資料。在優化前設定預算警報 | 使 Token 浪費與成本飆升可視化；不壓縮提示詞或輸出 | 5 分鐘 |
| 13 | **針對長工具鏈嘗試 CodeAct**（**僅限 Copilot CLI**，可選外掛）— [`copilot-codeact-plugin`](https://github.com/jsturtevant/copilot-codeact-plugin) 將多步驟工具鏈壓縮為一次沙盒執行，這可以減少系統提示詞、先前訊息與工具定義的重複重播 | 減少工具迴圈重播 | 10-15 分鐘 |
| 14 | **先規劃，然後在新工作階段中執行** — 使用規劃模式 (CLI) 或 Ask 模式 (VS Code) 與強大模型達成方案共識，將計畫儲存至 `plan.md` 或 issue，然後從該計畫在新工作階段中執行 — 通常使用較便宜的模型。首次即達到正確結果可避免 agent 朝錯誤方向編寫程式碼的昂貴重做。參閱 [先規劃後執行 §2.5.9](docs/06-workflow-optimization.md#259-plan-first-then-execute-and-route-the-phases) 與 [每個 Token 的成果](docs/13-outcome-per-token.md) | 避免錯誤方向的重做；更便宜的執行管道 | 0 分鐘（只需排定工作順序） |
| 15 | **使用 Graphify 建立持久的程式碼庫圖譜**（可選，VS Code + Copilot CLI）— [`graphify`](https://github.com/Graphify-Labs/graphify) 使用 tree-sitter AST 對儲存庫進行一次性對映並寫入 `graphify-out/graph.json`；Agent 查詢圖譜而非每個工作階段重新讀取專案檔案。最適合定向讀取佔據 agent 主要輸入的大型儲存庫。安裝方式：`uv tool install graphifyy` | 減少重複的檔案讀取輸入 | 5-10 分鐘 |

**若您是從企業或客戶治理角度，而非個人設定角度來看待此問題？** 請從 [企業治理](docs/12-enterprise-governance.md) 開始。該章節涵蓋了 AI 點數預算、單一使用者緊縮、模型存取策略、組織說明以及獨立組織的權衡。

*上述數字均限定於每行所列的機制，不可累加，且不等於總帳單的減少量。*

輸出控制（#1, #2）會立即產生回報並發揮複利效果 — 一次設定，每次呼叫皆能節省。結構性輸入控制（#3, #6）在每次互動中發揮複利效果。模型路由（#4, #5）在計費層級降低成本。模型特定的提示詞調整（#8）透過提升初次生成品質來減少浪費。MCP 稽核（#9）消除每個 agent 任務中數以千計的隱藏 Token；RTK/snip 風格的輸出篩選器可解決冗長 Shell 結果的額外成本。Markdown 轉換（#10）在模型看到之前就移除 DOCX/PDF/HTML 的版面配置雜訊。基於圖譜的導航（#15）一次性預先載入程式碼庫定向，然後在各個 agent 工作階段中重複使用。

![先規劃後便宜地執行：使用強大的模型進行規劃，儲存計畫，然後在全新且較便宜的管道中執行並驗證驗收條件。](docs/assets/diagrams/plan-execute-cheaply.svg)

---

## 指南目錄

### 第 1 部分：為何 Token 至關重要

瞭解 BPE tokenization、為何 Token 對於成本/速度/限制至關重要，以及 GitHub Copilot 如何在幕後使用 Token。

→ **[閱讀第 1 部分](docs/01-why-tokens-matter.md)**

---

### 第 2 部分：各項技術

#### [2.1 提示詞壓縮](docs/02-prompt-compression.md)

穴居人式說話 (Caveman-speak)、精準度等級（輕度/完整/極限）、結構化格式、縮寫以及以程式碼為中心的提示。節省 30-50% 輸入 Token；結合輸出控制 (2.4) 可節省輸出成本。

#### [2.2 語言比較](docs/03-language-comparison.md)

數據支持的比較：在這些範例中，英文是最具 Token 效率的語言。中文/日文/韓文 (CJK) 成本高出 1.7-2.4 倍。包含 8 種語言的 tokenization 表格。

#### [2.3 Context 管理](docs/04-context-management.md)

壓縮系統說明、壓縮記憶體檔案、使用 `applyTo` 限定 context 範圍、關閉未使用的編輯器分頁、在 AI 作業前將非文字檔案轉換為 Markdown、設定內容排除 (Content Exclusion)（Business/Enterprise 管理員）、開啟全新對話。控制傳送到模型的內容。

#### [2.4 輸出控制](docs/05-output-control.md)

"Code only, no explanation." 限制回應格式。將精簡輸出設定為專案預設值。

#### [2.5 工作流程優化](docs/06-workflow-optimization.md)

精簡 commit 訊息、單行 PR 審查、Ask 與 Agent 模式選擇、模型特定的提示詞調整，以及何時「不」進行壓縮。

#### [2.6 常駐 Context 問題](docs/07-agents-md-problem.md)

針對 LLM 生成 context 檔案的研究表明，它們通常在增加 Token 成本的同時損害 agent 的正確性。相同的教訓適用於 `AGENTS.md` 和 `.github/copilot-instructions.md` — 它利於不同的慣例（不同的檔名、不同的歷史所有者），但今天都是 Copilot 的常駐 context。將「僅留地雷區」方法應用於您儲存庫使用的任何檔案。將 context 檔案視為 Issue 追蹤器，而非 Wiki。

#### [2.7 MCP 與工具成本](docs/08-mcp-tool-costs.md)

隱藏的 Token 稅：每個 MCP 工具在每個 agent 步驟中花費 100-500 個 Token。15 個伺服器 × 15 個步驟 = 26.5 萬個 Token 的開銷。涵蓋 MCP 稽核、Copilot 控制基準、RTK、snip、minimal-context-tools 以及相鄰的輸出/context 壓縮工具。

---

### 第 3 部分：比較與資料

正面提示詞比較、語言 tokenization 表格、完整的技術對照矩陣（40+ 技術），以及帶有邊際效益遞減曲線的品質影響評估。

→ **[閱讀第 3 部分](docs/09-comparisons-data.md)**

---

### 第 4 部分：實作設定

循序漸進：設定 Copilot、優化 Coding Agent、設定 Agent 模式並建立習慣。包含 VS Code 設定、決策框架以及 4 週導入計畫。

→ **[閱讀第 4 部分](docs/10-practical-setup.md)**

---

### 隨附章節：模型選擇與計價

專用頁面，討論模型、PRU 時代乘數歷史、當前 Auto 指導方針、方案可用性，以及在 Copilot 確切的各模型 UBB 表格尚未公布時，廠商輸入/輸出 Token 定價的定位。包含連至 Auto 模型選擇、計費與方案/模型可用性官方 GitHub Docs 頁面的連結。

→ **[閱讀隨附章節](docs/11-models-and-pricing.md)**

---

### 隨附章節：企業治理

面向客戶管理員指導方針的專用章節：用量計費安全邊界、AI 點數預算、支出群組、FinOps-as-code 自動化、模型存取策略、組織層級說明以及獨立組織的權衡。

→ **[閱讀隨附章節](docs/12-enterprise-governance.md)**

---

### 隨附章節：每個 Token 的成果

專用章節，討論從 Token 最小化轉向每個 Token 的價值：先規劃後執行、提示詞技能進階、模型路由、基準測試注意事項，以及用量計費下的日常模型選擇。

→ **[閱讀隨附章節](docs/13-outcome-per-token.md)**

---

需要術語表、快速術語、工具或核心外部連結？請前往 [指南首頁](docs/index.md)。

---

## 最高影響力的技術

依成本影響力排序。輸出優先 — 其每 Token 成本比輸入貴 5 倍。

1. **輸出控制** — "Code only, no explanation" + `copilot-instructions.md` 中的精簡預設。程式碼任務可節省 40-70% 輸出，所有互動中節省 30-60%。一條指令，永久生效。
2. **縮減常駐 context** (`copilot-instructions.md` + `AGENTS.md`) — 壓縮無用填料，修剪至僅包含地雷區，刪除 LLM 生成的樣板。在每次互動與 agent 步驟中發揮複利效果；Agent 任務減少 20-23% 且提高正確性
3. **簡單問題使用 Ask 模式** — 避免 Agent 開銷以節省 60-90%
4. **稽核 MCP 伺服器與注入的工具** — 停用未使用的伺服器/擴充套件，或使用乾淨的程式碼編寫設定檔/自訂 agent，以在每個 agent 任務中節省 5K-190K Token
5. **Auto 模型選擇** — 較低成本的預設路由加符合條件用量的付費方案折扣，零努力
6. **先將豐富檔案轉換為 Markdown** — 避免在聊天、agent 與 RAG 工作流程中為 Word/PDF/HTML 版面配置雜訊付費
7. **建立持久的程式碼庫圖譜** — 在大型儲存庫上使用 Graphify，使 Agent 查詢 `graph.json` 而非每個工作階段重新讀取結構檔案
8. **針對目標模型重新調整提示詞** — 更好的初次生成輸出可減少重複的澄清來回
9. **精準提示詞** — 佔使用者提示詞輸入 Token 的 20-40%；對品質的重性高於單純的原始節省

---

*這是一份動態文件。隨著 tokenization 技術的發展、模型能力的改變以及新技術的出現，本指南將持續更新。請檢查儲存庫以取得最新版本。*
