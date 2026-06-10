# 2.1 提示詞壓縮 —— 用更少的 Token 表達相同的意思

[← 返回指南](index.md)

---

## 2.1.1 原始人式語法 (Caveman-Speak)

單一最有效的 Token 優化技術。丟掉那些增加 Token 卻不增加資訊的語言支架。

**要丟掉什麼：**

- 冠詞：a, an, the（英文中常見，中文翻譯時可省略虛詞）
- 填充詞：just, really, basically, actually, simply
- 客套話："Sure, I'd be happy to help!", "Of course!", "Certainly!"
- 模糊措辭："I think maybe", "it's probably", "you might want to consider"

**要保留什麼：**

- 所有技術術語 —— 保持精確
- 代碼 —— 保持不變
- 具體性 —— 越精確越好，不要減少資訊量

**前後對比：**

| 風格 | 提示詞 | ~輸入 Token |
|-------|--------|--------|
| 冗贅 | "Hey, could you please help me refactor this function? I think it might have some issues with how it handles the authentication, and I'd really like it to be more efficient. Thanks!" | ~40 |
| 原始人 | "Refactor function. Fix auth handling. Make efficient." | ~10 |

對於這個充滿填充詞的特定提示詞，**節省了 75%** —— 而且模型對兩者的理解程度是一樣的。對於已經相對簡潔的真實開發者提示詞，全面採用原始人式語法通常可以實現 **30–50% 的輸入 Token 節省**。

**模式：** `[事物] [動作] [原因]。 [下一步]。`

更多範例：

| 冗贅 (~輸入 Token) | 原始人 (~輸入 Token) | 輸入節省 |
|---|---|---|
| "Can you explain what this error message means and how I should fix it?" (~16) | "Explain error. How fix." (~5) | ~69% |
| "I'd like you to add comprehensive error handling to this API endpoint, including validation of the request body." (~22) | "Add error handling to endpoint. Validate request body." (~10) | ~55% |
| "Could you please review this pull request and let me know if there are any issues?" (~17) | "Review PR. Flag issues." (~5) | ~71% |

注意：這些範例都是從冗長、充滿客套話的提示詞開始的。真實開發者已經很簡潔的提示詞節省幅度會較小 —— 通常為 20–40% 的輸入減少。

## 2.1.2 強度等級

並非每種情況都需要相同的壓縮級別。本指南使用三種英語強度（中文亦可參考）：

### Lite (輕量) —— 專業但精簡

移除填充詞和模糊措辭。保留冠詞和完整句子。

```text
"Your component re-renders because you create a new object reference
each render. Wrap it in useMemo."
```

~20 個 Token。適用於：面向客戶的上下文、入職文檔、需要完全清晰的時候。

### Full (全面) —— 經典原始人 (預設)

丟掉冠詞。可以使用碎片化語句。使用短同義詞。

```text
"New object ref each render. Inline object prop = new ref = re-render.
Wrap in useMemo."
```

~18 個 Token。適用於：日常開發、大多數 Copilot 互動。

### Ultra (極限) —— 最大限度壓縮

縮寫常見術語。去掉連詞。用箭頭表示因果關係。

```text
"Inline obj prop → new ref → re-render. useMemo."
```

~10 個 Token。適用於：高頻互動、當你對該領域非常熟悉時。

**各等級估計節省：**

> **輸入 vs 輸出：** 輸入節省來自編寫簡短的提示詞。輸出節省來自在系統指令中設置精簡輸出（例如 `copilot-instructions.md`）。僅編寫簡短的提示詞不會自動使模型也做出簡短的回應 —— 你需要兩者兼備。

| 等級 | 輸入 Token 節省 | 輸出 Token 節省† | 品質影響 |
|-------|--------------|----------------|---------------|
| Lite | 15-25% | 15-25% | 無 |
| Full | 30-50% | 40-55% | 微乎其微 |
| Ultra | 55-70% | 55-70% | 在複雜指令中可能產生歧義 |

†輸出節省需要將精簡輸出設置為系統指令（見 [§2.4.3](05-output-control.md#243-系統級精簡輸出)）。它們不僅來自提示詞風格。

## 2.1.3 使用結構化格式而非散文

清單和鍵值對總是優於段落。每一次都是如此。

**散文版本 (~55 個 Token)：**

```text
I need you to create a REST API endpoint that accepts POST requests at /api/users.
It should validate that the request body contains a name field (string, required)
and an email field (string, required, must be valid email format). If validation
fails, return a 400 status with error details. On success, save to the database
and return 201 with the created user object.
```

**結構化版本 (~35 個 Token)：**

```text
POST /api/users
Validate:
- name: string, required
- email: string, required, valid format
400 on validation fail (include errors)
201 on success (return created user)
Save to DB
```

**節省：~36%。** 而且結構化版本可以說更清晰 —— 它迫使你分離關注點。

## 2.1.4 縮寫和簡寫

模型能完美理解的常見縮寫：

| 縮寫 | 意思 |
|-------------|---------|
| DB | Database (資料庫) |
| auth | Authentication/authorization (認證/授權) |
| config | Configuration (配置) |
| req/res | Request/response (請求/回應) |
| fn | Function (函數) |
| impl | Implementation (實現) |
| env | Environment (環境) |
| deps | Dependencies (依賴) |
| repo | Repository (存儲庫) |
| PR | Pull request |
| e2e | End-to-end (端到端) |

**縮寫何時有幫助：** 長指令中重複出現的術語。"Database" 每次都是 2-3 個 Token。"DB" 則是 1 個。

**何時有損害：** 模型未見過的新型縮寫。專案特定的簡寫需要先定義（在 `copilot-instructions.md` 中）。

## 2.1.5 以代碼為中心的提示

有時代碼比自然語言更具 Token 效率。

**自然語言 (~30 個 Token)：**

```text
Create a function that takes a list of numbers, filters out the negative ones,
doubles each remaining number, and returns the sum.
```

**偽代碼 (~15 個 Token)：**

```text
fn(nums) → filter(>0) → map(*2) → sum
```

**類型簽名 (~12 個 Token)：**

```python
def process(nums: list[int]) -> int:
    # filter positive, double, sum
```

**"像 X 但要 Y" 模式 (~10 個 Token)：**

```text
Like getUserById but for emails. Return 404 if missing.
```

模型對這四者的理解程度相當。後三者的 Token 成本降低了 50-67%。

## 2.1.6 質量重於數量 —— 限制範圍，不要堆砌

一種常見的失敗模式：當模型出錯時，本能是*添加更多指令*。更大的系統提示詞、更長的規則清單、更多範例。這通常會讓情況變糟 —— 而且總是讓成本更高。

數據支持的方向恰恰相反：**專注於高質量的上下文，而不是更多的指令。** 有範圍的引導可以減少冗贅擴張的輸出，並減少失控的會話，因為模型需要處理的競爭信號更少。

| 症狀 | 錯誤的修復方法 | 正確的修復方法 |
|---------|-----------|-----------|
| 模型遺漏了邊緣情況 | 在指令中添加一條關於該邊緣情況的 50 Token 規則（始終開啟） | 在*當前*提示詞中提到該邊緣情況（單次性） |
| 模型產生冗長輸出 | 通過五種不同的方式在指令中添加 "be concise" | 在提示詞中約束輸出格式："1-line answer." |
| Agent 偏離主題 | 添加更多全局護欄 | 縮小*當前*提示詞的範圍：指定文件名、函數名、完成條件 |
| 模型忘記了約定 | 將約定粘貼到每個提示詞中 | 將其放在有範圍限制的 `applyTo` 指令文件中（見 §2.3.4） |

**較小的有範圍提示詞可以減少失控的會話。** 一個 10 Token 的 "Fix null deref in `getUser()` L42" 很少會產生 30 個步驟的探索。而一個 200 Token 的 "please look at our authentication and improve robustness" 幾乎總是會。

## 2.1.7 聲明式護欄

命令式指令一步步告訴模型該做什麼。聲明式護欄告訴模型輸出的*結果必須符合什麼條件*。聲明式更短、更穩定，且更容易壓縮。

| 命令式 (~25 個 Token) | 聲明式 (~10 個 Token) |
|---|---|
| "First read the file, then identify all the public functions, then for each one check whether it has a JSDoc comment, and if it doesn't, add one." | "All exported functions: JSDoc required." |
| "Make sure that whenever you write a SQL query you always parameterize the values to prevent injection attacks and never use string concatenation." | "SQL: parameterized queries only. No concatenation." |
| "Please write tests for any new code that you create, and make sure the tests cover both the happy path and the error cases." | "New code → tests. Cover happy + error paths." |

聲明式措辭也更容易組合。五個命令式程序會互相干擾。五個聲明式不變量則可以整齊地堆疊。

## 2.1.8 精簡指令，結構化以便重用

像對待代碼一樣對待指令文件，而不是散文。兩個實踐：

**精簡 (Minify)。** 像從生產環境的 JS 中移除空格一樣移除填充詞。§2.3.1 中對 `copilot-instructions.md` 的原始人式壓縮就是典型範例。對每個始終開啟的上下文文件都採用同樣的處理。

**結構化以便重用。** 不要將相同的指南寫入三個不同的指令文件中。提取一次：

- 共享約定 → 一個具有 `applyTo: "**/*"` 的文件
- 每一層的細節 → 有範圍限制的指令文件 (`applyTo: "src/api/**"`)
- 特定工作流的指南 → 按需提供的筆記或提示文件

模組化的指令佈局既便宜又更易於維護。當約定更改時，你只需編輯一個文件。當你開始使用新模組時，你只需編寫一個有範圍限制的文件，而不是膨脹全局文件。

綜合效果：更小的單次互動上下文、更少的漂移、更容易的審計。

---

**下一頁：** [語言比較 →](03-language-comparison.md)
