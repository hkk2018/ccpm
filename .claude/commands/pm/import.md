---
allowed-tools: Bash, Read, Write, LS
---

# 匯入 (Import)

將現有的 GitHub issue 匯入到專案管理系統中。

## 用法 (Usage)
```
/pm:import [--epic <epic_name>] [--label <label>]
```

選項:
- `--epic` - 匯入到特定的 epic 中
- `--label` - 僅匯入帶有特定標籤的 issue
- 無參數 - 匯入所有未追蹤的 issue

## 指示 (Instructions)

### 1. 獲取 GitHub Issues

```bash
# 根據過濾器獲取 issues
if [[ "$ARGUMENTS" == *"--label"* ]]; then
  gh issue list --label "{label}" --limit 1000 --json number,title,body,state,labels,createdAt,updatedAt
else
  gh issue list --limit 1000 --json number,title,body,state,labels,createdAt,updatedAt
fi
```

### 2. 識別未追蹤的 Issues

對於每個 GitHub issue：
-   在本地檔案中搜索匹配的 github URL
-   如果找不到，則表示它未被追蹤，需要匯入

### 3. 分類 Issues

根據標籤：
-   帶有 "epic" 標籤的 Issues → 創建 epic 結構
-   帶有 "task" 標籤的 Issues → 在適當的 epic 中創建任務
-   帶有 "epic:{name}" 標籤的 Issues → 分配到該 epic
-   沒有 PM 標籤的 Issues → 詢問使用者或在 "imported" epic 中創建

### 4. 創建本地結構

對於要匯入的每個 issue：

**如果是 Epic：**
```bash
mkdir -p .claude/epics/{epic_name}
# 使用 GitHub 內容和 frontmatter 創建 epic.md
```

**如果是 Task：**
```bash
# 尋找下一個可用的編號 (001.md, 002.md, 等)
# 使用 GitHub 內容創建任務檔案
```

設定 frontmatter：
```yaml
name: {issue_title}
status: {open|closed，基於 GitHub}
created: {GitHub createdAt}
updated: {GitHub updatedAt}
github: https://github.com/{org}/{repo}/issues/{number}
imported: true
```

### 5. 輸出

```
📥 匯入完成

已匯入：
  Epics: {count}
  Tasks: {count}
  
已創建的結構：
  {epic_1}/
    - {count} 個任務
  {epic_2}/
    - {count} 個任務
    
已跳過 (已被追蹤)：{count}

下一步：
  運行 /pm:status 查看已匯入的工作
  運行 /pm:sync 確保完全同步
```

## 重要筆記 (Important Notes)

-   在 frontmatter 中保留所有 GitHub 元數據。
-   用 `imported: true` 旗標標記已匯入的檔案。
-   不要覆蓋現有的本地檔案。