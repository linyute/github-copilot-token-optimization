# 2.6 始終開啟的上下文問題 —— 為什麼越少越好

[← 返回指南](index.md)

---

> **不同的約定，共享的成本特性。** `AGENTS.md`（跨工具約定）、`.github/copilot-instructions.md`（Copilot 原生）和 `CLAUDE.md`（Claude Code 約定）是**具有不同歷史擁有者、不同範圍的不同文件** —— 它們不能互換，且一個存儲庫可以合法地使用多個文件。它們的共同點是本節所探討的成本動態：每個文件都充當**始終開啟的上下文**，被讀取它的工具加載到每次互動中。以下研究是針對 `AGENTS.md` 進行的，但其底層機制（冗餘的上下文會分散注意力並推高 Token 成本）適用於任何始終開啟的指令文件。請將下文的修剪建議應用於你存儲庫所使用的任何文件，同樣地，[第 2.3 部分](04-context-management.md) 中的壓縮建議也適用。

## 2.6.1 研究表明：上下文文件通常有害

AI 輔助開發中存在一個持久的假設：更多的上下文 = 更好的結果。編寫詳細的 `AGENTS.md`、`copilot-instructions.md` 或 `CLAUDE.md` —— 或者運行 `/init` 來生成一個 —— 代理的表現就會更好。

**蘇黎世聯邦理工學院的研究 (Gloaguen et al., AGENTBENCH, 2026 年 2 月) 大規模測試了這一點：涵蓋 12 個存儲庫中的 138 個任務，涉及 4 種不同的編碼代理。** 結果如下：

| 發現 | 數據 |
|---------|------|
| LLM 生成的上下文文件會損害性能 | 在 5/8 的實驗設置中出現下降 |
| 平均正確率變化 | 在 AGENTBENCH 上為 **−2%** |
| 使用 LLM 生成的上下文導致成本增加 | 額外消耗了 **20-23%** 的 Token |
| GPT-5.2 推理開銷 | 帶有上下文文件時，推理 Token 增加 **22%** |

這並非「中性」。LLM 生成的上下文文件在增加成本的同時，實際上讓代理變得更糟。

## 2.6.2 人工編寫的文件：效果微乎其微

人工編寫的上下文文件表現稍好 —— 但並不一致：

| 模型 | 人工編寫上下文文件的效果 |
|-------|--------------------------------------|
| 各模型平均 | 約 4% 的改善（不一致） |
| Claude Code | 使用人工編寫上下文後**表現變差** |
| 文件發現率 | 無論是否有上下文文件，結果完全相同 |

最後一點很重要：無論如何，代理都能找到相同的文件。上下文文件並不能幫助它們進行導覽 —— 它們已經知道如何使用 `ls` 和 `grep`。

## 2.6.3 為什麼更多的上下文反而有害

四個機制解釋了這些發現：

**1. 冗餘稅 (Redundancy tax)。** LLM 生成的上下文文件包含代理通過閱讀代碼就能發現的信息。代理閱讀你的 `package.json`、你的導入、你的目錄結構 —— 然後又閱讀了說著同樣事情的上下文文件。雙倍的 Token，零新增資訊。

**2. 注意力稅 (Attention tax)。** LLM 具有 U 型注意力：它們對上下文的開頭和結尾關注度很高，但中間部分會被遺忘 (Liu et al., "Lost in the Middle," 2023)。一個 200 行的 `AGENTS.md` 意味著第 50-150 行的指南會被忽視。你最重要的規則反而最容易被跳過。

**3. 錨定陷阱 (Anchoring trap)。** 代理過於忠實地執行上下文文件中的指令 —— 甚至是過時的指令。如果你的 `AGENTS.md` 提到了一個特定工具，代理使用該工具的次數會比不使用該文件時**高出 1.6 倍**，即使另一個工具更適合該任務。

**4. 信噪比。** 上下文的每個 Token 都在爭奪模型的注意力。低價值上下文（例如在存在 `tsconfig.json` 的情況下說明「此專案使用 TypeScript」之類的常規事實）會稀釋高價值上下文（例如專案特定的警示：「不要重構 auth 模組 —— 它正在進行安全審計」）。

## 2.6.4 反論：效率 vs. 正確性

Lulla et al. (2026 年 1 月) 發現人工編寫的 `AGENTS.md` **減少了 29% 的運行時間**和 **17% 的輸出 Token**。這聽起來很棒 —— 直到你閱讀了其方法論。

他們衡量的是**效率**，而非**正確性**。上下文文件幫助代理更快地導覽。但它們並不能幫助代理得出正確的答案。

這種區別很重要：一個更快的錯誤答案仍然是錯誤的。而且，快速導覽節省的 17% Token，被處理上下文文件本身增加的 20-23% 成本抵消了。

## 2.6.5 上下文文件中到底該放什麼

Addy Osmani 的過濾標準 (Google, 2026)：

> **「代理能通過閱讀代碼自己發現這件事嗎？如果可以，就刪掉它。」**

通過此過濾標準後留下的內容：

| 保留 | 刪除 |
|------|--------|
| "Use `uv` instead of `pip`" | "This is a Python project" |
| "Run tests with `--no-cache`" | "Tests are in the `tests/` directory" |
| "Don't refactor the auth module" | "We use JWT for authentication" |
| "Deploy requires VPN connection" | "Main branch is protected" |
| "DB migrations must run in order" | "We use PostgreSQL" |

**模式：** 僅保留 **地雷 (landmines)** —— 那些看起來正常但會導致錯誤的事物。刪除所有可被發現的內容。

## 2.6.6 理想狀態：像對待 Bug 追蹤器一樣對待它

```text
從幾乎空的文件開始。
代理在某處出錯 → 添加一行。
根本原因被修復 → 刪除該行。
```

你的上下文文件應該像 Bug 追蹤器一樣增長和縮小，而不是像維基 (Wiki) 那樣累積。如果它只是變得越來越長，那你做錯了。

## 2.6.7 本存儲庫的方法：6 行，約 50 個 Token

本專案的 `.github/copilot-instructions.md`：

```text
Terse like caveman. Technical substance exact. Only fluff die.
Drop: articles, filler (just/really/basically), pleasantries, hedging.
Fragments OK. Short synonyms. Code unchanged.
Pattern: [thing] [action] [reason]. [next step].
ACTIVE EVERY RESPONSE. No revert after many turns. No filler drift.
Code/commits/PRs: normal. Off: "stop caveman" / "normal mode".
```

**6 行。約 50 個 Token。** 每次互動都會載入。

對比典型的 `/init` 輸出：**200+ 行，約 1,500 個 Token。** 那意味著每次互動都會浪費 1,450 個 Token —— 而且根據研究，它很可能會讓代理的表現更差。

**會話中的計算：**

| 上下文文件風格 | 每次載入的 Token | 50 次互動 | 代理 (20 個步驟) |
|-------------------|:--------------:|:---------------:|:----------------:|
| `/init` 生成 (200 行) | ~1,500 | 75,000 | 30,000 |
| 典型手寫 (50 行) | ~400 | 20,000 | 8,000 |
| 原始人式壓縮 (6 行) | ~50 | 2,500 | 1,000 |
| **節省 (原始人式 vs /init)** | **1,450** | **72,500** | **29,000** |

---

**下一頁：** [MCP 和工具成本 →](08-mcp-tool-costs.md)
