# 模型選擇與定價

請選擇能可靠完成任務、且費用最低的可用模型。請將重試與審查時間也納入考量，而不只是單一請求的費率。

## 決定成本的因素

| 因素 | 應檢查的項目 |
|---|---|
| 模型 | 實際使用介面的輸入、快取輸入與輸出費率 |
| 推理強度 | 在模型提供此控制項時，額外的推理工作 |
| 內容脈絡 | 全新輸入與符合資格的快取輸入 |
| 工作流程 | 代理程式步驟、重試與重複的歷程記錄 |
| 方案與原則 | 模型可用性、折扣、預算與組織限制 |

請使用 [GitHub 模型比較](https://docs.github.com/en/copilot/reference/ai-models/model-comparison)、[支援的模型](https://docs.github.com/en/copilot/reference/ai-models/supported-models)，以及[計費文件](https://docs.github.com/en/copilot/concepts/billing/copilot-requests)，以了解目前的可用性與收費。來自 [Anthropic](https://www.anthropic.com/pricing) 或 [OpenAI](https://openai.com/api/pricing/) 的 API 直接標價，並不是 Copilot 帳單的估算。

[工作流程最佳化](06-workflow-optimization.md)涵蓋模式；[內容脈絡管理](04-context-management.md#235-caching-store-and-reuse-context-within-prompts)涵蓋快取穩定性與交接。

## 良好的預設經驗法則

| 任務 | 起始選擇 | 何時升級 |
|---|---|---|
| 語法查詢、簡短說明、小幅編輯 | Auto 或快速、便宜的模型 | 答案錯誤或缺少所需的內容脈絡 |
| 具有明確驗收檢查的例行實作 | 曾處理過類似工作的模型 | 重複失敗的成本超過更強模型的成本 |
| 跨檔案診斷或模稜兩可的規劃 | 較強的推理模型 | 驗證後仍暴露出未解決的假設 |
| 架構或高風險的審查 | 能力足夠的模型，加上明確的人工審查 | 需要專家審查；模型選擇無法取代 |
| 執行已核准的具體計畫 | 在全新工作階段中嘗試較便宜的模型 | 實作需要新的設計決策 |

這些是起始選擇，並不保證某個模型等級適合每項任務。請在你自己的工作上量測已被接受的成果。

### Auto 選擇

Auto 會在該介面支援的集區中選擇。在支援的情況下，針對任務最佳化的 Auto 除了健全度與可用性之外，也會考量任務的複雜度。請檢查其目前的集區、原則限制，以及任何符合資格的折扣，而不要把它當作可存取每一個高階模型。請參閱 [GitHub 的 Auto 選擇文件](https://docs.github.com/en/copilot/concepts/models/auto-model-selection)。

### 推理強度

在推理強度可設定時，請依任務來選擇。對於直接了當的工作使用較低的強度；當更深入的推理能充分改善驗收結果、足以證明額外的工作量合理時，再提高強度。簡潔的最終答案並不會顯示實際計費了多少推理。

請在長時間任務之前選好模型與推理強度。如果任務的變化大到足以證明需要不同的設定，請使用簡短的交接，而不是自動沿用整個歷程記錄。請遵循[快取工作流程](04-context-management.md#235-caching-store-and-reuse-context-within-prompts)。

## 原則與資料處理

可用性會因方案、預覽狀態、地理位置、組織原則與用戶端而異。選取器中出現的模型，並不代表已自動核准用於受規範的工作。

請對照目前的[模型與定價規則](https://docs.github.com/en/copilot/reference/copilot-billing/models-and-pricing)，檢查舊版年度方案的存取權、區域限制、合規附加費，以及延伸內容脈絡／推理強度的可用性。請勿將聊天的成本假設套用到程式碼完成或下一個編輯建議；請參閱[企業治理](12-enterprise-governance.md)。

在路由敏感工作之前，請確認適用的供應商條款、資料保留控制與組織原則。GitHub 記載了[模型主機代管](https://docs.github.com/en/copilot/reference/ai-models/model-hosting)與[原則控制](https://docs.github.com/en/copilot/concepts/enterprise/policies)。請不要假設每個模型都有相同的保留條款，尤其是預覽版。例如，Fable 模型預設會保留提示詞與輸出；符合資格的企業需要另外核准與設定，才能取得有時間限制的零保留豁免。

## 以完成的工作來評斷路由

請針對類似的任務，比較成本、經過時間、驗收檢查與重試次數。一個需要多次修復嘗試的便宜請求，可能比一次成功的較強模型執行更昂貴。

請將計費機制分開看待：模型費率、折扣、符合資格的快取與工作階段額外負擔，各自的作用方式不同。在合併節省估算之前，請參閱[比較與資料](09-comparisons-data.md)；組織層級的調整手段則請參閱[企業治理](12-enterprise-governance.md)。

---

**下一章：**[企業治理 →](12-enterprise-governance.md)
