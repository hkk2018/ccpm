---
allowed-tools: Bash, Read, Write, LS
---

# 重新開啟 Issue (Issue Reopen)

重新開啟一個已關閉的 issue。

## 用法 (Usage)
```
/pm:issue-reopen <issue_number> [reason]
```

## 指示 (Instructions)

### 1. 尋找本地任務檔案

在 frontmatter 中搜索包含 `github:.*issues/$ARGUMENTS` 的任務檔案。
如果找不到：「❌ 找不到 issue #$ARGUMENTS 的本地任務」

### 2. 更新本地狀態

獲取當前日期時間：`date -u +"%Y-%m-%dT%H:%M:%SZ"`

更新任務檔案的 frontmatter：
```yaml
status: open
updated: {current_datetime}
```

### 3. 重設進度

如果進度檔案存在：
-   保留原始的開始日期
-   將完成度重設為先前的值或 0%
-   新增關於重新開啟原因的註記

### 4. 在 GitHub 上重新開啟

```bash
# 附上評論重新開啟
echo "🔄 正在重新開啟 issue

原因: $ARGUMENTS

---
重新開啟於: {timestamp}" | gh issue comment $ARGUMENTS --body-file -

# 重新開啟 issue
gh issue reopen $ARGUMENTS
```

### 5. 更新 Epic 進度

將此任務重新設為開啟後，重新計算 epic 進度。

### 6. 輸出

```
🔄 已重新開啟 issue #$ARGUMENTS
  原因: {reason_if_provided}
  Epic 進度: {updated_progress}%
  
使用 /pm:issue-start $ARGUMENTS 開始工作
```

## 重要筆記 (Important Notes)

-   在進度檔案中保留工作歷史。
-   不要刪除先前的進度，只需重設狀態。