---
allowed-tools: Bash, Read, LS
---

# 顯示 Issue (Issue Show)

顯示 issue 及其子 issue 的詳細資訊。

## 用法 (Usage)
```
/pm:issue-show <issue_number>
```

## 指示 (Instructions)

您正在為 **Issue #$ARGUMENTS** 顯示一個 GitHub issue 及其相關子 issue 的綜合資訊。

### 1. 獲取 Issue 資料
-   使用 `gh issue view #$ARGUMENTS` 獲取 GitHub issue 詳細資訊。
-   尋找本地任務檔案：首先檢查 `.claude/epics/*/$ARGUMENTS.md`（新命名方式）。
-   如果找不到，則在 frontmatter 中搜索包含 `github:.*issues/$ARGUMENTS` 的檔案（舊命名方式）。
-   檢查相關的 issues 和子任務。

### 2. Issue 概覽
顯示 issue 標頭：
```
🎫 Issue #$ARGUMENTS: {Issue Title}
   狀態 (Status): {open/closed}
   標籤 (Labels): {labels}
   指派對象 (Assignee): {assignee}
   創建時間 (Created): {creation_date}
   更新時間 (Updated): {last_update}
   
📝 描述 (Description):
{issue_description}
```

### 3. 本地檔案映射
如果本地任務檔案存在：
```
📁 本地檔案:
   任務檔案: .claude/epics/{epic_name}/{task_file}
   更新: .claude/epics/{epic_name}/updates/$ARGUMENTS/
   上次本地更新: {timestamp}
```

### 4. 子 Issues 和依賴項
顯示相關的 issues：
```
🔗 相關 Issues:
   父 Epic: #{epic_issue_number}
   依賴於 (Dependencies): #{dep1}, #{dep2}
   阻擋 (Blocking): #{blocked1}, #{blocked2}
   子任務 (Sub-tasks): #{sub1}, #{sub2}
```

### 5. 最近活動
顯示最近的評論和更新：
```
💬 最近活動:
   {timestamp} - {author}: {comment_preview}
   {timestamp} - {author}: {comment_preview}
   
   查看完整討論串: gh issue view #$ARGUMENTS --comments
```

### 6. 進度追蹤
如果任務檔案存在，顯示進度：
```
✅ 驗收標準:
   ✅ 標準 1 (已完成)
   🔄 標準 2 (進行中)
   ⏸️ 標準 3 (受阻)
   □ 標準 4 (未開始)
```

### 7. 快速操作
```
🚀 快速操作:
   開始工作: /pm:issue-start $ARGUMENTS
   同步更新: /pm:issue-sync $ARGUMENTS
   新增評論: gh issue comment #$ARGUMENTS --body "your comment"
   在瀏覽器中查看: gh issue view #$ARGUMENTS --web
```

### 8. 錯誤處理
-   優雅地處理無效的 issue 編號。
-   檢查網路/認證問題。
-   提供有幫助的錯誤訊息和替代方案。

為開發者提供關於 Issue #$ARGUMENTS 的綜合 issue 資訊，以幫助他們了解上下文和當前狀態。
