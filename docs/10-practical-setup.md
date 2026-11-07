# 實務設定

[← 回到指南](index.md)

使用這份檢查清單來落實本指南。說明、證據與工具安裝，請參閱連結的章節。

先從簡潔回應的預設值開始，再只套用與您的工作負載相關的變更。步驟 4 適用於 Coding Agent 任務；其他工作流程請略過。請使用[步驟 5](#step-5-review-usage-and-adjust) 檢視結果，而不要預先假設會有節省。

## 步驟 1：設定簡潔回應的預設值

在您的儲存庫中建立 `.github/copilot-instructions.md`：

```bash
mkdir -p .github
touch .github/copilot-instructions.md
```

從平實簡潔的語言開始：

```markdown
先給答案。說明的篇幅與任務相稱。
省略問候語、重複的摘要，以及未經詢問的後續協助提議。
保留技術術語、否定、條件、數字、單位、程式碼與命令。
當簡短會使動作產生歧義時，使用完整句子。
對於程式碼變更，回報變更內容以及相關的測試結果或阻礙。
```

這只是起點，並非經過衡量的節省保證。輸出格式與選用的 Caveman 風格，請參閱[輸出控制](05-output-control.md)。

## 步驟 2：只加入專案特有的指引

保留代理程式需要、但無法可靠地從程式碼中發現的規則。將特定路徑的規則放在限定範圍的指示檔案中，將偶爾使用的檢查清單放在隨選指引中。

放置方式請參閱[上下文管理](04-context-management.md#scoping-guidance)，要保留哪些內容請參閱[常駐上下文的問題](07-agents-md-problem.md#265-what-to-actually-put-in-context-files)。

## 步驟 3：選擇模式、模型與上下文

在長時間執行之前：

1. 使用[模式決策指南](06-workflow-optimization.md#253-ask-mode-vs-agent-mode)判斷是否需要工具。
2. 選擇[模型與支援的推理強度](11-models-and-pricing.md#good-default-heuristics)。
3. 關閉無關的檔案，只附加相關的上下文。使用 [Markdown 工作流程](04-context-management.md#normalize-non-text-inputs-to-markdown-first)轉換富格式文件。
4. 檢查[已載入的工具](08-mcp-tool-costs.md#before-you-optimize-measure-whats-actually-loaded)；停用未使用的伺服器與擴充功能提供的工具。
5. 當工作階段設定必須變更時，請遵循[快取與交接指引](04-context-management.md#235-caching-store-and-reuse-context-within-prompts)。

設定時，只選擇工作負載所需要的項目：針對雜訊多的 shell 結果，使用 [RTK](08-mcp-tool-costs.md#277-compress-tool-output-at-the-source-rtk) **或** [snip](08-mcp-tool-costs.md#278-rtk-alternative-snip)；針對重複的結構性導覽，使用 [Graphify](08-mcp-tool-costs.md#2711-persistent-codebase-maps-graphify)。

## 步驟 4：讓 Coding Agent 任務可重現

### 準備環境

使用 `copilot-setup-steps.yml` 以確定性的方式預先安裝相依項目。讓設定針對該儲存庫量身打造，而不是要求代理程式重新探索。

### 撰寫精確的 issue

指出失敗情形、位置、預期行為與測試：

```text
Bug：當電子郵件包含 '+' 字元時，登入失敗。
檔案：src/auth/login.ts，L42 的 validateEmail()。
修正：在傳給 OAuth 提供者之前，先對電子郵件進行 URL 編碼。
測試：在 login.test.ts 中新增 "user+tag@example.com" 的案例。
```

請將此視為實作範例，而不是不檢查程式碼就直接套用的診斷結果。

### 適當時使用專注的代理程式設定檔

重複性的工作流程可以使用精簡的工具清單與特定任務的指示。請勿在每個設定檔中重複整個專案的指引。儲存庫中的 `agents/token-saver.agent.md` 是現成的精簡風格範例，並非經過基準測試的成本保證，也不是必要的預設值。

### 規劃與驗證

對於多步驟的工作，請使用[規劃／執行／驗證工作流程](13-outcome-per-token.md#plan-first-then-execute-cheaply)。定義驗收檢查、掌握起始的測試基準，並讓最終報告緊扣證據。

## 步驟 5：檢視使用情況並調整

| 訊號 | 動作 |
|---|---|
| 重複釐清或缺少條件 | 使用更清楚的措辭，並檢查任務範圍 |
| 已載入上下文的基準量很大 | 檢視常駐指示與啟用中的工具 |
| shell 結果很大 | 嘗試一種輸出過濾器；確認錯誤訊息仍然有用 |
| 花費高昂但結果不佳的工作階段 | 檢視模型選擇、重試、計畫與驗收檢查 |
| 某項風格規則看似有幫助 | 使用[三組對照協定](05-output-control.md#compare-default-concise-and-caveman)，而不是假設的節省比例 |

若要依歷史記錄進行檢視，請參閱 [Copilot CLI `/chronicle`](06-workflow-optimization.md#257-close-the-loop-with-chronicle) 與 [VS Code AI Engineering Coach](06-workflow-optimization.md#258-vs-code-usage-analytics-ai-engineering-coach)。

## 組織推行

在提示詞檔案之外設定[預算、每位使用者的上限與模型政策](12-enterprise-governance.md)。在擴大存取權限之前，請將支出與已交付的工作一併檢視。

---

**下一步：** [尋找其餘浪費的改善建議](index.md#find-the-source-of-waste)或[檢視證據](09-comparisons-data.md)。模型費率與路由，請參閱[模型選擇與定價](11-models-and-pricing.md)。
