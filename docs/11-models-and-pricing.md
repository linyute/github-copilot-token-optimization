# 模型選擇與定價

選擇能可靠完成任務、且費用最低的可用模型。請將重試與審查時間一併納入，而不只是單一請求的費率。

## 決定成本的因素

| 因素 | 檢查項目 |
|---|---|
| 模型 | 實際使用介面的輸入、快取輸入與輸出費率 |
| 推理強度 | 在模型提供此控制項時，額外的推理工作 |
| 上下文 | 全新輸入與符合資格的快取輸入之比較 |
| 工作流程 | 代理程式步驟、重試與重複的歷史記錄 |
| 方案與政策 | 模型可用性、折扣、預算與組織限制 |

請參閱 [GitHub 模型比較](https://docs.github.com/en/copilot/reference/ai-models/model-comparison)、[支援的模型](https://docs.github.com/en/copilot/reference/ai-models/supported-models)與[計費文件](https://docs.github.com/en/copilot/concepts/billing/copilot-requests)，以取得最新的可用性與收費。來自 [Anthropic](https://www.anthropic.com/pricing) 或 [OpenAI](https://openai.com/api/pricing/) 的直接 API 定價，並非 Copilot 帳單的估算依據。

[工作流程最佳化](06-workflow-optimization.md)涵蓋各種模式；[上下文管理](04-context-management.md#235-caching-store-and-reuse-context-within-prompts)涵蓋快取穩定性與交接。

## 良好的預設啟發法

| 任務 | 起點 | 何時升級 |
|---|---|---|
| 語法查詢、簡短說明、小幅編輯 | Auto 或快速、低成本的模型 | 答案有誤或缺少必要的上下文 |
| 具有明確驗收檢查的例行實作 | 已處理過類似工作的模型 | 重複失敗的成本超過使用更強的模型 |
| 跨檔案診斷或模糊不清的規劃 | 較強的推理模型 | 驗證後仍暴露出未解決的假設 |
| 架構或高風險審查 | 能力足夠的模型，加上明確的人工審查 | 需要專家審查；模型選擇無法取代 |
| 執行已核准的具體計畫 | 在全新的工作階段中嘗試較便宜的模型 | 實作需要新的設計決策 |

這些只是起點，並不保證某個模型等級適合每項任務。請根據您自己的工作，衡量被接受的成果。

### 自動選擇

Auto 會在該介面所支援的模型池中進行選擇。在支援的情況下，針對任務最佳化的 Auto 除了健康狀態與可用性外，也會考量任務的複雜度。請檢查其目前的模型池、政策限制與任何符合資格的折扣，而不要把它當成可存取所有高階模型。請參閱 [GitHub 的 Auto 選擇文件](https://docs.github.com/en/copilot/concepts/models/auto-model-selection)。

### 推理強度

在可設定強度的情況下，請依任務選擇。對直接了當的工作使用較低的強度；當更深入的推理能顯著改善驗收結果、足以抵銷額外的工作量時，再提高強度。簡潔的最終答案並不能顯示實際計費了多少推理。

請在長時間任務開始之前選好模型與強度。如果任務變動到足以需要不同的設定，請使用簡短的交接，而不是自動帶著整段歷史記錄。請遵循[快取工作流程](04-context-management.md#235-caching-store-and-reuse-context-within-prompts)。

## 政策與資料處理

可用性會因方案、預覽狀態、地理位置、組織政策與用戶端而異。選擇器中出現的模型，並不代表已自動核准用於受規範的工作。

請對照目前的[模型與定價規則](https://docs.github.com/en/copilot/reference/copilot-billing/models-and-pricing)，檢查舊版年度方案的存取權、區域限制、合規附加費，以及延伸上下文／強度的可用性。請勿將聊天的成本假設套用到程式碼完成或下一個編輯建議；請參閱[企業治理](12-enterprise-governance.md)。

在將敏感工作交給模型處理之前，請確認適用的供應商條款、資料保留控制與組織政策。GitHub 記載了[模型託管](https://docs.github.com/en/copilot/reference/ai-models/model-hosting)與[政策控制](https://docs.github.com/en/copilot/concepts/enterprise/policies)。請勿假設每個模型（尤其是預覽版）都適用相同的保留條款。例如，Fable 模型預設會保留提示詞與輸出；符合資格的企業需要另行核准並設定，才能取得有時間限制的零保留豁免。

## 以完成的工作來評判路由

請針對類似的任務，比較成本、經過時間、驗收檢查與重試次數。一個需要多次修復嘗試的便宜請求，成本可能高於一次成功的較強模型執行。

請將計費機制分開看待：模型費率、折扣、符合資格的快取與工作階段額外負擔，各自的作用方式不同。在合併節省估算之前，請參閱[比較與資料](09-comparisons-data.md)；組織層級的調節手段，請參閱[企業治理](12-enterprise-governance.md)。

---

**下一篇：** [企業治理 →](12-enterprise-governance.md)
