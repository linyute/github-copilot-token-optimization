# 模型選擇與定價

[← 返回指南](index.md)

---

本頁面的存在是因為在關於 Token 優化的討論中，關於模型的建議往往被混為一談。目前共有**三種不同的定價觀點**在發揮作用：

1. **GitHub Copilot 官方文檔** —— 根據計劃確定的模型可用性、Auto 的行為，以及 GitHub 發布的任何相對定價信號
2. **本存儲庫的 UBB 框架** —— 如何思考 Business 和 Enterprise 計劃在 2026 年 6 月 1 日向用量計費 (UBB) 的轉變：按企業、組織、成本中心和使用者進行的 AI 額度計量和預算
3. **廠商 API 定價** —— 來自 Anthropic 等提供商的每 Token 定價，其中輸入和輸出的計費方式不同

**不要**將這些視為相同的單元。請根據每個觀點實際能回答的問題來使用它們。

## 你需要的官方 GitHub 文檔

- [關於 Copilot 自動模型選擇](https://docs.github.com/en/copilot/concepts/auto-model-selection)
- [GitHub Copilot 中的請求](https://docs.github.com/en/copilot/concepts/billing/copilot-requests)
- [GitHub Copilot 計劃](https://docs.github.com/en/copilot/get-started/plans)

這三個頁面幾乎涵蓋了從業者今天需要的所有內容：

- 存在哪些模型
- 哪些計劃包含這些模型
- 在付費計劃中，哪些模型是免費的，哪些是進階 (Premium) 的
- 哪些模型目前被記錄為相對於其他模型更便宜或更昂貴
- Auto 實際上在做什麼

有一個時間點對本頁面的其餘部分至關重要：Copilot Business 和 Copilot Enterprise 的用量計費將於 **2026 年 6 月 1 日**開始。在該日期之後，AI 額度使用情況將成為 Business 和 Enterprise 治理的主要視角。下文中的任何進階請求措辭都應僅被視為過渡背景。

## 關於 Auto 最重要的澄清

本存儲庫已經推薦將 **Auto** 作為預設設置，這在方向上仍然是正確的。但 GitHub 官方文檔比本存儲庫之前的說明更具體。

### Auto 實際上在做什麼

根據 [關於 Copilot 自動模型選擇](https://docs.github.com/en/copilot/concepts/auto-model-selection)：

- Auto 會根據你的計劃和組織政策，從**受支持的自動選擇池 (supported Auto pool)** 中進行選擇
- 自動選擇基於**即時系統健康狀況和模型性能**
- 在付費計劃中，Copilot Chat 中的自動選擇可獲得 **10% 的折扣**
- Auto 會留在受支持的預設池中，而不會自動升級到每個進階模型

最後一點非常重要。

> **實際後果：** Auto 並非「在需要時挑選任何模型，包括昂貴的模型」。它是一個跨受支持自動集合的低摩擦預設值。如果你想要更高成本的進階模型，你應該**手動固定**。

## 每個定價觀點回答了什麼問題

| 定價觀點 | 單位 | 最佳用途 | 來源 |
|---|---|---|---|
| GitHub 公開的 Copilot 計費 | 模型倍率以及（6 月 1 日後）AI 額度使用量 | 「GitHub 目前對於 Copilot 內部的此模型成本披露了什麼？」 | [GitHub Copilot 中的請求](https://docs.github.com/en/copilot/concepts/billing/copilot-requests) |
| 存儲庫 UBB 框架 | AI 額度和未來的用量預算 | 「在 Business 和 Enterprise 於 2026 年 6 月 1 日轉向用量計費後，我們該如何思考支出？」 | [計費產品的預算設置](https://docs.github.com/en/billing/how-tos/budgets/setting-up-budgets-to-control-spending-on-metered-products) |
| 廠商 API 定價 | 每百萬 Token (MTok) 的輸入/輸出價格 | 「原始 Token 的價值有何不同，特別是輸入 vs 輸出？」 | [Anthropic 定價](https://platform.claude.com/docs/en/about-claude/pricing) |

## 輸入 vs 輸出定價的適用場景

官方公開的 GitHub Copilot 文檔並**沒有**發布按模型劃分的 Copilot 輸入 Token 價格 vs 輸出 Token 價格的公開表格。Anthropic 的 API 定價仍然提供了一個清晰的不對稱範例，這對於建立直覺仍然很有用。只是不要將其描述為當今 Copilot 的計費數學。

| 模型 | 輸入 / MTok | 輸出 / MTok |
|---|---:|---:|
| Claude Haiku 4.5 | $1 | $5 |
| Claude Sonnet 4.6 | $3 | $15 |
| Claude Opus 4.6 | $5 | $25 |

來源：[Anthropic 定價](https://platform.claude.com/docs/en/about-claude/pricing)

這對於建立直覺很有用：

- 輸出 Token 明顯比輸入 Token 昂貴
- 冗長的回應會比人們預期的更快主導成本
- 輸出控制仍然是本存儲庫中投資報酬率 (ROI) 最高的習慣之一

## 推理努力 (Reasoning Effort) 是一個獨立的成本撥盤

模型選擇並非唯一的撥盤。在受支持的具備推理能力的模型上，**思考努力 (thinking effort)** 或 **推理努力 (reasoning effort)** 會改變模型在回答前的工作量。

這已在 [實踐設置](10-practical-setup.md#推理努力-reasoning-effort-另一個撥盤) 中有更詳細的介紹，但它也屬於本頁面，因為它直接影響支出。

### 它改變了什麼

- 模型在回應前花費多少 Token 進行思考
- 它傾向於生成多少工具調用和前言
- 延遲，特別是在較難的任務上

### 實踐建議

| 情況 | 推薦努力程度 | 為什麼 |
|---|---|---|
| 高通量、簡單的聊天或分類 | `low` | 最便宜。當可接受微小的品質損失時適用 |
| 在受支持的推理模型上進行典型的編碼、重構、工具密集型工作 | `medium` | 成本與品質的最佳平衡；Anthropic 推薦將此作為 Sonnet 4.6 的預設值 |
| 艱難的架構、安全審查、新型分解 | `high` 或 `max` | 僅在任務明確證明其合理性時才增加支出 |

### 重要細微差別

> **僅在支持推理努力的模型上使用。** GPT-4.1 和 GPT-4o 等非推理模型不具備此控制項。

因此，完整的決策堆疊為：

1. 選擇正確的模型層級
2. 如果模型支持，選擇能完成工作的最低推理努力程度

當比較一個在 `medium` 努力下便宜且具備推理能力的模型，與一個在 `high` 努力下的高成本進階模型時，這尤其相關。在實踐中，努力程度的調優可以作為跳向更昂貴模型的廉價替代方案。

## 那麼你到底該怎麼做？

### 預設立場

1. **日常工作預設使用 Auto**
2. **對於瑣碎任務，當你知道不需要更強大的模型時，使用包含在內或較低成本的模型**
3. **當任務明確證明其合理性時，手動固定進階模型**
4. **在啟用前審核組織的模型政策**，以便有意識地擴展進階訪問權限，而非隨意漂移

### 良好的預設啟發法

| 任務 | 預設選擇 | 為什麼 |
|---|---|---|
| 語法查詢、快速解釋、微小編輯 | 包含在內之模型或 Auto | 最便宜的路徑，品質足夠 |
| 典型的實現、錯誤修復、重構 | Auto 或標準模型 | 最佳的品質-成本權衡 |
| 架構、威脅建模、新型分解 | 手動固定進階模型 | Auto 不會自動升級到進階車道 |

### 反模式

- 在整個會話中一直固定著昂貴的進階模型
- 假設當任務變難時，Auto 會自動升級到 Opus
- 將廠商 API 價格和 Copilot 定價信號視為相同的指標
- 在未檢查計劃是否包含該模型的情況下推薦該模型
- 在檢查誰實際需要之前，為整個組織開啟每個進階模型

## 組織推廣規則：啟用前審核

對於團隊來說，模型選擇既是一個治理問題，也是一個提示詞問題。

請使用 [配置 Copilot 中的 AI 模型訪問權限](https://docs.github.com/en/copilot/using-github-copilot/ai-models/configuring-access-to-ai-models-in-copilot) 來控制可用的 AI 模型。GitHub 文檔說明組織所有者和企業所有者可以為成員啟用或禁用 AI 模型的訪問權限。

實踐規則：

1. 先啟用較便宜的模型
2. 根據工作流、團隊和預期投資報酬率 (ROI) 審核進階模型的需求
3. 狹義地啟用進階訪問權限
4. 在擴大訪問權限之前觀察使用報告和 AI 額度消耗情況

當你需要直接的每使用者上限時，請使用使用者級別的 AI 額度預算。請記住，代碼補全和下一步編輯建議不按 AI 額度計費。更多內容請參閱 [企業治理](12-enterprise-governance.md)。

## 本存儲庫中的交叉引用

- [實踐設置](10-practical-setup.md) —— 日常設置和路由建議
- [實踐設置 § 推理努力](10-practical-setup.md#推理努力-reasoning-effort-另一個撥盤) —— 推理等級的深入探討
- [工作流優化](06-workflow-optimization.md) —— 為什麼 Auto 仍然是最佳預設值
- [企業治理](12-enterprise-governance.md) —— 預算、使用者級別上限、模型政策、指令範圍
- [首頁](index.md#快速術語) —— 快速術語和支持連結

---

**下一頁：** [企業治理 →](12-enterprise-governance.md)
