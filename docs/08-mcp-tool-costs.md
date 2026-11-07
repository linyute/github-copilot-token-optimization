# 2.7 工具與 MCP 伺服器成本——隱藏的 Token 稅

[← 返回指南](index.md)

---

## 最佳化之前：先測量實際載入了什麼

在變更 MCP 伺服器或指示檔案之前，先測量內容視窗中實際載入了哪些內容。

![MCP 與工具堆疊：各別的修正方式可降低結構描述成本、回合數、shell 輸出、重複的環境探查讀取，以及可視性缺口。](../../source/docs/assets/diagrams/mcp-tooling-stack.svg)

**Copilot CLI：** 在工作階段進行中執行 `/context`，即可取得實際的明細：

```text
Context Usage  claude-opus-4.6 · 104k/200k tokens (52%)
System/Tools:  62.5k (31%)   ← 永遠載入：MCP + 指示 + 系統提示
Messages:      41.8k (21%)   ← 對話歷程記錄
Free Space:    55.3k (28%)
Buffer:        40.4k (20%)
```

**VS Code Copilot：** 檢查任務實際載入的工具與擴充功能。僅憑已安裝的伺服器數量無法判定 token 成本：主機可能會延後載入定義。如果某個擴充功能注入了您在撰寫程式碼時用不到的工具，請在該工作區停用它，或改用專注的 VS Code 設定檔。

**區分已安裝、已載入與依需求載入的內容：**

| 元件 | 載入時機 | 是否在內容視窗中？ |
|-----------|:-----:|:------------------:|
| MCP 工具定義 | 完整目錄或延後探索，視主機而定 | 已載入的定義與探索用 Metadata |
| 適用的代理程式／儲存庫指示 | 被選取或符合條件時 | 已載入的指示 |
| 系統提示 | 每次模型要求 | 是 |
| 技能 | 先載入探索用 Metadata；呼叫時才載入完整指示 | Metadata 與任何已呼叫的指引 |
| 對話歷程記錄 | 持續累積，受壓縮機制約束 | 要求中保留的歷程記錄 |

縮短一個內文從未載入的技能，並不會節省該內文的 token。請檢查已載入的定義、探索用 Metadata 與指示，而不是磁碟上的檔案大小。偶爾才需要的指引請使用專注的技能，任務需要的即時功能則使用 MCP 伺服器；兩者都沒有固定的成本優勢。

> 請在長時間工作階段之前選好工具與代理程式。變更 `System/Tools` 可能會使已快取的要求前綴失效。如果任務需要不同的設定，請以簡短的交接內容開啟新的工作階段。請參閱[快取 §2.3.5](04-context-management.md#235-caching-store-and-reuse-context-within-prompts)。

> Dina Berry（Microsoft/GitHub 內容貢獻者）測量了真實的 Copilot CLI 正式環境設定，發現單一 Azure 外掛程式每則訊息就載入約 27K 個 token——在她執行 `/context` 之前完全看不出來。[完整文章 →](https://dfberry.github.io/2026-05-06-tuning-up-copilot-context)

---

## 2.7.1 每個工具都要花費 Token

已載入的 MCP 工具定義包含其函式名稱、描述與參數結構描述。在代理程式的各個步驟中重複這些定義，會增加輸入 token。

每個定義的大致大小：

| 元件 | ~Token 數 |
|-----------|:-------:|
| 工具名稱 + 描述 | 20-50 |
| 參數結構描述（簡單） | 30-80 |
| 參數結構描述（複雜） | 100-300 |
| **每個工具合計** | **100-500** |

## 2.7.2 乘數問題

對於每個步驟都載入完整目錄的設定：

```text
載入的工具 = 伺服器數 × 每個伺服器的工具數 × 每個工具的 token 數

範例（重度設定）：
  10 個 MCP 伺服器 × 每個 5 個工具 × 平均 200 個 token = 10,000 個 token

代理程式模式每個任務會執行 5-25 個步驟。
工具定義在「每一個」步驟都會重新載入。

10,000 個 token × 15 個步驟 = 光是工具定義就要 150,000 個 token。
```

在這個範例中，結構描述定義在整個任務中佔了 150K 個 token，這還不包含提示、檔案讀取與回應。

## 2.7.3 工具呼叫成本

除了定義之外，每次工具*呼叫*也都會花費 token：

| 階段 | 成本 |
|-------|------|
| 函式名稱 + 參數（輸出 token） | 每次呼叫 20-200 |
| 結果剖析（輸入 token，下一步驟） | 每次呼叫 50-2,000+ |
| 代理程式推論該使用哪個工具 | 每個步驟 50-200 |

單一 `read_file` 呼叫可能只花 50 個 token 來發出，卻回傳 2,000 個 token 的檔案內容。代理程式接著會在下一步驟處理這所有內容。

## 2.7.4 前後對照：MCP 伺服器稽核

**示意用的完整目錄設定，並非實際測量的 Copilot 執行結果。之前——「我全部都啟用」（15 個 MCP 伺服器）：**

| 伺服器 | 工具數 | ~Token 數／步驟 |
|--------|:-----:|:------------:|
| GitHub | 40 | 4,000 |
| Filesystem | 8 | 800 |
| Docker | 12 | 1,200 |
| Database (Postgres) | 10 | 1,000 |
| Database (Redis) | 8 | 800 |
| Slack | 15 | 1,500 |
| Jira | 12 | 1,200 |
| AWS | 20 | 2,000 |
| GCP | 18 | 1,800 |
| Kubernetes | 15 | 1,500 |
| Monitoring (Datadog) | 10 | 1,000 |
| Email | 8 | 800 |
| Calendar | 6 | 600 |
| Search (Brave) | 3 | 300 |
| Context7 | 2 | 200 |
| **合計** | **187** | **~18,700** |

以 15 個代理程式步驟計算：光是工具結構描述就要 **280,500 個 token**。

**之後——「只保留撰寫程式碼所需」（3 個 MCP 伺服器）：**

| 伺服器 | 工具數 | ~Token 數／步驟 |
|--------|:-----:|:------------:|
| GitHub | 40 | 4,000 |
| Context7 | 2 | 200 |
| Filesystem | 8 | 800 |
| **合計** | **50** | **~5,000** |

以 15 個代理程式步驟計算：工具結構描述共 **75,000 個 token**。

在這個假設的設定中，差距為 **205,500 個結構描述 token**，約 73%。這並非實際測量到的任務或帳單減幅；快取與延後載入都會改變計算結果。

## 2.7.5 依工作區設定 MCP 伺服器

不要在全域啟用所有 MCP 伺服器。請使用工作區層級的設定：

**全域設定**（`settings.json` 使用者設定）——只放通用必備的伺服器：

```json
{
  "mcp": {
    "servers": {
      "github": {
        "command": "github-mcp-server",
        "args": ["stdio"]
      }
    }
  }
}
```

**依工作區設定**（`.vscode/mcp.json`）——只放專案專屬的伺服器：

```json
{
  "servers": {
    "postgres": {
      "command": "mcp-server-postgres",
      "args": ["postgresql://localhost/mydb"]
    }
  }
}
```

停用目前任務用不到的伺服器。即使沒有呼叫任何工具，它們已載入的定義仍會增加內容。

**也要檢查擴充功能提供的工具。** 有些擴充功能會加入技能、聊天參與者、代理程式或工具。請使用專注的工作區或設定檔；停用不必要的 AI 整合，而不是與其無關的編輯器功能。

## 2.7.6 實務指引

1. **依任務載入工具**——將 Postgres MCP 留給資料庫工作，而不是每個撰寫程式碼的工作階段都載入。請在長時間工作開始前選好工具組；若有變動，請參考連結的交接指引。
2. **檢查實際內容**——已載入的定義會增加輸入；延後載入的工具與已安裝的伺服器則是不同的情況。
3. **優先使用內建工具**——VS Code 的內建工具（檔案讀寫、終端機、搜尋）已由代理程式載入。再疊加一個 MCP 檔案系統伺服器是多餘的
4. **讓呼叫有明確目的**——讀取足夠的內容以正確行動。如果缺少內容導致重工，減少呼叫次數並沒有幫助。
5. **使用隨需指引**——技能可以教導偶爾使用的 CLI 工作流程，而不需要另一個伺服器。也請測量其已載入的 Metadata 與指示。
6. **長串 CLI 工具鏈可考慮 CodeAct**——外部外掛程式 [`copilot-codeact-plugin`](https://github.com/jsturtevant/copilot-codeact-plugin) 會把多個小型操作合併在一次沙箱執行中。它可以減少重複重播內容，但不會縮小結構描述大小。
7. **重複性的工作流程使用專注的代理程式**——精簡的工具與穩定的指示能讓設定保持一致。僅在您的 Copilot 介面支援時，才在設定檔中指定模型。
8. **過濾雜訊較多的 shell 輸出**——[RTK (Rust Token Killer)](https://github.com/rtk-ai/rtk) 或 [`snip`](https://github.com/edouard-claude/snip) 可以縮小結果大小。請確認失敗資訊與診斷細節仍然保留。請參閱 §2.7.7 與 §2.7.8。
9. **優先選擇精確的命令**——[`minimal-context-tools`](https://github.com/SebastienDegodez/copilot-instructions/tree/main/plugins/minimal-context-tools) 提供 `rg`、`fd`、`jq`、`yq`、`ast-grep` 及相關 CLI 的指引。在需要過濾器之前，鎖定目標的查詢就能避免大量輸出。

## 2.7.7 在來源端壓縮工具輸出：RTK

MCP 結構描述的負擔是「開始任何工作之前」就產生的成本。另一方面，代理程式執行的每個 shell 命令所產生的輸出，都會成為下一步驟的輸入 token。在大型 PR 上失敗的 `cargo test` 或 `git diff`，可能回傳 10,000–25,000 個 token 的原始文字——通過的測試行、未變更的 diff 前後文、建構雜訊——代理程式會完整讀取這一切。

Copilot 已經使用提示快取、延後載入的工具結構描述、WebSocket 傳輸、壓縮與輸出上限。VS Code 的終端機工具以頭尾截斷的方式限制輸出；Copilot CLI 也會告知模型使用 `head`、`tail`、`grep` 或 `awk` 等工具過濾大型結果。RTK 與 snip 的過濾時機更早，會在輸出到達這些上限之前就產生針對特定命令的摘要。

[**RTK (Rust Token Killer)**](https://github.com/rtk-ai/rtk) 會執行原始命令並過濾其輸出：移除雜訊、突顯失敗的測試、去除重複的記錄，或將檔案清單分組。結果變小會改變代理程式所看到的內容，因此請確認有用的診斷資訊仍然保留。

**運作方式，逐步說明：**
1. 代理程式發出 Bash 工具呼叫（`git status`、`cargo test` 等）
2. RTK 攔截 hook 並改寫為 `rtk git status` / `rtk cargo test`
3. RTK 執行真正的命令並擷取完整輸出
4. RTK 套用針對特定命令的過濾器
5. 代理程式收到過濾後的結果

**預估的輸出縮減幅度**（RTK 自己在中型 TypeScript/Rust 專案上的基準測試——實際節省量取決於您專案的輸出量）：

| 命令 | 原始輸出 | 使用 RTK | 縮減幅度 |
|---------|:-----------:|:--------:|:---------:|
| `ls` / `tree` | ~2,000 個 token | ~400 | -80% |
| `git status` | ~3,000 個 token | ~600 | -80% |
| `git diff` | ~10,000 個 token | ~2,500 | -75% |
| `cargo test` / `npm test` | ~25,000 個 token | ~2,500 | -90% |
| `grep` / `rg` | ~16,000 個 token | ~3,200 | -80% |
| `git log -n 10` | ~2,500 個 token | ~500 | -80% |

輸出冗長的命令可節省的絕對量最大。簡短的結果則沒有太多可過濾的空間。

**安裝：**

```bash
# macOS
brew install rtk

# Linux / macOS
curl -fsSL https://raw.githubusercontent.com/rtk-ai/rtk/refs/heads/master/install.sh | sh
```

**Windows 注意事項：** 目前 RTK 在類 Unix 的 shell 路徑上表現最佳。在 Windows 上，shell hook 的行為與路徑處理可能不夠穩固，特別是橫跨 PowerShell、Git Bash、WSL 與 VS Code 代理程式執行時。請將它視為試行方案，而非預設建議：在您的團隊實際使用的儲存庫與 shell 上進行測試，若設定導致命令失敗或行為雜亂，就略過它。

**為 Copilot 設定：**

對於 Copilot，RTK 會安裝 PreToolUse hook 與提示性的指示。專案範圍的設定會寫入 `.github/`；較新的 RTK 版本也記載了安裝在 `~/.copilot/` / `$COPILOT_HOME` 底下的全域 Copilot 安裝方式。由於 hook 合約的變動比靜態文件更快，在推廣之前請先在您的 Copilot 介面上驗證。

```bash
cd your-repo
rtk init --copilot

# 選用：在您的 RTK 版本支援時進行全域 Copilot 設定：
rtk init -g --copilot
```

啟用後，hook 是透明的：您自己的終端機命令不會改變；只有代理程式的 Bash 工具呼叫會被攔截。如果您的環境無法穩定支援全域 Copilot hook 路徑，請維持每個儲存庫各自的設定，並為每個團隊工作區記錄下來。

**其他 AI 工具（提供全域安裝）：**

```bash
rtk init -g                   # Claude Code（全域）
rtk init -g --gemini          # Gemini CLI（全域）
rtk init -g --agent cursor    # Cursor（專案層級）
rtk init --agent cline        # Cline / Roo Code（專案層級）
```

**範圍：** 此 hook 攔截代理程式發出的 **Bash 工具呼叫**。VS Code Copilot 的內建工具（`Read`、`Grep`、`Glob`）不會經過 Bash，因此不受影響。若您特別想要 RTK 過濾，請使用對應的 shell 命令（`cat`、`rg`、`find`）。

**與 MCP 縮減相輔相成：** 結構描述稽核減少已載入的定義；RTK 減少 shell 結果。兩者處理的是不同的成本。

## 2.7.8 RTK 的替代方案：snip

[`snip`](https://github.com/edouard-claude/snip) 同樣會在代理程式收到 shell 輸出之前加以過濾。主要差異在於可自訂性：RTK 使用編譯的 Rust 命令登錄表；snip 則使用 YAML 過濾器，團隊不需變更 Go 程式碼即可新增。

**snip 最具吸引力的情況：**

- 您想為專案專屬的工具自訂過濾器
- 您想透過 `snip gain` 取得本機節省統計
- 您偏好貢獻 YAML 過濾器，而非編譯式的命令規則
- 您需要 Copilot CLI hook 支援，且能在您的環境中驗證 hook 路徑

**安裝：**

```bash
brew install edouard-claude/tap/snip
# 或
go install github.com/edouard-claude/snip/cmd/snip@latest
```

**Copilot 設定：**

```bash
snip init --agent copilot
```

截至撰文時，snip 的 Copilot 路徑會為 Copilot CLI 寫入 `preToolUse` hook。VS Code Copilot 代理程式 hook 仍是變動中的介面，因此除非您的團隊已驗證確切的版本與工作區設定，否則請將 VS Code 的設定視為試行。

**過濾器模型：**

```yaml
name: "git-log"
match:
  command: "git"
  subcommand: "log"
pipeline:
  - action: "head"
    n: 20
```

Snip 支援行過濾、頭尾截斷、移除 ANSI、JSON 與規則運算式擷取、分組、去除重複、彙總與範本。專案本機的過濾器需要信任核准。由於它們控制模型所看到的內容，請將其視為工具設定來審查。

**RTK 與 snip 比較：**

| 選擇 | 適用時機 |
|--------|-----------|
| RTK | 您想要單一 Rust 二進位檔、廣泛的代理程式支援，以及編譯內建的預設值 |
| snip | 您想要 YAML 過濾器、本機統計，以及更容易的專案／團隊自訂 |

預設情況下，請勿在同一個命令路徑上同時疊加 RTK 與 snip。每個代理程式介面只選擇一層輸出過濾，然後進行測量。疊加可能造成輸出被重複截斷，並使偵錯更加困難。

## 2.7.9 周邊生態系：還有哪些應納入思維模型

並非每個 token 工具都是 RTK/snip 的直接替代方案。請將這些類別區分開來：

| 工具 | 類別 | 用途 | 注意事項 |
|------|----------|------------|--------|
| [`snip-ai/snip`](https://github.com/snip-ai/snip) | 以 Claude Code 為主的輸出過濾器 | 以感知 AST 的程式碼處理來最佳化 Read/Bash/Grep/Glob | 與 `edouard-claude/snip` 是不同的專案；未驗證 Copilot 路徑 |
| [Redcon / ContextBudget](https://github.com/natiixnt/ContextBudget) | 內容封裝 + 命令壓縮 | 想要命令壓縮器加上 CI 品質關卡的團隊工作流程 | 推出前應審查授權與產品界線 |
| [Headroom](https://github.com/headroomlabs-ai/headroom) | 全端壓縮包裝器 | 更廣泛的檔案、命令、記憶體與 MCP 壓縮實驗 | 在記載為標準設定之前，請先驗證 `headroom wrap copilot` |
| [Tokentop](https://github.com/tokentopapp/tokentop) | 本機 token 與成本可視性 | 適用於 Copilot CLI 及其他支援的代理程式的即時工作階段、模型、token、成本與消耗速率儀表板 | 僅供監控；不進行壓縮 |
| [Tokalator](https://github.com/vfaraji89/tokalator) | VS Code token 可視性 | 預算儀表板、模型／內容視窗感知、指示檔案掃描 | 僅供監控；不進行壓縮 |
| [token-optimizer](https://github.com/alexgreensh/token-optimizer) | 內容稽核／狀態工具 | 稽核過期的記憶體、設定、壓縮損失與模型路由 | PolyForm Noncommercial 授權 |
| [Caveman](https://github.com/JuliusBrussee/caveman) | 模型輸出壓縮 | 更簡短的助理回應與精簡風格套件 | 不會減少 shell 命令的輸入；提示的負擔也要考量 |
| [ACON](https://github.com/microsoft/acon) | 研究框架 | 長時程內容壓縮的學術基礎 | 不是可直接取代的開發人員工具 |

從穩定的工作階段設定與較少載入的 MCP 結構描述開始。使用精確的命令，若結果仍然過大，再加入一個輸出過濾器。

## 2.7.10 案例研究：縮小大型外掛程式的範圍——Azure MCP

在 Dina Berry 的 Copilot CLI 稽核中，Azure MCP 外掛程式每則訊息載入 **~27K 個 token**——比她其他所有 MCP 伺服器加起來還多。

原因在於：Azure MCP Server（v3.0.0-beta.6）在 56 個命名空間中公開了 259 個工具。在其預設的 `namespace` 模式下，它已依服務分組，但如果您只使用少數幾個 Azure 服務，那麼每則訊息中的大部分結構描述都是雜訊。

**選項 A——完全停用**（如果此工作階段不需要 Azure）：

```json
// ~/.copilot/settings.json
"azure@azure-skills": false
```

在回報的設定中，停用它使每則訊息減少了約 27K 個已載入的 token。其他版本或延後載入的主機可能會有差異。

**選項 B——命名空間範圍限定（建議）：**

使用 `--namespace` 只載入您需要的服務：

```bash
# 在您的 MCP 伺服器設定 args 中：
--namespace appservice --namespace cosmos --namespace keyvault --namespace storage
```

這會載入約 24 個工具（4 個命名空間），而非全部 56 個命名空間。您需要的功能仍然保留；其餘的則不載入。

**常見的開發人員組合：**

| 角色 | 要保留的命名空間 |
|---------|-------------------|
| Web 應用程式 | `appservice`、`cosmos`、`keyvault`、`storage`、`functions` |
| 資料／分析 | `cosmos`、`sql`、`kusto`、`eventhubs`、`storage` |
| DevOps／基礎結構 | `compute`、`aks`、`azureterraform`、`deploy`、`monitor` |
| AI/ML | `foundryextensions`、`search`、`speech`、`applicationinsights` |

**Azure MCP 伺服器模式**（控制工具的公開方式）：

| 模式 | 公開的工具 | 內容成本 |
|------|:------------:|:------------:|
| `namespace`（預設） | 每個服務命名空間一個工具 | 中等 |
| `consolidated` | 依使用者意圖分組 | 較低 |
| `single` | 單一路由工具處理所有事項 | 最低 |
| `all` | 每個操作各自為一個工具（259 個） | 非常高 |

**VS Code：** 以視覺化方式限定 Azure MCP 的範圍——聊天面板旁的齒輪圖示 → 在伺服器、命名空間或個別工具層級選取／取消選取。不需要設定檔。

Dina Berry 設定的**實測結果**：

```text
之前（預設設定）：   System/Tools 62.5k (31%) — Free Space 28%
之後（限定 Azure 範圍）：      System/Tools 35.2k (18%) — Free Space 45%
之後（+ 精簡指示）： System/Tools 25.5k (13%) — Free Space 67%
```

當 `System/Tools` 偏高時，請檢查大型外掛程式。完整文章：[dfberry.github.io](https://dfberry.github.io/2026-05-06-tuning-up-copilot-context)。

---

## 2.7.11 持久的程式碼庫地圖：Graphify

新的代理程式工作階段常常需要重新讀取檔案，才能了解匯入關係、呼叫路徑與元件配置。[Graphify](https://github.com/Graphify-Labs/graphify) 使用 tree-sitter AST 剖析儲存庫，並寫入一份持久的圖形，供結構化查詢使用。

![持久的圖形導覽：建構一張地圖，然後查詢結構，而不是重複進行大範圍的檔案讀取。](../../source/docs/assets/diagrams/graphify-navigation.svg)

```bash
uv tool install graphifyy

# 在儲存庫中建構或更新圖形
graphify .

# 查詢鎖定目標的結構，而不是讀取大範圍的檔案
graphify query "where is auth middleware?"
graphify explain "UserService"
graphify path "Router" "Database"
```

核心輸出：

```text
graphify-out/
├── graph.json       供代理程式查詢的圖形
├── graph.html       互動式視覺化瀏覽器
└── GRAPH_REPORT.md  人類可讀的社群、核心節點與建議問題
```

適用於大型儲存庫、重複的工作階段，以及跨檔案的結構性問題。如果讀幾個已知的檔案就能完成任務，則可略過。

- 重構之後請重建過期的圖形，或使用更新／監看／hook 流程。
- 將產生的報告視為地圖，而不是原始碼。
- 決定要分享圖形檔案，還是忽略它們並在本機建構。請勿提交違反儲存庫政策的敏感原始碼關係。
- AST 擷取在本機進行。針對文件或媒體的選用語意／深度擷取，可能會使用已設定的 AI 後端。
- Graphify 減少重複的環境探查讀取；RTK 或 snip 則減少 shell 結果的大小。兩者處理的是不同的成本。

---

**下一篇：** [比較與資料 →](09-comparisons-data.md)
