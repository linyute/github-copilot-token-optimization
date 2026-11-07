# 2.3 上下文管理：控制要傳送的內容

[← 返回指南](index.md)

---

## 2.3.1 系統指示壓縮

Copilot 在每次互動時都會載入 `.github/copilot-instructions.md`。縮短它可以減少重複的輸入。

> `.github/copilot-instructions.md` 是 Copilot 的存放庫層級指示檔案。`AGENTS.md` 是跨工具的慣例，Copilot 也會讀取。兩者可以並存：共用規則放在一個檔案，Copilot 專屬規則放在另一個。請避免重複相同的指引，因為兩個檔案載入時都會計入上下文。要保留哪些內容，請參閱[第 2.6 部分](07-agents-md-problem.md)。

一份約 500 個字、相當於約 700 個 token 的指示檔案，會在你提出問題之前，就為每次互動增加這些輸入。

在縮短個別句子之前，請先移除重複與不相關的規則。回應偏好請使用[入門範本](10-practical-setup.md#step-1-set-a-concise-response-default)，專案指引則請使用[研究篩選準則](07-agents-md-problem.md#265-what-to-actually-put-in-context-files)。

## 2.3.2 記憶檔案壓縮

專案也會累積 `CLAUDE.md`、專案筆記、程式碼撰寫指南與 `.cursorrules`。當這些檔案進入上下文時，就會增加輸入 token。在縮短個別檔案之前，請先移除重複的指引。

刪除過時的筆記與重複的規則。保留命令、路徑、否定、條件與步驟順序。偶爾才用到的檢查清單請放在提示詞之外。只有在檔案原本就會被載入時，把它縮小才有幫助。

## 2.3.3 策略性的檔案組織

Copilot 可能納入開啟的檔案、匯入項目與鄰近的程式碼。請讓工作區的上下文專注於任務。

**官方的排除機制是 Content Exclusion（內容排除）**，由存放庫／組織／企業的管理員透過 GitHub 設定或 REST API 進行設定。它僅適用於 **Copilot Business 與 Enterprise**，而且**不**適用於 Copilot CLI、Copilot 雲端代理程式，或 IDE 中 Copilot Chat 的 Agent 模式。請將它視為隱私／政策控制（「不要讀取這個檔案」），而不是每位開發人員用來節省 token 的旋鈕。請參閱 [GitHub Docs — Excluding content from GitHub Copilot](https://docs.github.com/en/copilot/how-tos/configure-content-exclusion/exclude-content-from-copilot)。

若要減少自動納入的上下文：

- **關閉你沒有在處理的檔案。** 開啟的索引標籤會成為自動納入上下文的來源。請關閉與任務無關的大型檔案。
- **參照大型檔案中相關的部分。** 不要僅為了減少上下文而重組程式碼。
- **對建構輸出、vendor 目錄與產生的檔案使用 `.gitignore`。** 請確認你的主機實際納入了什麼；Git 的忽略規則並不是隱私邊界。
- **使用以 `applyTo:` 限定範圍的指示檔案**（請見下方 § 2.3.4）來提供特定路徑的指引。

當下列內容處於開啟或被參照的狀態時，請檢查這些大型上下文的來源：

- 大型 README 檔案
- 產生的檔案（建構輸出、打包後的 CSS、壓縮過的 JS）
- Vendor 目錄
- 資料檔案（CSV、JSON fixture）
- 封存的文件
- 豐富的文件格式（`.docx`、`.pdf`、`.pptx`、`.xlsx`、HTML 匯出檔、掃描影像、音訊／影片逐字稿）

每個進入上下文的檔案都會耗用 token。請有意識地決定要開啟哪些檔案，以及用 `#file` 參照哪些檔案。

### 先將非文字輸入正規化為 Markdown

當任務需要的是文字而非版面配置時，請在傳送給模型之前，先將豐富格式的文件轉換成乾淨的 Markdown。請確認擷取過程保留了任務所需的資訊。

![格式稅管線：豐富格式會產生版面雜訊，MarkItDown 將它們正規化為乾淨的 Markdown，而豐富的交付格式只在最後才產生。](../../source/docs/assets/diagrams/format-tax-pipeline.svg)

Marc Bara 在 [Your .docx Is Wasting 33% of Your AI Budget](https://medium.com/@marc.bara.iniesta/your-docx-is-wasting-33-of-your-ai-budget-86a3d229d042) 中將此稱為**格式稅（format tax）**。豐富格式帶有字型資料、XML、頁面位置、內嵌物件與標記，這些內容模型很少需要。在該文章的範例中，一份 10 頁的 PDF 報告約使用 12,400 個 token；乾淨的 Markdown 約使用 8,350 個，在資訊相同的情況下減少 33%。HTML 匯出檔還可能在內容周圍加入更多版面標記。

請將 Markdown 作為文字任務的**工作格式**。當版面配置、影像或文件結構很重要時，請保留原始檔案。當收件者需要時，再於最後產生 `.docx` 或 `.pdf`。

[Microsoft MarkItDown](https://github.com/microsoft/markitdown) 是用於 LLM 與文字分析管線的 Python 轉換器。它會保留標題、清單、表格、連結與擷取出的 Metadata，而不重現視覺版面配置。轉換器涵蓋 PDF、Word、PowerPoint、Excel、支援 EXIF/OCR 的影像、支援轉錄的音訊、HTML、CSV/JSON/XML、ZIP 內容、YouTube URL、EPUB 等。

快速做法：

```bash
pip install 'markitdown[all]'
markitdown report.docx -o report.md
markitdown deck.pptx -o deck.md
markitdown source.pdf > source.md
```

當你能控制工作流程、並希望減少相依套件時，請使用範圍較窄的額外套件：

```bash
pip install 'markitdown[pdf,docx,pptx,xlsx]'
```

安全性注意事項：MarkItDown 會以目前處理序的權限讀取檔案、串流與 URL。對於不受信任的輸入，請先驗證路徑與 URL，並優先採用符合工作流程的最窄範圍轉換 API。

<span id="scoping-guidance"></span>

## 2.3.4 有意識地限定上下文範圍：條件式優於常駐

請將特定路徑的規則排除在常駐上下文之外。React 元件的任務不需要資料庫移轉的指南。請以條件方式載入這些規則。

![上下文載入邊界：讓常駐的核心保持極小，將特定路徑的指引移到限定範圍的指示，並按需載入技能。](../../source/docs/assets/diagrams/context-loading-boundaries.svg)

### 在自訂指示中使用 `applyTo:` 路徑

位於 `.github/instructions/*.instructions.md` 的自訂指示檔案接受 `applyTo` frontmatter 欄位，用來將檔案限定在相符的路徑。只有當對話涉及符合該 glob 的檔案時，Copilot 才會載入它。

```markdown
---
applyTo: "src/api/**/*.ts"
---
API conventions:
- Routes in src/api/routes/. Handlers thin, logic in services/.
- Validate with zod. Errors via Result<T,E>, never throw.
- All endpoints return { data, error } envelope.
```

限定範圍可以讓這份 API 指引不出現在不相關的任務中。減少的幅度取決於主機載入指示的行為。

請將大型的 `copilot-instructions.md` 拆成一個小型的常駐核心，以及限定範圍的檔案，例如 `api.instructions.md`、`ui.instructions.md`、`tests.instructions.md` 與 `migrations.instructions.md`。為每個檔案提供各自的 `applyTo` glob，這樣只會載入相關的指引。

### 善用按需指引與 MCP 工具探索

特定工作流程的指引應遵循與 MCP 工具相同的規則：讓它不出現在常駐提示詞中，只在任務需要時才引入。如果 PR 審查檢查清單、發行範本或偵錯手冊你只是偶爾使用，就不應該放在 `copilot-instructions.md` 中。

請將偶爾使用的指引放在提示詞檔案、斜線命令片段，或按需載入的已儲存檢查清單中。

### 準則

- **常駐：** 真正適用於每次互動的少數規則，例如整個專案通用的慣例。
- **條件式（`applyTo`）：** 任何特定路徑的內容（依語言、依模組、依層級）。
- **按需筆記／MCP：** 任何偶爾才會叫用的特定工作流程內容。

減少的幅度取決於原本會載入哪些檔案。請保留每個任務所需的規則；並比較限定範圍前後實際載入的上下文。

> **要衡量載入的上下文，而不是磁碟上的檔案大小。** 技能在其完整指示載入之前，就可能貢獻探索用的 Metadata。關於已載入／延後載入的區別，請參閱 [MCP 與工具成本 §2.7](08-mcp-tool-costs.md)。

## 2.3.5 快取：在提示詞之間儲存並重複使用上下文

Copilot 可以快取穩定的上下文，包括系統提示詞、指示檔案與最近載入的檔案，而不是在每一輪都以完整的輸入費率處理。

在長時間的工作階段中，快取輸入可以帶來顯著的差異。折扣可高達約 90%，視提供者、模型、Copilot 介面與計費規則而定。

保留可重複使用上下文的三種方式：

![快取穩定性：穩定的執行緒會重複使用相同的模型、MCP 集合與代理程式設定檔；快取失效的執行緒則在工作階段中途切換控制項，應改用全新的交接。](../../source/docs/assets/diagrams/cache-stability.svg)

**1. 避免不必要的指示變更。** 前置詞快取取決於請求內容是否相符。請讓可重複使用的指引保持穩定；不要僅為了維持快取而保留不相關的檔案。

**2. 重複使用共用的上下文。** 只載入一次簡短的結構描述筆記或已儲存的片段，之後在後續提示詞中參照它，而不是再次貼上。

**3. 讓請求前置詞保持穩定。** 前置詞包含系統提示詞、工具／MCP 定義與指示檔案（即 `System/Tools` 基準；請參閱 [MCP 與工具成本 §2.7](08-mcp-tool-costs.md)）。下列變更可能使快取的上下文失效：

| 工作階段中途的變更 | 對快取與上下文的影響 |
|--------------------|---------------------------|
| **切換模型或推理投入程度** | 快取通常是以模型為單位，變更模型設定可能會捨棄已快取的前置詞。沿用的歷史記錄可能需要在新設定下重新處理。請參閱[模型與投入程度的選擇](11-models-and-pricing.md#good-default-heuristics)。 |
| **切換自訂代理程式** | 可能改變提示詞、工具與指示。已快取的前置詞可能不再相符；先前的歷史記錄也可能變得不相關。 |
| **切換 MCP 伺服器、工具或已載入的技能** | 可能改變載入的定義與指示，影響前置詞的重複使用以及後續的輸入大小。 |
| **大量更動上下文** | 附加或移除大型檔案可能影響前置詞的重複使用。請選取相關的上下文，而不是僅為了快取而保留的檔案。 |

快取的前置詞可以同時減少處理時間與輸入成本。

```text
Cache-stable long thread

Turn 1     Turn 2     Turn 3     Turn 4
|          |          |          |
v          v          v          v
[same model + same reasoning effort + same skills/MCP set + same agent/profile]
[stable prompt prefix reused from cache          ]  -> lower effective input cost
[only newest user request changes               ]

Cache-busted long thread

Turn 1     Turn 2        Turn 3
|          |             |
v          v             v
[model A + MCP set A + agent A]
[cached prefix builds         ]
                  |
                  v
          switch model / reasoning effort / skills / MCP / agent
                  |
                  v
[large prefix invalidated or repriced] -> cache discount lost; start fresh instead
```

### 必須切換時的安全交接模式

在進行長時間的工作之前，先選好模型、投入程度、代理程式與工具。當需要不同的設定時，請帶著相關的上下文重新開始，而不是沿用整個執行緒。計費行為會因介面與方案而異；這並不是保證的重新計價算法。

1. 摘要下一階段所需的決策、限制條件與待辦任務。
2. 以新的模型／代理程式／MCP 設定開啟新的聊天。
3. 只貼上摘要與所需的檔案，不要貼上整份舊的對話記錄。

這樣可以讓舊的、不相關的上下文不進入新的工作階段。

## 2.3.6 開啟全新的對話

對話歷史記錄會持續累積，直到主機將它壓縮或捨棄。長的執行緒可能把數萬個 token 帶入後續的請求。

**何時該重新開始：**

- 主題改變：新的問題、新的檔案、新的關注點
- 得到你需要的答案之後
- 當回應開始變慢時（上下文視窗快要填滿）
- 當你發現品質下降時（模型被舊的上下文搞混）

**如何維持連貫性：** 在新的提示詞中摘要關鍵決策。「延續認證重構的工作：我們選擇了 JWT 而非 session。現在實作 refresh token。」

## 2.3.7 持久性圖譜：取代每個工作階段的檔案讀取

如果重複的結構性檔案讀取占據了上下文的大部分，請考慮使用持久性的程式碼庫地圖。[Graphify 的設定、輸出與限制](08-mcp-tool-costs.md#2711-persistent-codebase-maps-graphify)與其他工具放在一起說明。

---

**下一章：** [輸出控制 →](05-output-control.md)
