---
allowed-tools: Bash, Read, Write, LS
---

# 編輯 Issue (Issue Edit)

在本地和 GitHub 上編輯 issue 的詳細資訊。

## 用法 (Usage)
```
/pm:issue-edit <issue_number>
```

## 指示 (Instructions)

### 1. 獲取當前 Issue 狀態

```bash
# 從 GitHub 獲取
gh issue view $ARGUMENTS --json title,body,labels

# 尋找本地任務檔案
# 搜索包含 github:.*issues/$ARGUMENTS 的檔案
```

### 2. 互動式編輯

詢問使用者要編輯什麼：
-   標題 (Title)
-   描述/內容 (Description/Body)
-   標籤 (Labels)
-   驗收標準 (Acceptance criteria) (僅限本地)
-   優先級/大小 (Priority/Size) (僅限本地)

### 3. 更新本地檔案

獲取當前日期時間：`date -u +"%Y-%m-%dT%H:%M:%SZ"`

用變更更新任務檔案：
-   如果標題已更改，更新 frontmatter 的 `name`
-   如果描述已更改，更新內容
-   使用當前日期時間更新 `updated` 欄位

### 4. 更新 GitHub

如果標題已更改：
```bash
gh issue edit $ARGUMENTS --title "{new_title}"
```

如果內容已更改：
```bash
gh issue edit $ARGUMENTS --body-file {updated_task_file}
```

如果標籤已更改：
```bash
gh issue edit $ARGUMENTS --add-label "{new_labels}"
gh issue edit $ARGUMENTS --remove-label "{removed_labels}"
```

### 5. 輸出

```
✅ 已更新 issue #$ARGUMENTS
  變更：
    {list_of_changes_made}
  
已同步到 GitHub：✅
```

## 重要筆記 (Important Notes)

-   始終先更新本地，然後再更新 GitHub。
-   保留未被編輯的 frontmatter 欄位。
-   遵循 `/rules/frontmatter-operations.md`。