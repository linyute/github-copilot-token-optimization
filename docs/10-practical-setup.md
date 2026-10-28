# 第 4 部分：實務設定

[← 返回指南](index.md)

請使用這份檢查清單來套用本指南。說明、證據與工具安裝請參閱所連結的章節。

## 步驟 1：設定簡潔的回應預設值

在你的儲存庫中建立 `.github/copilot-instructions.md`：

```bash
mkdir -p .github
touch .github/copilot-instructions.md
```

從一般的簡潔語言開始：

```markdown
先給答案。說明的篇幅要與任務成比例。
省略問候語、重複的摘要，以及主動提出的後續協助。
保留技術術語、否定、條件、數字、單位、程式碼與命令。
當簡短會使動作變得模稜兩可時，請使用完整的句子。
對於程式碼變更，請回報變更了什麼，以及相關的測試結果或阻礙因素。
```

這是一個起點，並非經過量測的節省保證。關於輸出格式與選用的 Caveman 風格，請參閱[輸出控制](05-output-control.md)。

## 步驟 2：只加入專案專屬的指引

請保留代理程式需要、但無法可靠地從程式碼中發現的規則。將特定路徑的規則放在限定範圍的指示檔案中，偶爾才用到的檢查清單則放在隨需指引中。

擺放位置請參閱[內容脈絡管理](04-context-management.md#scoping-guidance)，應保留哪些內容請參閱[常駐內容脈絡問題](07-agents-md-problem.md#265-what-to-actually-put-in-context-files)。

## 步驟 3：選擇模式、模型與內容脈絡

在長時間執行之前：

1. 使用[模式決策指南](06-workflow-optimization.md#253-ask-mode-vs-agent-mode)來決定是否需要工具。
2. 選擇[模型與支援的推理強度等級](11-models-and-pricing.md#good-default-heuristics)。
3. 關閉不相關的檔案，並只附加相關的內容脈絡。請使用 [Markdown 工作流程](04-context-management.md#normalize-non-text-inputs-to-markdown-first)轉換富格式文件。
4. 檢查[已載入的工具](08-mcp-tool-costs.md#before-you-optimize-measure-whats-actually-loaded)；停用未使用的伺服器與擴充功能提供的工具。
5. 當必須變更工作階段設定時，請遵循[快取與交接指引](04-context-management.md#235-caching-store-and-reuse-context-within-prompts)。

在設定時，只選擇工作負載所需的項目：針對雜亂的 shell 結果，選擇 [RTK](08-mcp-tool-costs.md#277-compress-tool-output-at-the-source-rtk) **或** [snip](08-mcp-tool-costs.md#278-rtk-alternative-snip)；針對重複的結構性導覽，選擇 [Graphify](08-mcp-tool-costs.md#2711-persistent-codebase-maps-graphify)。

## 步驟 4：讓 Coding Agent 任務可重現

### 準備環境

使用 `copilot-setup-steps.yml` 以確定性的方式預先安裝相依項目。請讓設定專屬於該儲存庫，而不是要求代理程式重新探索。

### 撰寫精確的 issue

請指出失敗情形、位置、預期行為與測試：

```text
Bug：當電子郵件包含 '+' 字元時，登入會失敗。
檔案：src/auth/login.ts，第 42 行的 validateEmail()。
修正：在傳給 OAuth 提供者之前，先對電子郵件進行 URL 編碼。
測試：在 login.test.ts 中新增 "user+tag@example.com" 的測試案例。
```

請將此視為一個實作範例，而不是不經檢視程式碼就套用的診斷。

### 在適當時使用專注的代理程式設定檔

重複的工作流程可以使用精簡的工具清單與任務專屬的指示。請不要在每個設定檔中重複整個專案的指引。儲存庫中的 `agents/token-saver.agent.md` 是現有的精簡風格範例，並非經過基準測試的成本保證，也不是必要的預設值。

### 規劃與驗證

對於多步驟的工作，請使用[規劃／執行／驗證工作流程](13-outcome-per-token.md#plan-first-then-execute-cheaply)。定義驗收檢查、了解起始的測試基準線，並讓最終報告與證據相互對應。

## 步驟 5：檢視用量並調整

| 訊號 | 動作 |
|---|---|
| 反覆釐清或遺漏條件 | 使用更清楚的措辭，並檢查任務範圍 |
| 已載入內容脈絡的基準線偏大 | 檢視常駐指示與啟用中的工具 |
| 大型 shell 結果 | 嘗試一個輸出篩選器；確認錯誤訊息仍然有用 |
| 成本高昂但成果不佳的工作階段 | 檢視模型選擇、重試、計畫與驗收檢查 |
| 某項風格規則似乎有幫助 | 使用[三組對照通訊協定](05-output-control.md#compare-default-concise-and-caveman)，而不是假設的節省率 |

關於以歷程記錄為基礎的檢視，請參閱 [Copilot CLI `/chronicle`](06-workflow-optimization.md#257-close-the-loop-with-chronicle) 與 [VS Code AI Engineering Coach](06-workflow-optimization.md#258-vs-code-usage-analytics-ai-engineering-coach)。

## 組織推行

請在提示詞檔案之外設定[預算、每位使用者的限制與模型原則](12-enterprise-governance.md)。在擴大存取之前，請將支出與實際交付的工作一併檢視。

---

**下一章：**[模型選擇與定價 →](11-models-and-pricing.md)
