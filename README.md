# GitHub Copilot Token 優化指南

> [!IMPORTANT]
> **這不是 GitHub 或 Microsoft 的官方指南。** 本指南是由實際現場經驗產生的社群資源 — 從開發用途採用 AI 的從業者所觀察到的模式、測試過的技術以及學到的教訓。它反映了產業的回饋：由下而上收集的實用知識，而不是由上而下的產品文件。請使用它來了解優化策略，並根據客戶的情境調整適用的內容。官方指南位於 [docs.github.com/copilot](https://docs.github.com/copilot)。

> 一份實用的、資料驅動的指南，旨在減少 Token 消耗，同時保持程式碼品質。
> 涵蓋 Chat、Inline 和 Coding Agent 工作流程。

---

## 快速入門 — 現在可以做的 12 件事

> **2026 年 6 月 1 日 — 用量計費 (Usage-Based Billing, UBB) 已上線。** GitHub Copilot 現在根據實際 Token（輸入 + 輸出 + 快取）計費，這些 Token 從匯集的 AI 額度（Business 方案每席位 30 美元，Enterprise 方案每席位 70 美元）中提取，而不是根據請求次數計費。本指南中的每項技術都直接轉化為額度節省 — 而且對快取友善的習慣比以往任何時候都更重要。有關客戶護欄，請參閱 [企業治理 (Enterprise Governance)](docs/12-enterprise-governance.md)，有關模型成本指導，請參閱 [模型選擇與定價 (Model Selection & Pricing)](docs/11-models-and-pricing.md)。

> **輸出 Token 的成本遠高於輸入 Token。** 這是本指南中最重要的定價事實。Anthropic 的公開定價使這種不對稱性變得具體（每百萬 Token 輸入/輸出：Haiku 1 美元/5 美元，Sonnet 3 美元/15 美元，Opus 5 美元/25 美元）。Copilot 確切的每模型 UBB 定價表尚未公開，但 UBB 仍然使冗長的輸出成本成倍增加。大多數輸入 Token 來自檔案內容、歷史紀錄和工具結構定義 (schemas) — 而不是來自你輸入的內容。你輸入的提示詞僅佔總輸入的一小部分。從輸出控制開始，然後解決結構性輸入的獲益。

沒有時間閱讀完整指南？今天就執行這些操作以削減你的 Token 使用量：

| # | 行動 | 主要效果 | 設定時間 |
|---|--------|----------------|----------------|
| 1 | **請求僅程式碼的回應** — 在 `copilot-instructions.md` 中加入 `Code only, no explanation.`。最高每 Token 投資報酬率 (ROI)：輸出成本是輸入的 5 倍，且這能永久減少每個程式碼任務 40-70% 的輸出 | 縮減回應長度 | 0 分鐘 |
| 2 | **預設限制輸出格式** — 在 `copilot-instructions.md` 中加入 `Bullets over paragraphs. No explanations unless asked.` | 保持回答簡潔 | 0 分鐘 |
| 3 | **縮減始終開啟的上下文 (Always-on context)** — 壓縮 `copilot-instructions.md` 並僅保留 `AGENTS.md` 中的關鍵內容。這兩個檔案中的每個 Token 都會在每次互動（以及每個 Agent 步驟）中計費。去除贅字，刪除 Agent 可透過讀取程式碼發現的任何內容，刪除 LLM 生成的 `/init` 樣板 | 減少始終開啟的輸入/上下文 | 15 分鐘 |
| 4 | **預設使用自動模型選擇 (Auto model selection)** — 將 Auto 作為基準，因為它會從支援的 Auto 池中選擇，並提供付費方案折扣。當任務明確需要時，手動固定成本較高的模型。請參閱 [模型選擇與定價 (Model Selection & Pricing)](docs/11-models-and-pricing.md) | 降低合格用量的計費費率 | 0 分鐘 |
| 5 | **針對簡單問題使用提問模式 (Ask Mode)** — 保留 Agent 模式用於多步驟任務 | 避免 Agent 開銷 | 0 分鐘 (只需選擇正確的模式) |
| 6 | **使用 `applyTo:` 路徑範圍限制上下文** — 將一個大型指令檔案分割成多個小的範圍指令檔案，僅在相關時載入 | 減少始終開啟的輸入/上下文 | 15 分鐘 |
| 7 | **在提示詞中保持精確** — 使用 "Add null check to `getUser()`"，而不是 "Can you please look at this and maybe add some error handling?" 注意：你輸入的提示詞僅佔總輸入的一小部分；精確度對於品質的影響大於對原始 Token 節省的影響 | 提高任務針對性 | 0 分鐘 |
| 8 | **根據目標模型重新調整提示詞** — 提供者的提示引導會根據模型/版本而變化。將官方指南 URL 貼入 Copilot，並要求它為你實際使用的模型調整 `.github/copilot-instructions.md`、Agent 個人檔案或應用程式提示 | 減少重做 | 每個模型變更 10 分鐘 |
| 9 | **稽核你的 MCP 伺服器** — 停用你未使用的伺服器；每台伺服器在每個 Agent 步驟中成本約 100-500 Token | 移除工具/結構定義開銷 | 5 分鐘 |
| 10 | **在 AI 工作前將豐富檔案轉換為 Markdown** — `.docx`、`.pdf`、`.pptx`、`.xlsx`、HTML、圖片、音訊、影片和 ZIP 檔案帶有格式稅。[Marc Bara 的文章](https://medium.com/@marc.bara.iniesta/your-docx-is-wasting-33-of-your-ai-budget-86a3d229d042)顯示了成本；在聊天、Agent 或 RAG 攝取之前使用 [Microsoft MarkItDown](https://github.com/microsoft/markitdown) | 減少雜亂的輸入上下文 | 5 分鐘 |
| 11 | **每週執行 `/chronicle improve`** (**僅限 Copilot CLI**，實驗性) — 此斜線指令在互動式 Copilot CLI 工作階段中運作，不作為一般的 Copilot Chat 功能。它會找出 CLI 工作階段歷史紀錄中反覆出現的混淆，並生成自定義指令修正，使同樣的誤解不再浪費 Token | 減少重複的重做 | 每次執行 2 分鐘 |
| 12 | **在長工具鏈中嘗試 CodeAct** (**僅限 Copilot CLI**，選配外部外掛程式) — [`copilot-codeact-plugin`](https://github.com/jsturtevant/copilot-codeact-plugin) 將多步驟工具鏈摺疊成一次沙箱執行，這可以減少系統提示、先前訊息和工具定義的重複重播 | 減少工具迴圈重播 | 10-15 分鐘 |

**是從企業或客戶治理的角度而不是個人設定的角度來看待這件事嗎？** 請參閱 [企業治理 (Enterprise Governance)](docs/12-enterprise-governance.md)。該章節涵蓋了 AI 額度預算、每使用者緊縮、模型存取政策、組織指令以及獨立組織權衡。

*上述數值僅針對各列中所述的機制，不可累加，也不代表總帳單減少量。*

輸出控制 (#1, #2) 立即生效並產生複利效果 — 設定一次，每次呼叫都能節省。結構性輸入控制 (#3, #6) 在每次互動中產生複利效果。模型路由 (#4, #5) 在計費層級降低成本。針對特定模型的提示調校 (#8) 透過提高首次產出的品質來減少浪費。MCP 稽核 (#9) 在每個 Agent 任務中消除數千個隱藏的 Token。Markdown 轉換 (#10) 在模型看到檔案之前去除 DOCX/PDF/HTML 的版面配置雜訊。

---

## 指南內容

### 第 1 部分：為什麼 Token 很重要

了解 BPE Token 化、為什麼 Token 對成本/速度/限制很重要，以及 GitHub Copilot 如何在幕後使用 Token。

→ **[閱讀第 1 部分](docs/01-why-tokens-matter.md)**

---

### 第 2 部分：技術

#### [2.1 提示詞壓縮 (Prompt Compression)](docs/02-prompt-compression.md)

原始人語 (Caveman-speak)、強度級別 (精簡/完整/極限)、結構化格式、縮寫以及以程式碼為中心的提示。可節省 30-50% 的輸入 Token；結合輸出控制 (2.4) 可節省輸出 Token。

#### [2.2 語言比較 (Language Comparison)](docs/03-language-comparison.md)

資料支持的比較：在這些範例中，英文是 Token 效率最高的語言。CJK (中日韓) 語言的成本高出 1.7-2.4 倍。包含 8 種語言的 Token 化表格。

#### [2.3 上下文管理 (Context Management)](docs/04-context-management.md)

壓縮系統指令、壓縮記憶體檔案、使用 `applyTo` 限制上下文範圍、關閉未使用的編輯器分頁、在 AI 工作前將非文字檔案轉換為 Markdown、設定內容排除 (Content Exclusion，適用於 Business/Enterprise 管理員)、開啟新對話。控制發送給模型的內容。

#### [2.4 輸出控制 (Output Control)](docs/05-output-control.md)

"Code only, no explanation." 限制回應格式。將簡潔的輸出設定為專案預設值。

#### [2.5 工作流程優化 (Workflow Optimization)](docs/06-workflow-optimization.md)

簡潔的提交訊息 (commit messages)、單行 PR 審閱、提問 (Ask) 與 Agent 模式選擇、特定模型的提示調校，以及何時「不要」壓縮。

#### [2.6 始終開啟的上下文問題 (The Always-On Context Problem)](docs/07-agents-md-problem.md)

對 LLM 生成的上下文檔案的研究顯示，它們通常會損害 Agent 的正確性，同時增加 Token 成本。同樣的教訓也適用於 `AGENTS.md` 和 `.github/copilot-instructions.md` — 它們是不同的慣例（不同的檔名、不同的歷史擁有者），但目前在 Copilot 中都作為始終開啟的上下文運作。對你的儲存庫所使用的檔案套用「僅保留關鍵內容 (landmines only)」的方法。將上下文檔案視為錯誤追蹤器 (bug tracker)，而不是維基 (wiki)。

#### [2.7 MCP 與工具成本 (MCP & Tool Costs)](docs/08-mcp-tool-costs.md)

隱藏的 Token 稅：每個 MCP 工具在每個 Agent 步驟中成本為 100-500 Token。15 台伺服器 × 15 個步驟 = 26.5 萬 Token 的開銷。包含稽核指南。

---

### 第 3 部分：比較與資料

對比提示詞比較、語言 Token 化表格、完整的技術矩陣（40 多種技術），以及帶有收益遞減曲線的品質影響評估。

→ **[閱讀第 3 部分](docs/09-comparisons-data.md)**

---

### 第 4 部分：實作設定

循序漸進：設定 Copilot、優化 Coding Agent、設定 Agent 模式並養成習慣。包含 VS Code 設定、決策框架和為期 4 週的採用計畫。

→ **[閱讀第 4 部分](docs/10-practical-setup.md)**

---

### 第 4.2 部分：模型選擇與定價

專門介紹模型的頁面、PRU 時代的倍數歷史、目前的 Auto 導向、方案可用性，以及當 Copilot 確切的每模型 UBB 表格尚未發布時，供應商輸入/輸出 Token 定價的適用之處。包含指向 GitHub 官方文件的連結，涵蓋自動模型選擇、計費以及方案/模型可用性。

→ **[閱讀第 4.2 部分](docs/11-models-and-pricing.md)**

---

### 第 4.3 部分：企業治理

針對面向客戶的管理員指導專章：用量計費護欄、AI 額度預算、每使用者緊縮、模型存取政策、組織層級指令，以及何時獨立組織值得這些開銷。

→ **[閱讀第 4.3 部分](docs/12-enterprise-governance.md)**

---

需要術語表、快速術語、工具或核心外部連結嗎？請前往 [指南首頁](docs/index.md)。

---

## 最高影響力的技術

按成本影響排序。輸出優先 — 每 Token 成本是輸入的 5 倍。

1. **輸出控制** — "Code only, no explanation" + 在 `copilot-instructions.md` 中設定簡潔預設值。程式碼任務可節省 40-70% 的輸出，所有互動中平均節省 30-60%。一次指令，永久生效。
2. **縮減始終開啟的上下文** (`copilot-instructions.md` + `AGENTS.md`) — 壓縮贅字，僅保留關鍵內容，刪除 LLM 生成的樣板。在每次互動和 Agent 步驟中產生複利效果；減少 20-23% 的 Agent 任務開銷並提高正確性。
3. **針對簡單問題使用提問模式 (Ask Mode)** — 透過避免 Agent 開銷可節省 60-90%。
4. **稽核 MCP 伺服器** — 停用未使用的伺服器，每個 Agent 任務可節省 5,000 到 190,000 Token。
5. **自動模型選擇** — 預設導向較低成本的模型，並在合格用量上提供付費方案折扣，無需額外努力。
6. **先將豐富檔案轉換為 Markdown** — 避免在聊天、Agent 和 RAG 工作流程中為 Word/PDF/HTML 的版面配置雜訊付費。
7. **根據目標模型重新調整提示詞** — 更好的首次產出可減少重複的釐清過程。
8. **精確的提示詞** — 佔使用者提示詞輸入 Token 的 20-40%；對於品質的影響大於對原始節省的影響。

---

*這是一份動態文件。隨著 Token 化技術的演進、模型能力的改變以及新技術的出現，本指南將會持續更新。請查看儲存庫以獲取最新版本。*
