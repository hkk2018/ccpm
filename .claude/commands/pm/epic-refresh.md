---
allowed-tools: Read, Write, LS
---

# 刷新 Epic (Epic Refresh)

根據任務狀態更新 epic 的進度。

## 用法 (Usage)
```
/pm:epic-refresh <epic_name>
```

## 指示 (Instructions)

### 1. 計算任務狀態

掃描 `.claude/epics/$ARGUMENTS/` 中的所有任務檔案：
-   計算總任務數
-   計算 `status: closed` 的任務數
-   計算 `status: open` 的任務數
-   計算正在進行中的任務數

### 2. 計算進度

```
進度 = (已關閉任務數 / 總任務數) * 100
```

四捨五入到最接近的整數。

### 3. 更新 GitHub 任務列表

如果 epic 有對應的 GitHub issue，同步任務的核取方塊：

```bash
# 從 epic.md 的 frontmatter 中獲取 epic issue 編號
epic_issue={extract_from_github_field}

if [ ! -z "$epic_issue" ]; then
  # 獲取當前的 epic 內容
  gh issue view $epic_issue --json body -q .body > /tmp/epic-body.md
  
  # 對於每個任務，檢查其狀態並更新核取方塊
  for task_file in .claude/epics/$ARGUMENTS/[0-9]*.md; do
    task_issue=$(grep 'github:' $task_file | grep -oE '[0-9]+$')
    task_status=$(grep 'status:' $task_file | cut -d: -f2 | tr -d ' ')
    
    if [ "$task_status" = "closed" ]; then
      # 標記為已選中
      sed -i "s/- \[ \] #$task_issue/- [x] #$task_issue/" /tmp/epic-body.md
    else
      # 確保未選中 (以防手動選中)
      sed -i "s/- \[x\] #$task_issue/- [ ] #$task_issue/" /tmp/epic-body.md
    fi
  done
  
  # 更新 epic issue
  gh issue edit $epic_issue --body-file /tmp/epic-body.md
fi
```

### 4. 決定 Epic 狀態

-   如果進度 = 0% 且沒有工作開始：`backlog`
-   如果進度 > 0% 且 < 100%：`in-progress`
-   如果進度 = 100%：`completed`

### 5. 更新 Epic

獲取當前日期時間：`date -u +"%Y-%m-%dT%H:%M:%SZ"`

更新 epic.md 的 frontmatter：
```yaml
status: {calculated_status}
progress: {calculated_progress}%
updated: {current_datetime}
```

### 6. 輸出

```
🔄 Epic 已刷新: $ARGUMENTS

任務:
  已關閉: {closed_count}
  未完成: {open_count}
  總計: {total_count}
  
進度: {old_progress}% → {new_progress}%
狀態: {old_status} → {new_status}
GitHub: 任務列表已更新 ✓

{如果已完成}: 運行 /pm:epic-close $ARGUMENTS 來關閉 epic
{如果在進行中}: 運行 /pm:next 查看優先任務
```

## 重要筆記 (Important Notes)

-   這在手動編輯任務或 GitHub 同步後很有用。
-   不要修改任務檔案，只修改 epic 狀態。
-   保留所有其他的 frontmatter 欄位。