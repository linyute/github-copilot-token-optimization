# Token 最佳化指南

把 token 花在刀口上，減少不必要的內容脈絡、冗長敘述與重工。保留完成並驗證任務所需的資訊。

[從實務設定開始](10-practical-setup.md){ .md-button .md-button--primary }
[了解 token 成本](01-why-tokens-matter.md){ .md-button }
<a class="md-button" href="https://olivomarco.github.io/github-copilot-token-optimization/slides/briefing.html">實務工作者簡報</a>
<a class="md-button" href="https://olivomarco.github.io/github-copilot-token-optimization/slides/index.html">完整 8 小時工作坊</a>

本文為社群指引，並非 GitHub 或 Microsoft 的官方政策。

## 找出浪費的來源

| 機制 | 閱讀位置 |
|---|---|
| 回應長度與格式 | [輸出控制](05-output-control.md) |
| 請求的清晰度與簡寫 | [提示詞壓縮](02-prompt-compression.md)與[語言比較](03-language-comparison.md) |
| 已載入的內容脈絡、指示範圍、檔案轉換、快取／交接 | [內容脈絡管理](04-context-management.md) |
| 常駐指示的關聯性 | [常駐內容脈絡問題](07-agents-md-problem.md) |
| 工具定義、命令輸出與程式碼庫地圖 | [MCP 與工具成本](08-mcp-tool-costs.md) |
| 模式與各工作流程的專屬習慣 | [工作流程最佳化](06-workflow-optimization.md) |
| 規劃、驗收檢查與重工 | [每個 Token 的成果](13-outcome-per-token.md) |
| 模型費率、路由與推理強度 | [模型選擇與定價](11-models-and-pricing.md) |
| 組織預算與存取 | [企業治理](12-enterprise-governance.md) |
| 證據與比較 | [比較與資料](09-comparisons-data.md) |

請使用[實務設定](10-practical-setup.md)來套用這些選擇。該章節會連結到各主題的詳細說明。

## 先求簡潔；再測試更嚴格的風格規則

一般的簡潔用語是預設做法。Caveman 仍是可選的做法，但敘述變短並不保證代理程式任務就會變便宜。在以節省成本為目的採用之前，請先閱讀[限定範圍的證據與三組對照比較](05-output-control.md#caveman-optional-output-style)。

## 名詞速覽

- **輸入（Input）**：提示詞，加上已載入的指示、檔案、歷程記錄與工具定義。
- **輸出（Output）**：產生的 token；可見的文字內容只是代理程式工作的一部分。
- **快取輸入（Cached input）**：符合資格的重複使用內容脈絡，依適用的快取費率計費。
- **UBB**：使用量計費（usage-based billing）。請確認目前的方案條款與 AI 點數規則。
- **模式（Mode）**：工作流程介面，例如 Ask、Edit 或 Agent；與模型選擇是分開的。
- **格式稅（Format tax）**：版面配置或擷取雜訊，可能增加輸入卻對任務沒有幫助。

## 參考連結

- [官方 GitHub Copilot 文件](https://docs.github.com/copilot)
- [組織與企業的使用量計費](https://docs.github.com/en/copilot/concepts/billing/usage-based-billing-for-organizations-and-enterprises)
- [Awesome GitHub Copilot Customizations](https://github.com/github/awesome-copilot-customizations)
- [OpenAI Tokenizer](https://platform.openai.com/tokenizer)
- [輸出風格證據與來源](05-output-control.md#sources)
- [工具目錄與設定](08-mcp-tool-costs.md)
- [CLI 教練與 VS Code 分析](06-workflow-optimization.md#257-close-the-loop-with-chronicle)

量測需要基準線、工作負載、模型與單位。不同機制所帶來的 token 減少量不可相加；請參閱[比較與資料](09-comparisons-data.md)。
