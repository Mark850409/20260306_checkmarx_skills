---
name: checkmarx-sast
description: Analyzes user-provided Checkmarx PDF reports or reports placed in the skill incoming folder; proposes MVP minimal, behavior-preserving fix plans that do not change existing functionality; interprets Checkmarx SAST results and guides remediation by CWE/severity; outputs structured remediation writeups to outComing folder (not chat) with both main markdown report and fix-details file; configures CxFlow for CI/CD. Does not perform standalone codebase SAST scanning. Use when the user asks to analyze a PDF report, minimal fixes for findings, or when working with Checkmarx, CxSAST, CxFlow, SAST results, or fixing security findings.
---

# Checkmarx SAST 技能

協助**讀取使用者提供的 Checkmarx PDF（或置於技能約定目錄之報告）並解析發現**、**依範本整理弱點修復說明**、**制訂 MVP 最小修正方案（維持既有行為、不影響現有功能）**、解讀 Checkmarx SAST 結果、依嚴重性與 CWE 進行修復指引，以及 CxFlow 與 CI/CD 的設定要點。

**不包含**：在未提供 Checkmarx 報告／結果的前提下，對工作區進行「替代 Checkmarx」的專案掃描或自動產報。

## 輸出方式（強制）

- **不得直接在對話畫面輸出完整分析結果**（包含完整報告內文、逐筆弱點詳情、完整修正建議列表）。
- 每次執行本技能時，**必須**將結果寫入：`C:\Users\zanehsu\.cursor\skills\checkmarx-sast\outComing\`
- 若 `outComing` 不存在，需先建立資料夾後再輸出。
- 每次執行至少輸出以下 2 份 Markdown：
  1. **主報告檔**（此次生成的弱點修復報告）
  2. **修正細節檔**（本次相對前一次的調整、修訂原因、差異與影響說明）
- 對話中僅回覆：輸出路徑、檔名、是否完成；如需摘要，僅提供 3-5 行高層摘要，不貼完整內容。
- 檔名建議（可依專案名調整）：
  - `YYYYMMDD_HHMM_Checkmarx弱點修復報告.md`
  - `YYYYMMDD_HHMM_修正細節.md`

## 適用時機

- **分析外部 Checkmarx 報告**：使用者於對話提供 **PDF 路徑／附件**，或將檔案放在本技能 **約定目錄**（見「外部報告輸入」）；解析其中發現，並依「弱點修復報告範本」整理輸出與 **MVP 最小修正方案**（不影響現有功能）
- **為每項（或優先級）發現制訂最小修正方案**：變更範圍最小、不改 API／合約／對外行為，並說明驗證方式
- 解讀或說明 Checkmarx / CxSAST 掃描結果
- 依 CWE、嚴重性（Severity）判斷修復優先順序
- 撰寫或調整安全修復程式碼（含 Best Fix Location）
- 設定或排查 CxFlow、`application.yml`、CI/CD 整合
- 討論 SAST、SCA、程式碼安全掃描流程時

---

## 弱點修復報告範本（輸出格式）

在**已具備 Checkmarx 報告或明細**（PDF、匯出檔，或使用者貼上之結果）的前提下，依下列方式整理輸出；**本技能不對專案原始碼執行替代性 SAST 掃描**。  
輸出時應優先寫入 `outComing`，避免在對話中直接展開完整報告。

### 產出報告時應包含

- 使用下方「弱點修復報告範本」，依序填寫摘要、發現清單與各項修復建議。
- 每個發現需包含：檔案路徑、行號（若報告有）、Severity、CWE、簡述、程式碼片段（可節錄）、修復建議，以及**最小修正方案**（見「最小修正方案（不影響現有功能）」）。
- 修復建議需具體（例如：改用 PreparedStatement、對輸出做 HTML 編碼、路徑用 `Path.normalize` 並限制根目錄）。
- **最小修正方案**需與報告範本中的對應欄位一致，並優先採用「單點、可驗證、不擴張需求」的作法。

### 弱點修復報告範本（Markdown）

產出時請依此結構填寫（**需存成 Markdown 檔於 `outComing`**）：

```markdown
# 弱點修復報告 — [專案名稱]

**報告日期**：YYYY-MM-DD  
**報告來源**：[例如 Checkmarx PDF／匯出檔名；若佐證工作區程式碼則註明模組]

---

## 1. 摘要

| 嚴重性 | 數量 |
|--------|------|
| High   | n    |
| Medium | n    |
| Low    | n    |
| Info   | n    |
| **合計** | **n** |

**建議**：優先修復 High，其次 Medium；Low/Info 納入技術債或下版處理。

---

## 2. 發現清單

| # | Severity | CWE | 簡述 | 檔案:行號 |
|---|----------|-----|------|-----------|
| 1 | High     | …   | …    | path:line |
| 2 | …        | …   | …    | …         |

---

## 3. 發現詳情與修復建議

### 發現 #1 — [簡短標題]

- **Severity**：High
- **CWE**：CWE-xxx
- **檔案**：`path/to/file`
- **位置**：約第 n 行
- **說明**：[一至兩句描述風險]
- **程式碼片段**：
  ````text
  [節錄有問題的程式碼]
  ````
- **修復建議**：[具體步驟或範例程式方向]
- **Best Fix Location**：[若與上述不同，註明建議修復的根因位置]
- **最小修正方案**（必填）：
  - **變更範圍**：僅列將修改的檔案／方法（避免無關重構）
  - **作法**：在不大改架構的前提下消除弱點（例如僅將拼接改為參數化、僅對輸出加編碼層）
  - **行為不變聲明**：說明為何修正後輸入／輸出／錯誤語意與修正前一致（或僅多安全邊界、無功能差異）
  - **驗證**：建議的單元／整合或手動檢查步驟（含回歸要點）

（其餘發現依相同格式重複）

---

## 4. 附註

- 本文件為依 **使用者提供之 Checkmarx 報告**（及如有之工作區原始碼核對）整理之修復說明，**非**取代官方 Checkmarx 掃描；仍以 IDE／Pipeline 之掃描結果為準。
- 修復後請於 Checkmarx 重新掃描並進行回歸測試以確認弱點已排除。
```

### 產出報告時注意事項

- 發現內容應**可回溯至 Checkmarx 報告明細**，不臆造 finding。
- 若佐證工作區程式碼，僅根據可讀取之原始碼與設定核對，不臆測執行時環境。
- 若專案有 `.cursor/rules` 或編碼規範，修復建議需與其一致。
- 報告中勿包含真實密碼、金鑰或個人資料；若需舉例，以佔位符取代。
- 另需同步產出一份「修正細節」檔，至少包含：本次修正項目、修正原因、調整差異、可能影響與驗證建議。

### 嚴重性分級（解讀報告時參考）

| 級別 | 說明 |
|------|------|
| **High** | 可直接導致 RCE、SQL Injection、認證繞過、敏感資料洩漏等 |
| **Medium** | XSS、Path Traversal、資訊洩漏、不當錯誤處理等 |
| **Low** | 硬編碼非高敏感資訊、防禦性不足、最佳實踐偏離等 |
| **Info** | 建議改進、程式碼品質或可維護性相關 |

### 常見 CWE 與修復聯想（解讀 Query／CWE 時參考）

解讀 Checkmarx 報告中的 CWE／規則名稱時，可對照下列類型擬定修法（**非**對專案自動掃描之依據）：

- **CWE-89**：SQL Injection — 參數化查詢、避免字串拼接 SQL。
- **CWE-79**：XSS — 輸出編碼、避免未消毒資料進 HTML/JS。
- **CWE-78**：OS Command Injection — 避免將使用者輸入拼入系統指令。
- **CWE-22**：Path Traversal — 路徑驗證與正規化、限制根目錄。
- **CWE-798**：硬編碼密碼／金鑰 — 改環境變數或密鑰管理。
- **CWE-200**：資訊洩漏 — 錯誤回應與日誌不暴露內部細節。
- 其餘：**輸入驗證**、**不安全的依賴或設定**等，依報告描述與官方 CWE 說明處理。

---

## 外部報告輸入（PDF／技能約定目錄）

當使用者提供 **Checkmarx 匯出之 PDF**，或將報告置於本技能約定目錄時，**優先依本節流程**解析發現，再銜接「最小修正方案（不影響現有功能）」產出 **MVP 修法**。

### 約定目錄（放置報告用）

- **路徑**：`C:\Users\zanehsu\.cursor\skills\checkmarx-sast\incoming\`
- **用途**：使用者可將 Checkmarx **PDF**（或團隊約定之匯出檔，如 `.pdf`）複製至此目錄；於對話中說明檔名或請助手讀取該目錄內檔案即可。
- **命名建議**：`Checkmarx_[專案]_[日期].pdf`，避免覆蓋他人檔案。
- **敏感資料**：目錄僅供本機分析；若報告含內網路徑或帳號資訊，回覆使用者時應節錄或匿名化。

### 輸入方式（擇一或併用）

1. **對話提供路徑**：使用者貼上 PDF 的**絕對路徑**（含置於專案、`incoming` 目錄、或下載資料夾）。
2. **附件／工作區**：使用者將 PDF 放入工作區或上述 `incoming` 目錄後，告知檔名。
3. **讀取工具**：使用環境允許的方式讀取 PDF（例如編輯器／助手可讀取之 PDF 讀取能力）；若需從二進位 PDF 萃取文字且環境支援，可依專案慣用方式處理。**若 PDF 為純影像掃描、無法抽取文字**，應請使用者改提供 Checkmarx 之 **CSV／XML／JSON 匯出**，或先 OCR，避免臆測發現內容。

### 從報告中應抽取的欄位（解析檢查清單）

依 Checkmarx 報告版面可能略有差異，盡量擷取下列資訊以利對應程式碼與 CWE：

| 擷取項目 | 用途 |
|----------|------|
| **Query / Rule 名稱** | 對照弱點類型與修復模式 |
| **Severity** | 排序與是否需當迭代處理 |
| **CWE**（若有） | 對應標準修法 |
| **Source／Sink 或節點說明** | 理解資料流 |
| **檔案路徑 + 行號** | 於工作區開啟原始碼核對 |
| **Code Snippet** | 與實際檔案比對，避免行號偏移 |
| **Best Fix Location**（若有） | 作為 MVP 修改優先錨點 |

若報告僅有彙總數字而無明細，應請使用者匯出 **詳細結果** 或 **含明細之 PDF／檔案**。

### 分析與產出流程

1. **建立發現清單表**：依 PDF 明細整理為表格（Severity、CWE、Query、檔案:行、摘要）。
2. **對照原始碼（若目前工作區為目標專案）**：依路徑與行號 **Read** 對應檔案；行號與報告不一致時，以 **Snippet** 搜尋（`Grep`）定位實際位置。
3. **判讀**：區分需修復之真實問題、與可能之**誤報**（簡述理由）；誤報仍可提供「如何於 Checkmarx 標記／抑制」之建議，但不強行改碼。
4. **MVP 最小修正方案**：對每一筆需修復之發現，依「最小修正方案（不影響現有功能）」填寫 **變更範圍、作法、行為不變聲明、驗證**；優先單檔／單方法修正。
5. **產出格式**：可直接使用本技能「弱點修復報告範本」之 **「3. 發現詳情與修復建議」** 結構，或精簡為「發現摘要 + MVP 方案 + 驗證」列表。

### 限制與誠實邊界

- **非取代官方掃描**：PDF 分析結果仍應以 IDE／Pipeline 內之 Checkmarx 為準；此處為協助解讀與擬定修法。
- **無原始碼時**：僅能依 CWE／Query 給**通用 MVP 方向**，無法保證與專案框架完全一致；取得工作區後應再收斂。
- **報告過期**：若分支／commit 已變更，行號可能漂移，務必以 Snippet 或關鍵字在現況程式碼中確認。

---

## 最小修正方案（不影響現有功能）

在產出報告或協助修復時，**必須同時**協助制訂**最小修正方案**：以**最少程式變更**消除或降低該項弱點，且**不改變既有對外行為與功能**（含 API 契約、回應格式、業務規則、使用者可見流程）。

### 原則

1. **最小變更面（Minimal diff）**  
   - 優先修改 **Best Fix Location** 或單一責任點；避免連帶重構、更名、搬檔，除非為消除弱點所必要。
2. **行為保留（Behavior-preserving）**  
   - 成功路徑：輸入／輸出資料結構與語意與修正前一致。  
   - 失敗路徑：仍回傳專案慣用的錯誤格式與 HTTP 狀態；不將 stack trace 或內部細節暴露給前端（與專案規範一致即可）。
3. **安全強化不當成功能變更**  
   - 參數化查詢、輸出編碼、路徑正規化、移除硬編碼密鑰等，屬**防禦性補強**，不應藉此改變業務邏輯或新增需求。
4. **避免「順便」**  
   - 不順便升級框架、不順便調整 API 版本、不順便改資料庫 schema，除非使用者明確要求且範圍獨立。

### 制訂步驟

1. **釐清資料流**：從入口（Controller／路由）到弱點行，標註哪些輸入為外部可控；確認修正僅影響「如何處理」而非「處理什麼業務」。
2. **選擇標準修法**：對應 CWE 採用業界標準單點修補（參數化、編碼、白名單、環境變數取代密文等），與本技能「常見 CWE 對應修復方向」一致。
3. **評估副作用**：列出若修正錯可能影響的呼叫端；若需加驗證，驗證規則應與**既有欄位語意**相容（不任意收緊導致合法輸入被拒）。
4. **寫明驗證**：每個最小方案至少包含—**相同測試案例應仍通過**；針對弱點類型補一條**安全向**驗證（例如惡意字串被阻擋或無害化）。

### 與「大改」的界線

| 應採最小方案 | 應另開需求／排程（非本技能預設產出） |
|--------------|--------------------------------------|
| 單一方法內改寫查詢或加編碼 | 全面換 ORM、重寫模組 |
| 設定改走環境變數、不寫死密鑰 | 導入全新密鑰管理服務與登入流程 |
| 在現有錯誤處理鏈中統一不洩漏細節 | 重新設計 API 錯誤碼與前端契約 |

### 與報告的關係

- 依 Checkmarx 報告整理之每一筆 **發現詳情**，應盡可能含 **「最小修正方案」** 子欄位（見「弱點修復報告範本」）。
- 若使用者僅要求「方案」而未要求完整報告，仍可依同一結構僅輸出：**發現摘要 + 最小修正方案 + 驗證建議**。

## 解讀 Checkmarx 結果（欄位）

### 結果欄位

| 欄位 | 說明 |
|------|------|
| **Severity** | High / Medium / Low / Info，決定修復優先順序 |
| **CWE** | Common Weakness Enumeration，標準化弱點類型（如 SQL Injection、XSS） |
| **Query** | Checkmarx 偵測規則名稱 |
| **Node / Snippet** | 程式碼位置與片段，用於定位問題 |
| **Best Fix Location** | 建議修復的根因位置，修一次可解決多個相關發現 |

### 優先順序建議

1. **High**：先修，尤其是可導致 RCE、SQL Injection、認證繞過等
2. **Medium**：排入當次迭代或下一版
3. **Low / Info**：依專案政策與技術債規劃處理

依 CWE 對應到 OWASP、CWE 官方說明，再決定具體修復方式（參數化查詢、輸出編碼、輸入驗證等）。

## 修復流程

1. **定位**：依結果中的檔案、行號與 Snippet 找到對應程式碼
2. **根因**：優先查看 Best Fix Location，避免只治標
3. **最小方案**：依「最小修正方案（不影響現有功能）」章節，先收斂變更範圍與行為保留條件，再動手改碼
4. **修復**：依 CWE/Query 類型採用標準作法（例如 SQL 用參數化、輸出用編碼）
5. **驗證**：重新執行 SAST，確認該 finding 已消失且無新引入問題；並執行／確認與該變更相關之回歸（見最小方案中的驗證欄位）
6. **紀錄**：若為誤報，在 Checkmarx 中標記或調整 Query，避免重複處理

### 常見 CWE 對應修復方向

- **CWE-89 (SQL Injection)**：使用參數化查詢 / PreparedStatement，避免字串拼接 SQL
- **CWE-79 (XSS)**：輸出時編碼（HTML entity 等），或使用安全框架的輸出機制
- **CWE-78 (OS Command Injection)**：避免直接拼接使用者輸入至系統指令；改用白名單或 API
- **CWE-22 (Path Traversal)**：驗證與正規化路徑，禁止 `..` 等跳出目錄

修復時需符合專案既有規範（如 `java-coding-standards.mdc`、`api-design-standards.mdc`）。

## CxFlow 與 CI/CD 設定要點

CxFlow 以 Spring Boot 執行，透過 `application.yml`（或指令列、環境變數）設定。

### 常用設定區塊

- **server**：`server.port`（預設 8080）
- **cx-flow**：Bug tracker、分支過濾、嚴重性/類別/CWE 過濾、break-build 政策
- **checkmarx**：連線與認證（如 base URL、username、password 或 client-id/client-secret）、掃描設定（preset、incremental）
- **Repository**：依使用平台設定 GitHub、GitLab、Azure DevOps、Bitbucket 等對應區塊

### 設定優先順序

指令列參數 > 環境變數 > `application.yml`。敏感資訊應使用環境變數，勿寫死於設定檔。

### Break-build 政策

在 `cx-flow` 下可依 severity、狀態、CWE 等條件設定掃描失敗時是否讓 pipeline 失敗，以配合團隊門檻。

### 設定檔位置

`application.yml` 通常與 CxFlow JAR 同目錄，或透過 `--spring.config.location=/path/to/application.yml` 指定。路徑使用正斜線，例如：`config/application.yml`。

## 進階參考

- 更完整的 CxFlow 設定範例與屬性說明，見 [reference.md](reference.md)。

## 參考資源

- [Checkmarx SAST 官方文件](https://docs.checkmarx.com/en/34965-44074-checkmarx-sast.html)
- [CxFlow 設定說明](https://github.com/checkmarx-ltd/cx-flow/wiki/Configuration)
- [CI/CD 整合](https://docs.checkmarx.com/en/34965-68684-ci-cd-integrations.html)
- CWE：https://cwe.mitre.org/ （查詢弱點定義與修復建議）

## 注意事項

- 勿在程式碼或設定檔中寫死 Checkmarx 帳密、API 金鑰；使用環境變數或密鑰管理服務。
- 解讀結果時若需引用專案程式碼，請依既有專案結構與命名規範。
- 若團隊使用 Checkmarx One 或 SCA，補強依賴掃描與授權政策時可另訂 SCA 相關流程。
- 預設交付形式為 `outComing` 目錄中的 Markdown 檔案，不以對話全文輸出結果取代檔案交付。
