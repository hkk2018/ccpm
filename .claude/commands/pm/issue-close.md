---
allowed-tools: Bash, Read, Write, LS
---

# 關閉 Issue (Issue Close)

將一個 issue 標記為完成並在 GitHub 上關閉它。

## 用法 (Usage)
```
/pm:issue-close <issue_number> [completion_notes]
```

## 指示 (Instructions)

### 1. 尋找本地任務檔案

首先檢查 `.claude/epics/*/$ARGUMENTS.md` 是否存在（新命名方式）。
如果找不到，則在 frontmatter 中搜索包含 `github:.*issues/$ARGUMENTS` 的任務檔案（舊命名方式）。
如果找不到：「❌ 找不到 issue #$ARGUMENTS 的本地任務」

### 2. 更新本地狀態

獲取當前日期時間：`date -u +"%Y-%m-%dT%H:%M:%SZ"`

更新任務檔案的 frontmatter：
```yaml
status: closed
updated: {current_datetime}
```

### 3. 更新進度檔案

如果進度檔案存在於 `.claude/epics/{epic}/updates/$ARGUMENTS/progress.md`：
-   設定完成度：100%
-   新增帶有時間戳的完成註記
-   使用當前日期時間更新 `last_sync`

### 4. 在 GitHub 上關閉

新增完成評論並關閉：
```bash
# 新增最終評論
echo "✅ 任務完成

$ARGUMENTS

---
關閉於: {timestamp}" | gh issue comment $ARGUMENTS --body-file -

# 關閉 issue
gh issue close $ARGUMENTS
```

### 5. 在 GitHub 上更新 Epic 任務列表

在 epic issue 中勾選任務核取方塊：

```bash
# 從本地任務檔案路徑獲取 epic 名稱
epic_name={extract_from_path}

# 從 epic.md 獲取 epic issue 編號
epic_issue=$(grep 'github:' .claude/epics/$epic_name/epic.md | grep -oE '[0-9]+$')

if [ ! -z "$epic_issue" ]; then
  # 獲取當前的 epic 內容
  gh issue view $epic_issue --json body -q .body > /tmp/epic-body.md
  
  # 勾選此任務
  sed -i "s/- \[ \] #$ARGUMENTS/- [x] #$ARGUMENTS/" /tmp/epic-body.md
  
  # 更新 epic issue
  gh issue edit $epic_issue --body-file /tmp/epic-body.md
  
  echo "✓ 已在 GitHub 上更新 epic 進度"
fi
```

### 6. 更新 Epic 進度

-   計算 epic 中的總任務數
-   計算已關閉的任務數
-   計算新的進度百分比
-   更新 epic.md frontmatter 的 progress 欄位

### 7. 輸出

```
✅ 已關閉 issue #$ARGUMENTS
  本地：任務已標記為完成
  GitHub：Issue 已關閉 & epic 已更新
  Epic 進度：{new_progress}% ({closed}/{total} 個任務已完成)
  
下一步：運行 /pm:next 查看下一個優先任務
```

## 重要筆記 (Important Notes)

-   遵循 `/rules/frontmatter-operations.md` 進行更新。
-   遵循 `/rules/github-operations.md` 執行 GitHub 命令。
-   始終在操作 GitHub 之前同步本地狀態。