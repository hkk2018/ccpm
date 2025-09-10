---
allowed-tools: Bash, Read, Write
---

# 合併 Epic (Epic Merge)

將已完成的 epic 從 worktree 合併回主分支。

## 用法 (Usage)
```
/pm:epic-merge <epic_name>
```

## 快速檢查 (Quick Check)

1. **驗證 worktree 是否存在：**
   ```bash
   git worktree list | grep "epic-$ARGUMENTS" || echo "❌ 找不到 epic 的 worktree：$ARGUMENTS"
   ```

2. **檢查是否有活動的代理：**
   讀取 `.claude/epics/$ARGUMENTS/execution-status.md`
   如果存在活動的代理：「⚠️ 偵測到活動的代理。請先使用 /pm:epic-stop $ARGUMENTS 停止它們」

## 指示 (Instructions)

### 1. 合併前驗證

導航到 worktree 並檢查狀態：
```bash
cd ../epic-$ARGUMENTS

# 檢查未提交的變更
if [[ $(git status --porcelain) ]]; then
  echo "⚠️ worktree 中有未提交的變更："
  git status --short
  echo "請在合併前提交或儲藏變更"
  exit 1
fi

# 檢查分支狀態
git fetch origin
git status -sb
```

### 2. 運行測試 (選擇性但建議)

```bash
# 尋找測試命令
if [ -f package.json ]; then
  npm test || echo "⚠️ 測試失敗。是否仍要繼續？(是/否)"
elif [ -f Makefile ]; then
  make test || echo "⚠️ 測試失敗。是否仍要繼續？(是/否)"
fi
```

### 3. 更新 Epic 文件

獲取當前日期時間：`date -u +"%Y-%m-%dT%H:%M:%SZ"`

更新 `.claude/epics/$ARGUMENTS/epic.md`：
-   將狀態設為 "completed"
-   更新完成日期
-   新增最終摘要

### 4. 嘗試合併

```bash
# 返回主儲存庫
cd {main-repo-path}

# 確保 main 是最新的
git checkout main
git pull origin main

# 嘗試合併
echo "正在將 epic/$ARGUMENTS 合併到 main..."
git merge epic/$ARGUMENTS --no-ff -m "合併 epic: $ARGUMENTS

已完成的功能：
$(cd .claude/epics/$ARGUMENTS && ls *.md | grep -E '^[0-9]+' | while read f; do
  echo "- $(grep '^name:' $f | cut -d: -f2)"
done)

關閉 epic #$(grep 'github:' .claude/epics/$ARGUMENTS/epic.md | grep -oE '#[0-9]+')"
```

### 5. 處理合併衝突

如果合併因衝突而失敗：
```bash
# 檢查衝突狀態
git status

echo "
❌ 偵測到合併衝突！

衝突檔案：
$(git diff --name-only --diff-filter=U)

選項：
1. 手動解決：
   - 編輯衝突的檔案
   - git add {files}
   - git commit
   
2. 中止合併：
   git merge --abort
   
3. 尋求幫助：
   /pm:epic-resolve $ARGUMENTS

Worktree 保留在：../epic-$ARGUMENTS
"
exit 1
```

### 6. 合併後清理

如果合併成功：
```bash
# 推送到遠端
git push origin main

# 清理 worktree
git worktree remove ../epic-$ARGUMENTS
echo "✅ 已移除 Worktree: ../epic-$ARGUMENTS"

# 刪除分支
git branch -d epic/$ARGUMENTS
git push origin --delete epic/$ARGUMENTS 2>/dev/null || true

# 本地封存 epic
mkdir -p .claude/epics/archived/
mv .claude/epics/$ARGUMENTS .claude/epics/archived/
echo "✅ 已封存 Epic: .claude/epics/archived/$ARGUMENTS"
```

### 7. 更新 GitHub Issues

關閉相關的 issue：
```bash
# 從 epic 獲取 issue 編號
epic_issue=$(grep 'github:' .claude/epics/archived/$ARGUMENTS/epic.md | grep -oE '[0-9]+$')

# 關閉 epic issue
gh issue close $epic_issue -c "Epic 已完成並合併到 main"

# 關閉 task issues
for task_file in .claude/epics/archived/$ARGUMENTS/[0-9]*.md; do
  issue_num=$(grep 'github:' $task_file | grep -oE '[0-9]+$')
  if [ ! -z "$issue_num" ]; then
    gh issue close $issue_num -c "在 epic 合併中完成"
  fi
done
```

### 8. 最終輸出

```
✅ Epic 成功合併: $ARGUMENTS

摘要：
  分支：epic/$ARGUMENTS → main
  合併的提交數：{count}
  變更的檔案數：{count}
  關閉的 issue 數：{count}
  
清理完成：
  ✓ 已移除 Worktree
  ✓ 已刪除分支
  ✓ 已封存 Epic
  ✓ 已關閉 GitHub issues
  
下一步：
  - 如果需要，部署變更
  - 開始新的 epic: /pm:prd-new {feature}
  - 查看已完成的工作: git log --oneline -20
```

## 衝突解決幫助

如果需要解決衝突：
```
epic 分支與 main 有衝突。

這通常發生在：
- epic 開始後 main 已有變更
- 多個 epics 修改了相同的檔案
- 依賴項已更新

要解決：
1. 開啟衝突的檔案
2. 尋找 <<<<<<< 標記
3. 選擇正確的版本或合併
4. 移除衝突標記
5. git add {resolved files}
6. git commit
7. git push

或中止並稍後再試：
  git merge --abort
```

## 重要筆記

-   務必先檢查未提交的變更。
-   盡可能在合併前運行測試。
-   使用 --no-ff 以保留 epic 的歷史記錄。
-   封存 epic 資料而不是刪除。
-   關閉 GitHub issues 以保持同步。