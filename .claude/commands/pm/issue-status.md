---
allowed-tools: Bash, Read, LS
---

# Issue 狀態 (Issue Status)

檢查 issue 狀態（開啟/關閉）和當前狀況。

## 用法 (Usage)
```
/pm:issue-status <issue_number>
```

## 指示 (Instructions)

您正在為 **Issue #$ARGUMENTS** 檢查一個 GitHub issue 的當前狀態並提供快速狀態報告。

### 1. 獲取 Issue 狀態
使用 GitHub CLI 獲取當前狀態：
```bash
gh issue view #$ARGUMENTS --json state,title,labels,assignees,updatedAt
```

### 2. 狀態顯示
顯示簡潔的狀態資訊：
```
🎫 Issue #$ARGUMENTS: {Title}
   
📊 狀態 (Status): {OPEN/CLOSED}
   上次更新 (Last update): {timestamp}
   指派對象 (Assignee): {assignee or "Unassigned"}
   
🏷️ 標籤 (Labels): {label1}, {label2}, {label3}
```

### 3. Epic 上下文
如果 issue 是 epic 的一部分：
```
📚 Epic 上下文:
   Epic: {epic_name}
   Epic 進度: {completed_tasks}/{total_tasks} 個任務已完成
   此任務: {total_tasks} 中的第 {task_position} 個
```

### 4. 本地同步狀態
檢查本地檔案是否同步：
```
💾 本地同步:
   本地檔案: {exists/missing}
   上次本地更新: {timestamp}
   同步狀態: {in_sync/needs_sync/local_ahead/remote_ahead}
```

### 5. 快速狀態指示器
使用清晰的視覺指示器：
- 🟢 開啟且就緒
- 🟡 開啟但有阻礙
- 🔴 開啟且逾期
- ✅ 已關閉且完成
- ❌ 已關閉但未完成

### 6. 可行的下一步
根據狀態建議操作：
```
🚀 建議操作:
   - 開始工作: /pm:issue-start $ARGUMENTS
   - 同步更新: /pm:issue-sync $ARGUMENTS
   - 關閉 issue: gh issue close #$ARGUMENTS
   - 重新開啟 issue: gh issue reopen #$ARGUMENTS
```

### 7. 批次狀態
如果檢查多個 issues，支援逗號分隔的列表：
```
/pm:issue-status 123,124,125
```

保持輸出簡潔但資訊豐富，非常適合在開發 Issue #$ARGUMENTS 期間進行快速狀態檢查。
