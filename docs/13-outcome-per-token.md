# 每個 Token 的成果

[← 返回指南](index.md)

---

目標是讓每花費一個 token 所產出的**被接受的成果更多**：已合併的 pull request、已關閉的錯誤、通過的測試，以及更少回合的重工。

如果代理程式靠猜測、編輯錯誤的檔案，然後重頭來過，那麼簡短的提示整體上反而更花錢。一份較長的計畫若能讓第一次實作就做對，反而可能更便宜。

## 為什麼每個 Token 的成果很重要

以完成的工作來衡量 token 的花費。

Tomasz Tunguz 將此描述為轉向「每美元的智慧」：應用程式層競爭的是一張已關閉的工單、一個已出貨的 PR 或一件已解決的支援案例的成本，而不是最便宜的原始 token。[^tunguz] 這可直接對應到 Copilot 的工作。有用的指標是：

```text
每個 token 的成果 = 已驗證完成的工作 / 花費的 token 總數
```

Microsoft/Stanford 的論文〈How Do AI Agents Spend Your Money?〉回報了代理式程式開發與程式碼聊天之間的巨大差異。以下的發現描述的是所研究的任務與執行，並非適用於所有工作負載的固定倍數。[^agent-costs]

### 研究顯示了什麼

| 發現 | 為什麼重要 |
|---|---|
| 代理式程式開發消耗的 token 可能比程式碼聊天多出約 **1,000 倍** | 不要把聊天成本的直覺外推到代理程式工作階段 |
| 同一個任務在不同次執行間可能相差高達 **30 倍** | 預算要留有餘裕；單次執行不是穩定的成本估計 |
| 使用更多 token 並不一定能提高準確度 | 更多的探索不見得就是更好的成果 |
| 準確度通常在中等成本時達到高峰，之後趨於飽和 | 每個步驟都預設使用最大的模型可能會浪費金錢 |
| 在該研究中，輸入 token 主導了代理式成本 | 在優先考量輸出精簡之前，先檢查內容與計費的 token 類別 |
| 模型會低估自己的 token 使用量 | 不要相信代理程式在任務前的成本猜測；請使用預算與停止規則 |
| 不同模型的 token 效率各異，與通過率無關 | 要同時比較成果與成本，而不是只看基準測試分數 |

## 每個 Token 的成果迴圈

使用這些習慣來減少浪費的工作：

![每個 token 的成果迴圈：挑選任務型態、先規劃再寫程式碼、為模型選擇路由、鎖定內容、驗證證據，並乾淨地收尾。](../../source/docs/assets/diagrams/outcome-per-token-loop.svg)

1. 挑選正確的任務型態
2. 實作之前先規劃
3. 將正確的模型分派給正確的階段
4. 保持乾淨的內容與快取邊界
5. 宣稱完成之前先驗證

請留意模糊的提示、過大的內容、一直選著昂貴的模型、缺少驗收條件，以及在大量編輯之後才執行測試。這些模式會讓重工更難避免。

## 使用技能來減少重工

社群專案 [`obra/superpowers`](https://github.com/obra/superpowers) 將可重複的代理程式實務打包成技能。[^superpowers] 它並非 GitHub 的官方產品。本章採用其做法，來整理可減少重工的實務：

| 技能 | 作用 | 潛在效益 |
|---|---|---|
| 腦力激盪 | 在做選擇之前先探索多種做法 | 避免過早鎖定與方向錯誤的程式碼 |
| 規劃 | 將意圖轉換為檔案層級的步驟與檢查 | 減少執行期間的猜測 |
| 綠燈維持 | 在整個工作過程中讓測試保持通過 | 避免除錯未知的基準失敗 |
| 完成前驗證 | 要求在「完成」之前提出證據 | 避免虛假完成與重工 |
| 無瑕收尾 | 讓驗收條件、測試與 PR 摘要一致 | 減少審查問題與後續工作 |
| 分支收尾紀律 | 合併後乾淨地結束分支／工作階段 | 避免過期的內容滲入下一個任務 |

以下的技能是一些模式。接下來的公開函式庫則是這些模式的範例實作，並非必要的相依項目。

## 值得借鏡的技能函式庫

請借鏡能釐清需求、工具使用、測試、交接或審查的技能。這些是社群函式庫，並非 GitHub 或 Microsoft 的官方指引。它們的效益大多是定性且基於經驗，並不能證明普遍能節省 token。

| 函式庫 | 最值得借鏡的技能 | 適用時機 | 注意事項 |
|---|---|---|---|
| [`obra/superpowers`](https://github.com/obra/superpowers) | TDD、規劃、驗證、分支收尾 | 需要廣泛的代理程式 SDLC 紀律 | 社群框架 |
| [`softaworks/agent-toolkit`](https://github.com/softaworks/agent-toolkit) | 規劃協調、交接、降低熵 | 複雜功能、長時間工作階段、臃腫的指示檔案 | 個人工具組；定性的主張 |
| [`catpilotai/catpilot-ai-guardrails`](https://github.com/catpilotai/catpilot-ai-guardrails) | 安全性與工具迴圈護欄 | 代理程式會接觸機密、雲端、資料庫、Docker 或供應鏈 | 指引，而非執行階段強制措施 |
| [`vercel-labs/agent-browser`](https://github.com/vercel-labs/agent-browser/blob/main/skills/agent-browser/SKILL.md) | 快照、等待與證據紀律 | 代理程式測試瀏覽器 UI | 特定於瀏覽器；以已安裝的 CLI 內容為準 |
| [`vercel-labs/writing-guidelines`](https://github.com/vercel-labs/writing-guidelines) | 計畫即提示、輸出審查、AI 痕跡偵測 | 文件、PR 描述、規格、產生的文章 | 編輯指引，並非經過測量的 token 縮減 |
| [`mattpocock/skills`](https://github.com/mattpocock/skills) | TDD、錯誤診斷、程式碼審查、領域建模 | 工程任務需要更精準的迴圈 | 範例偏向 Claude 與 TypeScript 工作流程 |
| [`PramodDutta/qaskills`](https://github.com/PramodDutta/qaskills) | QA 與測試產生技能 | 需要 Playwright、API、BDD、安全性、無障礙或錯誤報告的深度 | 技能檔案龐大；早期專案 |

### `obra/superpowers`：基準的代理程式 SDLC 紀律

Superpowers 將「要小心」這類一般性建議，轉變為代理程式可以遵循的具名程序。[^superpowers]

當 TDD、規劃、驗證或分支收尾的指引能提供所需的下一步時，就借鏡它們。比起泛泛地要求「要小心」，應優先採用具體的證據要求。一行規則可能就足夠了。

### `softaworks/agent-toolkit`：規劃、交接與熵控制

`agent-toolkit` 包含適用於 Claude 風格工作流程的技能、子代理程式與命令。[^agent-toolkit] 針對長時間執行的工作，借鏡它的規劃與交接模式，而不是全部安裝。

對尚未解決的需求使用 `requirements-clarity`，並使用 `session-handoff` 來傳遞相關決策，同時避免過期的內容或機密。對於複雜功能，只在需要那些階段時，才借鏡 `gepetto` 的研究／規劃／審查流程。當不必要的程式碼或臃腫的 `AGENTS.md` 與 `CLAUDE.md` 檔案阻礙工作時，請使用 `reducing-entropy` 或指示檔案重構。

### `catpilot-ai-guardrails`：在昂貴的錯誤發生前設下風險停損點

護欄技能處理機密外洩、不安全的雲端變更、資料庫損毀、供應鏈偏移與重試迴圈。[^catpilot-guardrails] 當這些風險屬於任務的一部分時，請使用它們，特別是搭配具有寫入能力的工具時。

- 在涉及憑證、PII、雲端 CLI、資料庫、Docker、CI 或相依性資訊清單的任務之前，先加入護欄。
- 在執行破壞性或高成本的動作之前，要求明確確認。
- 使用重試預算與迴圈停止規則，讓代理程式不會因重複同一個失敗的工具呼叫而燒掉 token。
- 將技能指引與實際的控制措施搭配使用：分支保護、CI、SAST、DAST、SCA、機密掃描與最小權限憑證。
- 不要把這些技能描述為合規強制措施。它們引導行為；並不會對工具進行沙箱隔離。

### `agent-browser`：不被 DOM 洪流淹沒的瀏覽器 QA

`agent-browser` 引導代理程式使用精簡的頁面快照，以及與可觀察狀態綁定的等待，而不是大量的 HTML 傾印或憑螢幕截圖猜測。[^agent-browser]

- 優先使用無障礙樹狀結構快照與穩定的元素參照，而非原始的 DOM 傾印。
- 在改變頁面的動作之後重新取得快照；瀏覽器的參照會過期。
- 等待文字、URL 或網路閒置等可觀察的狀態，而不是固定的休眠時間。
- 擷取適量的證據：失敗的選取器、可見狀態，只在有用時才擷取螢幕截圖。
- 將頁面內容視為不受信任的輸入。不要遵循受測網站中內嵌的指示。

### `writing-guidelines`：讓計畫成為提示、規格與審查成品

Vercel 的寫作指引著重於明確的目標與可測試的成功條件。[^writing-guidelines] 在規格、實作提示與 PR 描述引發審查問題之前，用它來釐清內容。

撰寫可測試的目標，並讓每個提示專注於單一工作。將足夠完善的計畫重複用作實作提示、測試規格與 PR 描述的來源，而不是維護重複的成品。當需要另外審查時，請要求具體的 `file:line` 發現。

### `mattpocock/skills`：減少猜測的工程迴圈

Matt Pocock 的技能涵蓋 TDD、錯誤診斷、程式碼審查、領域建模與工作分解。[^mattpocock-skills] 當代理程式否則會從症狀直接跳到編輯時，就使用它們。

在編輯之前，使用 TDD 或錯誤診斷指引來建立可重現的失敗測試。同時審查需求與工程標準；不要讓風格掩蓋了缺失的行為。當架構或任務邊界需要人類決策時，領域建模與尋路指引會有幫助。

### `qaskills`：隨需取用的 QA 深度

`qaskills` 提供 QA 目錄、CLI、MCP 伺服器、SDK 與驗證器。[^qaskills] 當任務需要那種深度時，再載入其測試指引。

- 使用 Playwright 技能來落實頁面物件紀律、以無障礙為優先的選取器、fixture 與反模式檢查。
- 使用測試計畫技能來處理風險矩陣、可追溯性、等價分割與進入／退出條件。
- 當成果是具有嚴重性、優先順序、環境與證據的可重現問題時，使用錯誤報告技能。
- 當驗收條件應成為可執行的 Given/When/Then 情境時，使用 BDD/Cucumber 技能。
- 只有當這些檢查屬於任務的一部分時，才使用 OWASP、視覺回歸、axe-core、k6 與 API 測試技能。

### 不要安裝每一個技能

已載入的技能會增加內容。請針對目前的任務來選擇。

| 任務 | 載入 |
|---|---|
| 測試設計、QA 自動化、錯誤報告 | QA 技能 |
| 瀏覽器 UI 調查 | 瀏覽器快照／等待／證據技能 |
| 雲端、資料庫、Docker、機密、相依性 | 護欄技能 |
| 規格、文件、PR 描述 | 寫作與審查技能 |
| 長時間工作階段、複雜功能、交接 | 規劃與交接技能 |

不會改變下一次編輯、命令、測試或審查的技能，就不要載入。

### 不要過度最佳化技能堆疊

不要讓挑選技能拖延了工作。小而明確的任務可能只需要直接編輯與有針對性的驗證。

對於複雜的工作，在加入更多機制之前，先使用 `plan + grill-me` 迴圈：

1. 撰寫包含驗收條件、可能涉及的檔案、風險與驗證命令的計畫
2. 要求第二輪對計畫進行拷問：遺漏的邊界情況、錯誤的假設、更便宜的路徑與失敗模式
3. 修訂一次，然後執行

只有在專門技能會改變下一步行動時，才加入它們。如果任務不是安全性、瀏覽器、QA 或交接密集的，那麼多載入技能很可能只是內容稅。

### 腦力激盪技能

當比較不同做法能解決重要的設計抉擇時，請使用這個：

```text
在撰寫程式碼之前，列出三種實作做法。
針對每一種：可能涉及的檔案、風險、測試策略，以及 token／成本風險。
不要編輯檔案。
```

在選定一個可能需要復原的做法之前，先比較各種做法。

### 規劃技能

規劃將「建置 X」轉變為可執行的步驟。

可用時請使用官方的 VS Code [Plan 代理程式](https://code.visualstudio.com/docs/agents/planning)。可以從代理程式下拉式選單選取，或以 `/plan` 呼叫，它會產生高階計畫以及實作與驗證步驟，並透過 `chat.planAgent.defaultModel` 與 `github.copilot.chat.implementAgent.model` 支援為規劃與實作分別設定模型。[^plan-agent]

對於 GitHub.com 上的任務，Copilot 雲端代理程式支援「研究、規劃、迭代」流程：要求它研究儲存庫、迭代計畫，並且只在準備就緒時才實作已同意的計畫。[^cloud-plan]

重要的注意事項：

- VS Code Plan 代理程式的工作階段記憶體會在對話結束時清除。在關閉工作階段之前，請將重要的計畫儲存到外部。[^plan-agent]
- 在建立 PR 之前進行規劃與迭代，是 Copilot 雲端代理程式在 GitHub.com 上的功能；Azure Boards、JIRA、Linear、Slack 或 Teams 等整合僅支援直接建立 PR。[^cloud-agent]
- 雲端代理程式工作階段有 59 分鐘的硬性限制。請將大型工作拆分為較小的任務。[^cloud-agent]
- Business 與 Enterprise 使用者需要先啟用相關的管理員原則，才能使用雲端代理程式。[^cloud-agent]

### 綠燈維持技能

「綠燈維持（Greenness）」是本指南對一項簡單紀律所取的名稱：讓測試基準保持通過。

在要求代理程式修改程式碼之前：

1. 知道目前測試是否通過
2. 告知代理程式基準狀態
3. 要求它維持該基準
4. 在接受完成之前執行測試

記錄既有的失敗，以免代理程式把它們誤認為迴歸。在後續編輯掩蓋原因之前，先調查新的失敗。

### 完成前驗證技能

沒有證據就不要接受「完成」。

```text
在您回報完成之前，請執行有針對性的測試或建構，
說明確切的命令，並確認每一項驗收條件。
```

驗證有助於在審查或另一個代理程式工作階段之前，抓出未完成的工作。

### 無瑕收尾技能

「無瑕收尾（Impeccable close）」是本指南自創的名稱，並非 Copilot 的官方概念。

良好的收尾包括：

1. 已檢查驗收條件
2. 在相關之處已執行測試／建構
3. 沒有意外的範圍蔓延
4. 沒有過期的 TODO 或被註解掉的嘗試
5. PR 摘要與實際的 diff 相符
6. 只在真的需要時才指明下一步

清楚的收尾能減少在審查與後續工作中重建「改了什麼」的需要。

### 分支收尾紀律

合併之後，請關閉分支以及任何長時間執行的代理程式工作階段。不要把同一份內容用於下一個不相關的任務。

過期的工作階段會累積決策、工具輸出、檔案讀取與被放棄的做法。之後的每個提示都可能把這些歷程記錄當作輸入 token 拖著走。乾淨的收尾能讓下一個任務不必為上一個任務付費。

## 先規劃，再以低成本執行

對於有具體交接內容的複雜工作，請將規劃與執行分開：

![先規劃，再以低成本執行：以強大的推理模型規劃、儲存驗收條件、在全新且較便宜的工作階段中執行，然後驗證或修訂計畫。](../../source/docs/assets/diagrams/plan-execute-cheaply.svg)

GitHub 的[最佳化 AI 使用](https://docs.github.com/en/copilot/tutorials/optimize-ai-usage)指引也提出相同的觀點：預設使用最強大的模型可能會增加 token 用量卻沒有改善成果，而在以執行為主的任務中過度使用推理模型，可能會讓模型想太多或引入不必要的變更，進而降低品質。[^optimize-ai] 同一頁面也給出了實用的規則：以強大的推理模型規劃，然後以較便宜的模型實作。

為什麼全新的工作階段很重要：

- 規劃的對話不會在每個執行回合都被重新傳送
- 執行的內容從乾淨的狀態開始
- 計畫成為穩定、對快取友善的前綴
- 模型的路由是刻意的，而不是偶然的

這是[先規劃，再執行 §2.5.9](06-workflow-optimization.md#259-plan-first-then-execute-and-route-the-phases) 的擴充版本。

請使用[模型選擇與定價](11-models-and-pricing.md)來選擇規劃與執行的模型、設定投入程度，並查看定價與原則限制。

## 基準測試：有用，但不具決定性

基準測試有助於列出候選模型的短名單。它們無法取代在您自己的儲存庫上進行測量。

| 基準測試 | 衡量內容 | 為什麼重要 | 注意事項 |
|---|---|---|---|
| [SWE-bench Verified](https://www.swebench.com/verified.html) | 500 個經人工驗證的 Python GitHub 問題 | 經典的軟體工程代理指標 | 偏重 Python、靜態、可能有污染風險 |
| [SWE-bench Pro](https://labs.scale.com/leaderboard/swe_bench_pro_public) | 更困難的專業儲存庫任務 | 顯示企業級的難度斷崖 | 分數取決於鷹架與目前的即時排行榜 |
| [SWE-bench Live](https://swe-bench-live.github.io/) | 持續更新的問題 | 減少飽和與背誦記憶 | 動態分數會隨時間變動 |
| [DeepSWE](https://github.com/datacurve-ai/deep-swe) | 橫跨多種語言的 113 個原創長時程任務 | 檢視程式開發代理程式與每任務成本的好視角 | 任務數量少；推理層級會改變結果 |
| [Terminal-Bench](https://www.tbench.ai/) | 終端機與 shell 工作流程 | 對應到建構／測試／DevOps 的代理程式工作 | 任務數量少；終端機技能並非全部的程式開發技能 |
| [Artificial Analysis](https://artificialanalysis.ai/methodology) | 智慧、定價、延遲、提供者比較 | 良好的成本／智慧散佈圖 | 綜合分數可能不符合僅限程式開發的需求 |

### 分數快照

| 基準測試 | 模型／條件 | 分數 | 可信度 |
|---|---|---:|---|
| SWE-bench Verified | SWE-bench Pro 論文時期交叉參照的頂尖模型 | >70% | 已由 Scale AI Pro 頁面驗證 |
| SWE-bench Pro public | GPT-5，論文時期 | 23.3% | 已驗證 |
| SWE-bench Pro public | Claude Opus 4.1，論文時期 | 23.1% | 已驗證 |
| SWE-bench Pro private | GPT-5，論文時期 | 14.9% | 已驗證 |
| Terminal-Bench 2.0 | 前沿模型 | <65% | 已由基準測試摘要驗證 |
| DeepSWE | GPT-5.6 Sol `[max]` | 72.7% | 僅供參考；第三方鏡像，且取決於推理層級 |
| DeepSWE | 前三名差距 | <3.1 分 | 僅供參考；第三方鏡像，且取決於推理層級 |

引用基準測試數字時請加上標籤：

- **已驗證（Verified）**：直接從第一手排行榜或論文讀取的來源。
- **僅供參考（Directional）**：彙整者或次級來源。
- **軼事（Anecdotal）**：Reddit、Discord、社群媒體、單一工作階段的回報。

## 為什麼 Harness 很重要

基準測試分數除了取決於模型，也取決於設定：

- 代理程式鷹架
- 工具存取
- 擷取策略
- 推理投入程度
- 快取定價假設
- 任務的語言組成
- 基準測試版本
- 執行是單次嘗試還是多次嘗試

SWE-bench 本身就指出，使用不同動作格式的版本無法直接比較。DeepSWE 的項目常在模型名稱中包含推理層級，例如 `[max]`。Artificial Analysis 可能回答的是不同的問題：廣泛的每美元智慧，而非純粹的程式開發代理程式通過率。[^swebench][^deepswe][^artificial-analysis]

使用基準測試來挑選候選者。使用您自己的儲存庫任務來挑選預設值。

## 實用的決策檢查清單

在進行昂貴的工作之前，確認任務需要代理程式、預期結果明確，且已定義驗證方式。記錄測試基準、附上相關內容，並檢查模型原則。

對於複雜的任務，將設計決策記錄在計畫中。當該計畫足夠時，嘗試在全新的工作階段中以較低成本執行；完成時請回報測試證據與尚未解決的阻礙。

## 交叉參照

- [工作流程最佳化 §2.5.9](06-workflow-optimization.md#259-plan-first-then-execute-and-route-the-phases) — 先規劃習慣的精簡版
- [內容管理](04-context-management.md) — 快取與內容衛生
- [輸出控制](05-output-control.md) — 輸出 token 縮減
- [實務設定](10-practical-setup.md) — 設定與操作習慣
- [模型選擇與定價](11-models-and-pricing.md) — 模型與定價介面
- [企業治理](12-enterprise-governance.md) — 預算、模型原則與管理員推行

## 參考資料

[^agent-costs]: Longju Bai et al., ["How Do AI Agents Spend Your Money? Analyzing and Predicting Token Consumption in Agentic Coding Tasks"](https://arxiv.org/abs/2604.22750), arXiv:2604.22750.

[^tunguz]: Tomasz Tunguz, ["Intelligence Per Dollar"](https://tomtunguz.com/tokens-per-result).

[^superpowers]: [`obra/superpowers`](https://github.com/obra/superpowers)，社群代理式技能框架。

[^agent-toolkit]: [`softaworks/agent-toolkit`](https://github.com/softaworks/agent-toolkit)，社群代理程式技能工具組。

[^catpilot-guardrails]: [`catpilotai/catpilot-ai-guardrails`](https://github.com/catpilotai/catpilot-ai-guardrails)，社群安全性護欄技能。

[^agent-browser]: [`vercel-labs/agent-browser` 代理程式瀏覽器技能](https://github.com/vercel-labs/agent-browser/blob/main/skills/agent-browser/SKILL.md)。

[^writing-guidelines]: [`vercel-labs/writing-guidelines`](https://github.com/vercel-labs/writing-guidelines)，實務工作者的寫作與審查指引。

[^mattpocock-skills]: [`mattpocock/skills`](https://github.com/mattpocock/skills)，社群工程技能集合。

[^qaskills]: [`PramodDutta/qaskills`](https://github.com/PramodDutta/qaskills)，QA 技能目錄與工具。

[^plan-agent]: VS Code Docs，["Planning with Copilot"](https://code.visualstudio.com/docs/agents/planning)。

[^cloud-plan]: GitHub Docs，["Use Copilot agents: Research, plan, iterate"](https://docs.github.com/en/copilot/how-tos/copilot-on-github/use-copilot-agents/research-plan-iterate)。

[^cloud-agent]: GitHub Docs，["About Copilot cloud agent"](https://docs.github.com/en/copilot/concepts/agents/cloud-agent/about-cloud-agent)。

[^optimize-ai]: GitHub Docs，["Optimize AI Usage"](https://docs.github.com/en/copilot/tutorials/optimize-ai-usage)。

[^swebench]: [SWE-bench Verified](https://www.swebench.com/verified.html)。

[^deepswe]: [DeepSWE](https://github.com/datacurve-ai/deep-swe)。

[^artificial-analysis]: [Artificial Analysis methodology](https://artificialanalysis.ai/methodology)。

---

**下一篇：** [回到首頁 →](index.md)
