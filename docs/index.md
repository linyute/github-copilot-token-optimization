# Token 優化指南

減少 GitHub Copilot Token 支出的實用指南，同時保持答案和程式碼的實用性。

[從第 1 部分開始](01-why-tokens-matter.md){ .md-button .md-button--primary }
[跳至實作設定](10-practical-setup.md){ .md-button }
<a class="md-button" href="https://linyute.github.io/github-copilot-token-optimization/slides/briefing.html">精簡實務人員簡報</a>
<a class="md-button" href="https://linyute.github.io/github-copilot-token-optimization/slides/index.html">完整 8 小時工作坊</a>

## 本文涵蓋內容

- 為何 Token 使用量在用量計費模式 (Usage-Based Billing) 下會產生實際費用
- 為何輸出控制在原始 ROI 上通常勝過提示詞壓縮
- 如何縮減常駐 context、歷史紀錄與工具開銷
- 特定模型的提示詞指南如何提升初次生成品質並減少重做
- 何時使用 Ask 模式、Edit 模式與 Agent 模式最符合經濟效益
- 如何在不依賴未支援控制項的情況下設定企業級安全邊界
- 如何將此儲存庫轉化為可重複的團隊習慣

## 最快獲勝法

1. 預設限制輸出：`Code only, no explanation.` 以及 `No explanations unless asked.`
2. 保持 `.github/copilot-instructions.md` 簡短且具體。
3. 在長時間工作階段中保護快取：保持 `{model, reasoning effort, loaded skills, active MCP/tool set, agent/profile}` 穩定；若必須變更其中一項，請帶著簡短的交接摘要開啟新的聊天。
4. 對不需要工具的簡單問題使用 Ask 模式。
5. 根據目標模型的官方指南重新調整提示詞和說明。
6. 停用未使用的 MCP 伺服器。
7. 在進行 AI 作業前，將 DOCX/PDF/Office/媒體輸入轉換為 Markdown；可先使用 [MarkItDown](https://github.com/microsoft/markitdown)。
8. 稽核長時間執行的 Agent 工作階段與重複的來回對話。
9. 安裝一個 Shell 輸出篩選器：[RTK](https://github.com/rtk-ai/rtk) 或 [`snip`](https://github.com/edouard-claude/snip)。這些 CLI 代理軟體會在 `git`、測試執行器、`grep`、建構工具以及其他命令輸出到達 Agent 之前進行篩選。每個命令路徑使用一個篩選層；預設情況下不要堆疊使用。
10. 使用 [Graphify](https://github.com/Graphify-Labs/graphify) 建立持久的程式碼庫圖譜 — 透過 tree-sitter AST 對程式碼進行一次性對映，寫入 `graphify-out/graph.json`，然後讓 Agent 查詢圖譜，而不是在每個工作階段中重新讀取專案檔案。安裝方式：`uv tool install graphifyy`。

## 按主題閱讀

### 基礎

- [為何 Token 至關重要](01-why-tokens-matter.md)

### 技術

- [提示詞壓縮](02-prompt-compression.md)
- [語言比較](03-language-comparison.md)
- [Context 管理](04-context-management.md)
- [輸出控制](05-output-control.md)
- [工作流程優化](06-workflow-optimization.md)
- [常駐 Context 問題](07-agents-md-problem.md)
- [MCP 與工具成本](08-mcp-tool-costs.md)

### 比較

- [比較與資料](09-comparisons-data.md)
- [每個 Token 成果](13-outcome-per-token.md)

### 實作

- [實作設定](10-practical-setup.md)
- [模型選擇與計價](11-models-and-pricing.md)
- [企業治理](12-enterprise-governance.md)

## 快速術語

- **UBB**：用量計費 (usage-based billing)。Copilot Business 和 Enterprise 的支出是透過 AI 點數 (AI-credit) 使用量進行追蹤，而非請求計數器。
- **AI 點數 (AI credits)**：切換後使用的統一定價計費單位。
- **Auto 模式**：Copilot 的預設模型選擇器。不需要固定模型時的良好預設選項。
- **Ask 模式**：單次互動。對於簡單問題開銷最低的選擇。
- **Agent 模式**：多步驟互動。槓桿更高，成本也更高。
- **內容排除 (Content Exclusion)**：管理員控制項，用於防止選定的儲存庫內容進入 Copilot context。
- **格式稅 (Format tax)**：來自 DOCX、PDF、HTML、簡報、試算表、影像和影音擷取中豐富檔案 metadata 及版面配置雜訊的額外 Token。請先轉換為 Markdown。

## 有用的連結

- [GitHub Copilot 官方文件](https://docs.github.com/copilot)
- [組織與企業的用量計費](https://docs.github.com/en/copilot/concepts/billing/usage-based-billing-for-organizations-and-enterprises)
- [OpenAI Tokenizer](https://platform.openai.com/tokenizer)
- [Awesome GitHub Copilot Customizations](https://github.com/github/awesome-copilot-customizations)
- [LLMLingua](https://github.com/microsoft/LLMLingua)
- [Caveman 專案](https://github.com/JuliusBrussee/caveman)
- [RTK — Rust Token Killer](https://github.com/rtk-ai/rtk)
- [snip](https://github.com/edouard-claude/snip) — 適用於 Copilot CLI 與其他 Agent Shell 的 YAML 可擴充 Shell 輸出篩選器
- [Tokentop](https://github.com/tokentopapp/tokentop) — 用於 Agent Token、成本與消耗率可視化的本地即時儀表板；支援 Copilot CLI
- [minimal-context-tools](https://github.com/SebastienDegodez/copilot-instructions/tree/main/plugins/minimal-context-tools) — 用於低 Context CLI 搜尋/查詢模式的技能套件
- [Graphify](https://github.com/Graphify-Labs/graphify) — 為您的程式碼庫建立持久的知識圖譜；Agent 可查詢 `graphify-out/graph.json` 而無需重新讀取檔案。支援 GitHub Copilot、VS Code 工作流程及其他助手。PyPI 套件：`graphifyy`
- [Microsoft MarkItDown](https://github.com/microsoft/markitdown) — 將 PDF、Office 檔案、影像、音訊、HTML、ZIP 內容、YouTube URL、EPUB 等轉換為適用於 LLM 工作流程的 Markdown
- [Marc Bara: "Your .docx Is Wasting 33% of Your AI Budget"](https://medium.com/@marc.bara.iniesta/your-docx-is-wasting-33-of-your-ai-budget-86a3d229d042)
- [Dina Berry: "How I Cut Token Usage from 52% to 13%"](https://dfberry.github.io/2026-05-06-tuning-up-copilot-context) — 來自 Copilot CLI 生產環境設定的真實測量數據 (Microsoft/GitHub 內容貢獻者)

## 備註

- `/chronicle` 僅適用於 **Copilot CLI**（亦可於 JetBrains 中透過互動式 Copilot CLI 工作階段使用）。其**不**適用於 VS Code — 請在該處使用 [AI Engineering Coach](06-workflow-optimization.md#258-vs-code-usage-analytics-ai-engineering-coach)。子命令包含 `cost tips`、`improve`、`tips`、`standup`、`search` 以及 `reindex`。
- 用量計費在本文庫中標記為 **UBB**。
