---
allowed-tools: Bash, Read, Write, LS
---

# 關閉 Epic (Epic Close)

當所有任務都完成時，將一個 epic 標記為完成。

## 用法 (Usage)
```
/pm:epic-close <epic_name>
```

## 指示 (Instructions)

### 1. 驗證所有任務已完成

檢查 `.claude/epics/$ARGUMENTS/` 中的所有任務檔案：
-   驗證所有檔案的 frontmatter 中都有 `status: closed`。
-   如果發現任何未完成的任務：「❌ 無法關閉 epic。尚有未完成的任務：{list}」

### 2. 更新 Epic 狀態

獲取當前日期時間：`date -u +"%Y-%m-%dT%H:%M:%SZ"`

更新 epic.md 的 frontmatter：
```yaml
status: completed
progress: 100%
updated: {current_datetime}
completed: {current_datetime}
```

### 3. 更新 PRD 狀態

如果 epic 引用了某個 PRD，則將其狀態更新為 "complete"。

### 4. 在 GitHub 上關閉 Epic

如果 epic 有對應的 GitHub issue：
```bash
gh issue close {epic_issue_number} --comment "✅ Epic 已完成 - 所有任務均已結束"
```

### 5. 封存選項

詢問使用者：「是否封存已完成的 epic？(是/否)」

如果是：
-   將 epic 目錄移動到 `.claude/epics/.archived/{epic_name}/`
-   創建包含完成日期的封存摘要

### 6. 輸出

```
✅ Epic 已關閉: $ARGUMENTS
  已完成任務數: {count}
  持續時間: {從創建到完成的天數}
  
{如果已封存}: 已封存至 .claude/epics/.archived/

下一個 epic: 運行 /pm:next 查看優先工作
```

## 重要筆記 (Important Notes)

-   僅在所有任務都完成後才關閉 epics。
-   封存時保留所有資料。
-   更新相關的 PRD 狀態。