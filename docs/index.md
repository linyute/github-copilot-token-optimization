# Token 優化指南

減少 GitHub Copilot Token 支出的實踐指南，同時保持答案和代碼的實用性。

[從第 1 部分開始](01-why-tokens-matter.md){ .md-button .md-button--primary }
[跳轉到實踐設置](10-practical-setup.md){ .md-button }

## 本指南涵蓋內容

- 為什麼在用量計費 (UBB) 下 Token 使用量實際上會花錢
- 為什麼輸出控制在原始投資報酬率 (ROI) 上通常優於提示詞壓縮
- 如何縮減始終開啟的上下文、歷史記錄和工具開銷
- 模型特定的提示指南如何提高初次嘗試的品質並減少返工
- 何時使用詢問模式 (Ask)、編輯模式 (Edit) 和代理模式 (Agent) 在經濟上是合理的
- 如何在不依賴未記錄的控制項的情況下設置企業護欄
- 如何將本存儲庫轉化為可重複的團隊習慣

## 最快獲勝的方法

1. 預設約束輸出：`Code only, no explanation.` 以及 `No explanations unless asked.`
2. 保持 `.github/copilot-instructions.md` 簡短且具體。
3. 在長時間工作階段中保護快取：保持 `{ 模型, 啟用的 MCP 集合, 啟用的代理程式/設定檔 }` 穩定；若必須切換，請以簡短的交接摘要開始新的對話。
4. 對於不需要工具的問題，使用詢問模式 (Ask Mode)。
5. 根據目標模型的官方指南重新調整提示詞與說明。
6. 停用您不使用的 MCP 伺服器。
7. 在進行 AI 工作前，將 DOCX/PDF/Office/media 輸入轉換為 Markdown；可從 [MarkItDown](https://github.com/microsoft/markitdown) 開始。
8. 審計長時間運作的代理工作階段及反覆的來回溝通。
9. 安裝 [RTK](https://github.com/rtk-ai/rtk) — 一個 CLI 代理，可在輸出到達代理之前過濾 `git`、測試執行器、`grep` 及其他 100 多種 Shell 命令。只需一次安裝，即可在代理和程式碼代理工作階段中節省 60-90% 的工具呼叫結果 Token。

## 按主題閱讀

### 基礎知識

- [為什麼 Token 很重要](01-why-tokens-matter.md)
- [對比與數據](09-comparisons-data.md)

### 技術

- [上下文管理](04-context-management.md)
- [輸出控制](05-output-control.md)
- [工作流優化](06-workflow-optimization.md)
- [MCP 和工具成本](08-mcp-tool-costs.md)

### 實施

- [實踐設置](10-practical-setup.md)
- [模型選擇與定價](11-models-and-pricing.md)
- [企業治理](12-enterprise-governance.md)

## 快速術語

- **UBB**: 用量計費 (usage-based billing)。Copilot Business 和 Enterprise 的支出是通過 AI 額度使用情況而非請求計數器來追蹤的。
- **AI 額度 (AI credits)**: 切換後使用的池化計費單位。
- **自動模式 (Auto mode)**: Copilot 的預設模型選擇器。當你不需要固定模型時，這是一個很好的預設路徑。
- **詢問模式 (Ask Mode)**: 單步互動。處理簡單問題的最低開銷選擇。
- **代理模式 (Agent Mode)**: 多步互動。更高的槓桿，更高的成本。
- **內容排除 (Content Exclusion)**: 用於將選定的存儲庫內容排除在 Copilot 上下文之外的管理員控制項。
- **格式稅 (Format tax)**: 來自 DOCX、PDF、HTML、投影片、電子表格、圖像和音頻/視頻提取中的豐富文件元數據和佈局噪音所產生的額外 Token。請先轉換為 Markdown。

## 實用連結

- [GitHub Copilot 官方文檔](https://docs.github.com/copilot)
- [針對組織和企業的用量計費](https://docs.github.com/en/copilot/concepts/billing/usage-based-billing-for-organizations-and-enterprises)
- [OpenAI Tokenizer](https://platform.openai.com/tokenizer)
- [Awesome GitHub Copilot Customizations](https://github.com/github/awesome-copilot-customizations)
- [LLMLingua](https://github.com/microsoft/LLMLingua)
- [Caveman 專案](https://github.com/JuliusBrussee/caveman)
- [RTK — Rust Token Killer](https://github.com/rtk-ai/rtk)
- [Microsoft MarkItDown](https://github.com/microsoft/markitdown) —— 將 PDF、Office 文件、圖像、音頻、HTML、ZIP 內容、YouTube URL、EPUB 等轉換為 Markdown，供 LLM 工作流使用
- [Marc Bara: "Your .docx Is Wasting 33% of Your AI Budget"](https://medium.com/@marc.bara.iniesta/your-docx-is-wasting-33-of-your-ai-budget-86a3d229d042)
- [Dina Berry: "How I Cut Token Usage from 52% to 13%"](https://dfberry.github.io/2026-05-06-tuning-up-copilot-context) —— 來自 Copilot CLI 生產環境設置的真實測量數據 (Microsoft/GitHub 內容貢獻者)

## 備註

- `/chronicle` 僅適用於 **Copilot CLI**（也可在 JetBrains 中透過互動式 Copilot CLI 工作階段使用）。它**不**適用於 VS Code — 請在該處使用 [AI Engineering Coach](06-workflow-optimization.md#258-vs-code-usage-analytics-ai-engineering-coach)。子指令包括 `cost tips`、`improve`、`tips`、`standup`、`search` 和 `reindex`。
- 用量計費在本存儲庫中被標記為 **UBB**。
