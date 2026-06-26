# 2.4 輸出控制 —— 告訴模型「不要」說什麼

[← 返回指南](index.md)

> **為什麼這一節對成本最重要。** 在 Anthropic/Copilot UBB 的所有定價層級中，輸出 Token 的成本比輸入 Token **貴 5 倍**。儘管回應的數量通常比總輸入（包括文件上下文、歷史記錄和工具架構）小，但定價的不對稱性使得輸出控制成為本指南中投資報酬率 (ROI) 最高的行為。在 `copilot-instructions.md` 中加入一行 —— "Code only, no explanation" —— 就能在每個代碼任務上永久減少 40-70% 的輸出成本。

---

## 2.4.1 請求「僅代碼 (Code-Only)」回應

LLM 非常喜歡解釋。一個簡單的代碼生成請求可能會返回 50 行代碼，外加你並未要求的 200 個 Token 的解釋。

**在你的提示詞或 `copilot-instructions.md` 中添加：**

```text
Code only, no explanation.
```

或者在你的提示詞中：

```text
Add input validation to processOrder(). Code only.
```

**節省：代碼生成任務中 40-70% 的輸出 Token。**

**權衡：** 如果你正在學習或調試，你可能需要解釋。當你清楚自己在做什麼，只需要具體實現時，請使用 "code only"。

## 2.4.2 約束回應格式

明確告訴模型使用什麼格式：

| 指令 | 效果 | 輸出節省 |
|-------------|--------|----------------|
| "Answer in one sentence" (用一句話回答) | 限制贅字 | ~60-80% |
| "3 bullet points max" (最多 3 個列點) | 強制限制項目數量 | ~50-70% |
| "Reply as JSON" (以 JSON 回覆) | 結構化，無散文 | ~30-60% |
| "Table format" (表格格式) | 緊湊的對比方式 | ~40-60% |
| "Yes or no, then one line why" (是或否，然後一行解釋) | 極簡回應 | ~70-90% |

## 2.4.3 系統級精簡輸出

通過 `copilot-instructions.md` 將精簡輸出設置為專案預設值：

```text
Be concise. No explanations unless asked.
Code only for generation tasks.
Bullets over paragraphs.
```

這會自動套用於每次互動。你不必每次都記得輸入 "code only"。

**節省：每次互動 30-60% 的輸出 Token。**

**何時覆蓋：** 當你需要解釋時，只需明確詢問："Explain why this approach is better than X." 模型在被明確要求時仍會展開解釋，即使在精簡指令下也是如此。

---

**下一頁：** [工作流優化 →](06-workflow-optimization.md)
