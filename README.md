# Checkmarx SAST Agent Skill

本專案為 **Cursor／Claude Code 等支援 Agent Skills 的環境** 使用的技能套件，協助解讀 **Checkmarx SAST** 掃描結果、整理弱點修復說明、擬定 **MVP 最小修正方案**（維持既有行為），並提供 **CxFlow** 與 CI/CD 設定的參考要點。

## 功能概要

| 能力 | 說明 |
|------|------|
| 報告分析 | 解析使用者提供的 Checkmarx **PDF** 或置於約定目錄之報告，整理發現與修復建議 |
| 修復指引 | 依 **Severity**、**CWE** 排序與說明常見修法，對應報告中的 Query／檔案／行號 |
| MVP 方案 | 產出**變更範圍最小、不擴張需求**的修正計畫與驗證方式 |
| CxFlow／CI | 摘要 `application.yml`、break-build、倉庫整合等設定方向；細節見 `reference.md` |

**不包含**：在未提供 Checkmarx 報告或官方掃描結果的前提下，對程式庫進行「替代 Checkmarx」的獨立 SAST 掃描或產報。

## 專案結構

```
checkmarx_skills/
├── README.md          # 本說明
├── SKILL.md           # 技能主文件（frontmatter、工作流程、報告範本、輸出規則）
└── reference.md       # CxFlow／application.yml 進階參考與官方連結
```

若技能內文有約定 **incoming**（輸入報告）與 **outComing**（輸出 Markdown）目錄，請依 `SKILL.md` 內路徑於本機建立對應資料夾，或將路徑改為您環境下的絕對路徑。

## 使用方式

1. **安裝技能**  
   將本目錄複製或連結至您使用的技能目錄（例如 Cursor 的 `skills` 或團隊約定路徑），使助理能載入 `SKILL.md`。

2. **觸發情境**（節錄）  
   - 分析 Checkmarx PDF、匯出檔或對話中提供之路徑  
   - 詢問弱點的**最小修正**、CWE 對應修法、誤報處理  
   - 設定或排查 CxFlow、`application.yml`、CI/CD 整合  

3. **輸出規範**（以 `SKILL.md` 為準）  
   完整分析與報告內文應寫入約定之 **outComing** 目錄（主報告 + 修正細節等 Markdown），對話中僅簡短摘要與檔案路徑；請勿在聊天中貼上完整長篇報告全文。

4. **進階設定**  
   需要較完整的 YAML 屬性、環境變數覆寫與官方連結時，請開啟 [reference.md](reference.md)。

## 報告與修正原則（摘要）

- 發現應能**回溯至 Checkmarx 報告**，不臆造 finding。  
- **最小修正**：優先單點／Best Fix Location，避免順便重構或升級框架。  
- **行為保留**：修正以安全為主，不擅自改 API 契約或業務規則。  
- 敏感資訊勿寫入報告或範例程式；帳密與金鑰請用環境變數或密鑰管理。

## 參考資源

- [Checkmarx SAST 官方文件](https://docs.checkmarx.com/en/34965-44074-checkmarx-sast.html)  
- [CxFlow Configuration](https://github.com/checkmarx-ltd/cx-flow/wiki/Configuration)  
- [CI/CD Integrations](https://docs.checkmarx.com/en/34965-68684-ci-cd-integrations.html)  
- [CWE](https://cwe.mitre.org/)  

## 授權與責任

本技能為輔助解讀與文件化之用，**不取代** Checkmarx IDE／Pipeline 之正式掃描結果。實際修復與上線前請仍以官方工具與團隊安全流程為準。
