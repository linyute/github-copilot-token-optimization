# 2.4 輸出控制

[← 返回指南](index.md)

要求你需要的回應。從一般的簡潔指示開始；當說明、警告與驗證證據有助於你採取行動時，請予以保留。

## 2.4.1 要求僅回應程式碼

當你已經理解做法、要進行程式碼產生時：

```text
Code only, no explanation.
```

或是限制單一任務：

```text
Add input validation to processOrder(). Code only.
```

請不要用它來壓下錯誤、缺漏的需求或失敗的測試。在學習與偵錯時，請要求提供說明。

## 2.4.2 限制回應格式

| 需求 | 指示 |
|---|---|
| 快速回答 | "Answer in one sentence" |
| 簡短摘要 | "3 bullet points max" |
| 機器可讀的擷取 | "Reply as JSON" |
| 比較 | "Table format" |
| 決策 | "Yes or no, then one line why" |

這些是格式上的選擇，並非固定的節省率。請判斷結果是否包含足夠可用的資訊。

## 2.4.3 系統層級的精簡輸出

專案預設值請使用[簡潔入門範本](10-practical-setup.md#step-1-set-a-concise-response-default)。當任務需要更多細節時，再覆寫它。

## Caveman：選用的輸出風格

Caveman 可以縮短助理的敘述。相較於一般的簡潔指示，額外的節省可能很小。在將它視為成本最佳化手段之前，請先衡量整體任務成本與正確性。

這個**回應風格技能**不會壓縮輸入上下文或思考 token；它的規則還會增加輸入。上游的記憶檔案壓縮器與代理程式（proxy）是另外獨立的機制。[^honest]

### 已衡量的內容

證據查核日期為 2026 年 10 月 27 日。上游連結固定在特定的原始碼修訂版本，並不代表對後續版本的承諾。

| 研究 | 比較與結果 | 限制 |
|---|---|---|
| 上游回應風格評估 | 標準 Caveman：**相較 `Answer concisely.`，輸出的中位數僅再減少 3%** | 十個提示詞、單次執行、`claude-opus-5-5`、近似的 `o200k_base` 計數。測試的技能為 3.1.0；3.2.0 規則變更後未重新執行。沒有經過審查的忠實度快照。[^honest][^evals] |
| JetBrains，2026 年 7 月 | 強制使用 Caveman 對比原生 Claude Code：在 82 個成對任務上**輸出 token 少 8.5%** | SkillsBench、`claude-sonnet-5`、低投入程度。在統計上未偵測到品質差異（`p = 0.82`），但這不構成等效的證明。[^jetbrains] |
| 同一份 JetBrains 完整執行，計費成本 | 技能組的原始總計**高出 11.6%** | 一個長上下文的離群值扭轉了整體成本的結果。輸出節省並不保證支出較低。[^jetbrains] |

不同的對照組、工作負載與技能版本回答的是不同的問題。請不要將這些結果取平均，也不要把任何一項變成通用的節省率。對於舊有「預設冗長回覆」的標題說法，上游並未發布經過審查的彙總結果。[^honest]

### 比較預設、簡潔與 Caveman

改編自上游的三組設計。[^evals]

| 組別 | 額外的回應風格指示 |
|---|---|
| A：預設 | 無 |
| B：簡潔 | `Answer concisely.` |
| C：Caveman | 相同的簡潔指示，加上固定版本的 Caveman 技能 |

使用相同的任務、存放庫修訂版本、模型、投入程度、工具、權限與預算。請隔離風格指示，避免預先載入的 Caveman 規則汙染對照組。任務語言保持不變。

每個任務在每一組中都要重複執行、交錯執行順序，並回報執行次數與離散程度。將純文字問答與大量使用工具的實作分開。保留原始輸出與提供者的用量記錄。

每次執行都要記錄：

- 驗收檢查、測試結果，以及遺漏的事實或條件
- 輸入、輸出、快取讀取／快取寫入，以及（若有揭露的）推理 token
- 計費成本、經過時間、重試次數與工具呼叫次數

將 C 與 B 比較，以隔離 Caveman 所增加的價值；將 B 與 A 比較，以衡量一般性的簡潔效果。請回報每個任務的成對差異與總計費成本，包含失敗的執行與離群值。離線 tokenizer 的計數只能估計文字長度，無法反映提供者的收費。

在結果仍然有用、且衡量到的取捨符合工作負載時，才使用 Caveman。若簡潔的一般指示做得一樣好，請優先採用。本指南並未自行執行付費的三組基準測試。

## 來源

[^honest]: Caveman，[Honest Numbers](https://github.com/JuliusBrussee/caveman/blob/9b68f7ed12b05e7efaff18e65c5a3fc2849fd0a4/docs/HONEST-NUMBERS.md)。
[^evals]: Caveman，[評估設計與快照限制](https://github.com/JuliusBrussee/caveman/blob/9b68f7ed12b05e7efaff18e65c5a3fc2849fd0a4/evals/README.md)。
[^jetbrains]: Denis Shiryaev，JetBrains，[Does Speaking to Agents Like Cavemen Really Save 65% of Tokens? We Test](https://blog.jetbrains.com/ai/2026/07/speak-to-ai-agents-like-cavemen-tosave-tokens/)。

---

**下一章：** [工作流程最佳化 →](06-workflow-optimization.md)
