# Token 最佳化指南

減少花在不必要的上下文、冗長敘述與重工上的 token，同時保留完成並驗證任務所需的資訊。

<div class="guide-entry" markdown="1">

<div class="guide-entry__actions" markdown="1">

[從實務設定開始](10-practical-setup.md){ .md-button .md-button--primary }
[了解 token 成本](01-why-tokens-matter.md){ .md-button }

</div>

<div class="guide-entry__presentations" markdown="1">

[實務工作者簡報](https://olivomarco.github.io/github-copilot-token-optimization/slides/briefing.html)
[完整 8 小時工作坊](https://olivomarco.github.io/github-copilot-token-optimization/slides/index.html)

</div>

</div>

社群指引，並非 GitHub 或 Microsoft 的官方政策。

## 找出浪費的來源

| 機制 | 閱讀章節 |
|---|---|
| 回應長度與格式 | [輸出控制](05-output-control.md) |
| 請求清晰度與簡寫 | [提示詞壓縮](02-prompt-compression.md)與[語言比較](03-language-comparison.md) |
| 載入的上下文、指示範圍、檔案轉換、快取／交接 | [上下文管理](04-context-management.md) |
| 常駐指示的相關性 | [常駐上下文問題](07-agents-md-problem.md) |
| 工具定義、命令輸出與程式碼庫地圖 | [MCP 與工具成本](08-mcp-tool-costs.md) |
| 模式與特定工作流程的習慣 | [工作流程最佳化](06-workflow-optimization.md) |
| 規劃、驗收檢查與重工 | [單位 Token 產出](13-outcome-per-token.md) |
| 模型費率、路由與投入程度 | [模型選擇與定價](11-models-and-pricing.md) |
| 組織預算與存取權 | [企業治理](12-enterprise-governance.md) |
| 證據與比較 | [比較與資料](09-comparisons-data.md) |

## 從簡潔開始；測試更嚴格的風格規則

一般的簡潔語言是預設做法。Caveman 仍是可選方案，但較短的敘述並不保證代理程式任務會更便宜。在採用它來節省成本之前，請先閱讀[限定範圍的證據與三組比較](05-output-control.md#caveman-optional-output-style)。

<details markdown="1">
<summary>術語與參考連結</summary>

### 快速術語

- **輸入（Input）**：提示詞，加上載入的指示、檔案、歷史記錄與工具定義。
- **輸出（Output）**：產生的 token；可見的文字內容只是代理程式工作的一部分。
- **快取輸入（Cached input）**：符合條件而被重複使用的上下文，依適用的快取費率計費。
- **UBB**：用量計費（usage-based billing）。請查閱目前的方案條款與 AI 點數規則。
- **模式（Mode）**：工作流程介面，例如 Ask、Edit 或 Agent；與模型選擇無關。
- **格式稅（Format tax）**：版面配置或擷取雜訊，可能增加輸入卻無助於任務。

### 參考連結

- [GitHub Copilot 官方文件](https://docs.github.com/copilot)
- [組織與企業的用量計費](https://docs.github.com/en/copilot/concepts/billing/usage-based-billing-for-organizations-and-enterprises)
- [Awesome GitHub Copilot Customizations](https://github.com/github/awesome-copilot-customizations)
- [OpenAI Tokenizer](https://platform.openai.com/tokenizer)
- [輸出風格的證據與來源](05-output-control.md#sources)
- [工具目錄與設定](08-mcp-tool-costs.md)
- [CLI 教練與 VS Code 分析](06-workflow-optimization.md#257-close-the-loop-with-chronicle)

</details>

衡量必須有基準線、工作負載、模型與單位。不同機制所達成的 token 減幅不能相加；請參閱[比較與資料](09-comparisons-data.md)。
