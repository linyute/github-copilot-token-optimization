# GitHub Copilot 的 Token 最佳化指南

減少不必要的輸入、輸出與重工。評估調整成效時，應以實際完成的有效工作與總成本為準，而非只看 token 數量。

> [!IMPORTANT]
> 本文為社群指引，並非 GitHub 或 Microsoft 的官方政策。請至[官方 Copilot 文件](https://docs.github.com/copilot)確認產品行為、定價與組織控制設定。

## 從這裡開始

[實務設定](docs/10-practical-setup.md)是實作檢查清單。[為什麼 Token 很重要](docs/01-why-tokens-matter.md)則說明成本模型。

先從簡潔的白話文、相關的內容脈絡，以及合適的模式／模型開始。只有在確實找出浪費之處時，才加入工具或更嚴格的風格規則。

![Token 成本剖析：輸入包含隱藏的內容脈絡；輸出與符合資格的快取輸入各有獨立費率。](../source/docs/assets/diagrams/token-cost-anatomy.svg)

## 選擇適用的機制

| 需要處理的問題 | 對應章節 |
|---|---|
| 回答過長或選用 Caveman 風格 | [輸出控制](docs/05-output-control.md) |
| 不清楚或冗長的請求 | [提示詞壓縮](docs/02-prompt-compression.md) |
| 語言／分詞器的取捨 | [語言比較](docs/03-language-comparison.md) |
| 已載入的檔案、限定範圍的指示、Markdown 轉換、快取／交接 | [內容脈絡管理](docs/04-context-management.md) |
| 常駐指引是否值得保留 | [常駐內容脈絡問題](docs/07-agents-md-problem.md) |
| MCP 額外負擔、RTK/snip 與 Graphify 設定 | [MCP 與工具成本](docs/08-mcp-tool-costs.md) |
| Ask/Edit/Agent 的選擇與各工作流程的專屬習慣 | [工作流程最佳化](docs/06-workflow-optimization.md) |
| 計畫、驗證與避免重工 | [每個 Token 的成果](docs/13-outcome-per-token.md) |
| 模型路由、推理強度、可用性與定價 | [模型選擇與定價](docs/11-models-and-pricing.md) |
| 預算、存取原則與企業控制 | [企業治理](docs/12-enterprise-governance.md) |
| 如何比較各種主張與量測結果 | [比較與資料](docs/09-comparisons-data.md) |

## Caveman：選用，而非預設

請先從一般的簡潔指示開始。[證據審查](docs/05-output-control.md#caveman-optional-output-style)會將輸出長度的變化與整體任務成本區分開來，並說明為何較早期的亮眼宣稱並不足以證明你的工作負載也能節省成本。

請使用[預設／簡潔／Caveman 通訊協定](docs/05-output-control.md#compare-default-concise-and-caveman)，測試在 `Answer concisely.` 之外是否還有額外價值。本指南尚未自行執行付費的三組對照基準測試。

## 簡報與參考資料

- [實務工作者簡報（18 張投影片）](../source/slides/briefing.html)
- [完整客戶工作坊（8 小時）](../source/slides/index.html)
- [指南首頁與參考連結](docs/index.md)

節省幅度取決於工作負載、模型、主機環境與計費方式。請勿將不同機制的百分比相加，也不要把僅限輸出的結果當成帳單的減少。
