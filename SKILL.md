---
name: checkmarx-sast
description: Scans project source code for common SAST-style vulnerabilities and generates a structured vulnerability remediation report; interprets Checkmarx SAST results and guides remediation by CWE/severity; configures CxFlow for CI/CD. Use when the user asks to scan the project, generate a security or vulnerability report, or when working with Checkmarx, CxSAST, CxFlow, SAST results, or fixing security findings.
---

# Checkmarx SAST 技能

協助**掃描專案並產出弱點修復報告**、解讀 Checkmarx SAST 結果、依嚴重性與 CWE 進行修復指引，以及 CxFlow 與 CI/CD 的設定要點。

## 適用時機

- **掃描專案並產生弱點修復報告**（依本技能掃描流程與報告範本產出）
- 解讀或說明 Checkmarx / CxSAST 掃描結果
- 依 CWE、嚴重性（Severity）判斷修復優先順序
- 撰寫或調整安全修復程式碼（含 Best Fix Location）
- 設定或排查 CxFlow、`application.yml`、CI/CD 整合
- 討論 SAST、SCA、程式碼安全掃描流程時

---

## 專案掃描與弱點修復報告

當使用者要求「掃描專案」或「產生弱點修復報告」時，依下列流程執行並產出報告。

### 掃描流程

1. **界定範圍**
   - 後端：`ebg_server` 或專案內 Java/Spring 原始碼（Controller、Service、Repository、設定檔）。
   - 前端：`ebg_client` 或專案內 Vue/JS/HTML（views、components、api、含使用者輸入或輸出的邏輯）。
   - 設定：`application.yml`、`.env`、環境或部署相關檔案（檢查是否有硬編碼密碼、金鑰、URL）。

2. **掃描項目（SAST 常見模式）**
   - **CWE-89 SQL Injection**：字串拼接組成 SQL、未使用參數化查詢 / PreparedStatement 的程式碼。
   - **CWE-79 XSS**：使用者可控資料未經編碼直接輸出到 HTML/JS（`v-html`、`innerHTML`、`document.write`、組字串成 HTML）。
   - **CWE-78 OS Command Injection**：使用者輸入傳入 `Runtime.exec`、`ProcessBuilder`、shell 指令組字串。
   - **CWE-22 Path Traversal**：檔案路徑含使用者輸入或 `..`、未做路徑驗證或正規化。
   - **CWE-798 硬編碼密碼／金鑰**：密碼、API key、secret 寫死在程式碼或設定檔（應改用環境變數或密鑰管理）。
   - **CWE-200 資訊洩漏**：stack trace、內部錯誤細節回傳給前端或記錄在可對外日誌。
   - **輸入驗證不足**：API 或表單輸入未驗證、未消毒即進入查詢或輸出。
   - **不安全的依賴或設定**：已知不安全的設定（例如 debug 開在正式環境、CORS 過寬）。

3. **嚴重性分級**
   - **High**：可直接導致 RCE、SQL Injection、認證繞過、敏感資料洩漏。
   - **Medium**：XSS、Path Traversal、資訊洩漏、不當錯誤處理。
   - **Low**：硬編碼非高敏感資訊、防禦性不足、最佳實踐偏離。
   - **Info**：建議改進、程式碼品質或可維護性相關。

4. **產出報告**
   - 使用下方「弱點修復報告範本」，依序填寫摘要、發現清單與各項修復建議。
   - 每個發現需包含：檔案路徑、行號（若可標示）、Severity、CWE、簡述、程式碼片段（可節錄）、修復建議。
   - 修復建議需具體（例如：改用 PreparedStatement、對輸出做 HTML 編碼、路徑用 `Path.normalize` 並限制根目錄）。

### 弱點修復報告範本

產出時請依此結構填寫（可存成 Markdown 檔或直接回覆）：

```markdown
# 弱點修復報告 — [專案名稱]

**掃描日期**：YYYY-MM-DD  
**掃描範圍**：[例如 ebg_server + ebg_client]

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

（其餘發現依相同格式重複）

---

## 4. 附註

- 本報告為靜態程式碼檢視結果，非取代正式 Checkmarx 掃描；建議仍於 CI 中執行 Checkmarx 以持續追蹤。
- 修復後請重新掃描或進行回歸測試以確認弱點已排除。
```

### 掃描時注意事項

- 僅根據可讀取的專案原始碼與設定進行檢視，不臆測執行時環境。
- 若專案有 `.cursor/rules` 或編碼規範，修復建議需與其一致。
- 報告中勿包含真實密碼、金鑰或個人資料；若需舉例，以佔位符取代。

## 解讀掃描結果

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
3. **修復**：依 CWE/Query 類型採用標準作法（例如 SQL 用參數化、輸出用編碼）
4. **驗證**：重新執行 SAST，確認該 finding 已消失且無新引入問題
5. **紀錄**：若為誤報，在 Checkmarx 中標記或調整 Query，避免重複處理

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
