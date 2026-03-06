# Checkmarx SAST 進階參考

供需要細部 CxFlow 設定或 CI/CD 整合時查閱。基本使用請以 [SKILL.md](SKILL.md) 為主。

## application.yml 常用屬性摘要

### checkmarx 區塊

```yaml
checkmarx:
  url: https://your-checkmarx-server
  username: ${CX_USERNAME}   # 建議用環境變數
  password: ${CX_PASSWORD}
  # 或 OAuth：client-id, client-secret
  project: "${repo-name}"
  preset: "Default"
  incremental: true          # 僅掃描變更
```

### cx-flow 區塊（節錄）

```yaml
cx-flow:
  break-build: true
  filter-severity: High,Medium
  # filter-category: 可依需要篩選
  # filter-cwe: 可限制特定 CWE
  bug-tracker: GitHub  # 或 GitLab, Jira, Azure 等
```

### 倉庫整合（以 GitHub 為例）

```yaml
github:
  url: https://api.github.com
  token: ${GITHUB_TOKEN}
  api-url: https://api.github.com
```

其餘平台見 [CxFlow Configuration](https://github.com/checkmarx-ltd/cx-flow/wiki/Configuration)。

## 環境變數覆寫

Spring Boot 慣例：`PROPERTY_NAME` 對應 `property.name`。例如：

- `CX_USERNAME` → `checkmarx.username`
- `CX_FLOW_BREAK_BUILD` → `cx-flow.break-build`

## 官方文件連結

- [Checkmarx SAST](https://docs.checkmarx.com/en/34965-44074-checkmarx-sast.html)
- [CxFlow Configuration](https://github.com/checkmarx-ltd/cx-flow/wiki/Configuration)
- [CxFlow Troubleshooting](https://github.com/checkmarx-ltd/cx-flow/wiki/Troubleshooting)
- [CI/CD Integrations](https://docs.checkmarx.com/en/34965-68684-ci-cd-integrations.html)
