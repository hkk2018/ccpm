---
allowed-tools: Bash, Read, Write, LS, Task
---

# 同步 Epic (Epic Sync)

將 epic 和任務作為 issue 推送到 GitHub。

## 用法 (Usage)
```
/pm:epic-sync <feature_name>
```

## 快速檢查 (Quick Check)

```bash
# 驗證 epic 是否存在
test -f .claude/epics/$ARGUMENTS/epic.md || echo "❌ 找不到 Epic。請運行：/pm:prd-parse $ARGUMENTS"

# 計算任務檔案數量
ls .claude/epics/$ARGUMENTS/*.md 2>/dev/null | grep -v epic.md | wc -l
```

如果找不到任務：「❌ 沒有要同步的任務。請運行：/pm:epic-decompose $ARGUMENTS」

## 指示 (Instructions)

### 0. 檢查遠端儲存庫

遵循 `/rules/github-operations.md` 以確保我們不會同步到 CCPM 樣板：

```bash
# 檢查遠端 origin 是否為 CCPM 樣板儲存庫
remote_url=$(git remote get-url origin 2>/dev/null || echo "")
if [[ "$remote_url" == *"automazeio/ccpm"* ]] || [[ "$remote_url" == *"automazeio/ccpm.git"* ]]; then
  echo "❌ 錯誤：您正試圖與 CCPM 樣板儲存庫同步！"
  echo ""
  echo "此儲存庫 (automazeio/ccpm) 是一個供他人使用的樣板。"
  echo "您不應該在此處創建 issue 或 PR。"
  echo ""
  echo "要修正此問題："
  echo "1. 將此儲存庫 fork 到您自己的 GitHub 帳戶"
  echo "2. 更新您的遠端 origin："
  echo "   git remote set-url origin https://github.com/YOUR_USERNAME/YOUR_REPO.git"
  echo ""
  echo "或者如果這是一個新專案："
  echo "1. 在 GitHub 上創建一個新的儲存庫"
  echo "2. 更新您的遠端 origin："
  echo "   git remote set-url origin https://github.com/YOUR_USERNAME/YOUR_REPO.git"
  echo ""
  echo "目前的遠端：$remote_url"
  exit 1
fi
```

### 1. 創建 Epic Issue

移除 frontmatter 並準備 GitHub issue 內容：
```bash
# 提取不含 frontmatter 的內容
sed '1,/^---$/d; 1,/^---$/d' .claude/epics/$ARGUMENTS/epic.md > /tmp/epic-body-raw.md

# 移除 "## Tasks Created" 區塊並替換為統計數據
awk '
  /^## Tasks Created/ {
    in_tasks=1
    next
  }
  /^## / && in_tasks {
    in_tasks=0
    # 當我們遇到「Tasks Created」之後的下一個區塊時，新增統計數據
    if (total_tasks) {
      print "## Stats\n"
      print "Total tasks: " total_tasks
      print "Parallel tasks: " parallel_tasks " (can be worked on simultaneously)"
      print "Sequential tasks: " sequential_tasks " (have dependencies)"
      if (total_effort) print "Estimated total effort: " total_effort " hours"
      print ""
    }
  }
  /^Total tasks:/ && in_tasks { total_tasks = $3; next }
  /^Parallel tasks:/ && in_tasks { parallel_tasks = $3; next }
  /^Sequential tasks:/ && in_tasks { sequential_tasks = $3; next }
  /^Estimated total effort:/ && in_tasks {
    gsub(/^Estimated total effort: /, "")
    total_effort = $0
    next
  }
  !in_tasks { print }
  END {
    # 如果在檔案結尾時我們仍在 tasks 區塊中，則新增統計數據
    if (in_tasks && total_tasks) {
      print "## Stats\n"
      print "Total tasks: " total_tasks
      print "Parallel tasks: " parallel_tasks " (can be worked on simultaneously)"
      print "Sequential tasks: " sequential_tasks " (have dependencies)"
      if (total_effort) print "Estimated total effort: " total_effort
    }
  }
' /tmp/epic-body-raw.md > /tmp/epic-body.md

# 從內容判斷 epic 類型 (feature vs bug)
if grep -qi "bug\|fix\|issue\|problem\|error" /tmp/epic-body.md; then
  epic_type="bug"
else
  epic_type="feature"
fi

# 創建帶有標籤的 epic issue
epic_number=$(gh issue create \
  --title "Epic: $ARGUMENTS" \
  --body-file /tmp/epic-body.md \
  --label "epic,epic:$ARGUMENTS,$epic_type" \
  --json number -q .number)
```

儲存返回的 issue 編號以供 epic frontmatter 更新。

### 2. 創建任務子 Issue

檢查 gh-sub-issue 是否可用：
```bash
if gh extension list | grep -q "yahsan2/gh-sub-issue"; then
  use_subissues=true
else
  use_subissues=false
  echo "⚠️ 未安裝 gh-sub-issue。正在使用後備模式。"
fi
```

計算任務檔案數量以決定策略：
```bash
task_count=$(ls .claude/epics/$ARGUMENTS/[0-9][0-9][0-9].md 2>/dev/null | wc -l)
```

### 對於小批次 (< 5 個任務)：循序創建

```bash
if [ "$task_count" -lt 5 ]; then
  # 對於小批次，循序創建
  for task_file in .claude/epics/$ARGUMENTS/[0-9][0-9][0-9].md; do
    [ -f "$task_file" ] || continue

    # 從 frontmatter 提取任務名稱
    task_name=$(grep '^name:' "$task_file" | sed 's/^name: *//')

    # 從任務內容中移除 frontmatter
    sed '1,/^---$/d; 1,/^---$/d' "$task_file" > /tmp/task-body.md

    # 創建帶有標籤的子 issue
    if [ "$use_subissues" = true ]; then
      task_number=$(gh sub-issue create \
        --parent "$epic_number" \
        --title "$task_name" \
        --body-file /tmp/task-body.md \
        --label "task,epic:$ARGUMENTS" \
        --json number -q .number)
    else
      task_number=$(gh issue create \
        --title "$task_name" \
        --body-file /tmp/task-body.md \
        --label "task,epic:$ARGUMENTS" \
        --json number -q .number)
    fi

    # 記錄映射以便重命名
    echo "$task_file:$task_number" >> /tmp/task-mapping.txt
  done

  # 創建所有 issue 後，更新參考並重命名檔案
  # 這遵循下面的步驟 3 的相同過程
fi
```

### 對於較大批次：並行創建

```bash
if [ "$task_count" -ge 5 ]; then
  echo "正在並行創建 $task_count 個子 issue..."

  # 檢查 gh-sub-issue 是否可用於並行代理
  if gh extension list | grep -q "yahsan2/gh-sub-issue"; then
    subissue_cmd="gh sub-issue create --parent $epic_number"
  else
    subissue_cmd="gh issue create"
  fi

  # 分批處理任務以進行並行處理
  # 生成代理以並行創建帶有適當標籤的子 issue
  # 每個代理必須使用：--label "task,epic:$ARGUMENTS"
fi
```

使用 Task 工具進行並行創建：
```yaml
Task:
  description: "創建 GitHub 子 issue 批次 {X}"
  subagent_type: "general-purpose"
  prompt: |
    為 epic $ARGUMENTS 中的任務創建 GitHub 子 issue
    父 epic issue: #$epic_number

    要處理的任務：
    - {3-4 個任務檔案的列表}

    對於每個任務檔案：
    1. 從 frontmatter 提取任務名稱
    2. 使用 sed '1,/^---$/d; 1,/^---$/d' 移除 frontmatter
    3. 使用以下命令創建子 issue：
       - 如果 gh-sub-issue 可用：
         gh sub-issue create --parent $epic_number --title "$task_name" \
           --body-file /tmp/task-body.md --label "task,epic:$ARGUMENTS"
       - 否則：
         gh issue create --title "$task_name" --body-file /tmp/task-body.md \
           --label "task,epic:$ARGUMENTS"
    4. 記錄：task_file:issue_number

    重要：始終包含帶有 "task,epic:$ARGUMENTS" 的 --label 參數

    返回檔案到 issue 編號的映射。
```

整合來自並行代理的結果：
```bash
# 從代理收集所有映射
cat /tmp/batch-*/mapping.txt >> /tmp/task-mapping.txt

# 重要：整合後，遵循步驟 3 以：
# 1. 建立舊 -> 新 ID 映射
# 2. 更新所有任務參考 (depends_on, conflicts_with)
# 3. 使用適當的 frontmatter 更新重命名檔案
```

### 3. 重命名任務檔案並更新參考

首先，建立一個舊編號到新 issue ID 的映射：
```bash
# 創建從舊任務編號 (001, 002 等) 到新 issue ID 的映射
> /tmp/id-mapping.txt
while IFS=: read -r task_file task_number; do
  # 從檔名中提取舊編號 (例如，從 001.md 中提取 001)
  old_num=$(basename "$task_file" .md)
  echo "$old_num:$task_number" >> /tmp/id-mapping.txt
done < /tmp/task-mapping.txt
```

然後重命名檔案並更新所有參考：
```bash
# 處理每個任務檔案
while IFS=: read -r task_file task_number; do
  new_name="$(dirname "$task_file")/${task_number}.md"

  # 讀取檔案內容
  content=$(cat "$task_file")

  # 更新 depends_on 和 conflicts_with 參考
  while IFS=: read -r old_num new_num; do
    # 將像 [001, 002] 這樣的陣列更新為使用新的 issue 編號
    content=$(echo "$content" | sed "s/\b$old_num\b/$new_num/g")
  done < /tmp/id-mapping.txt

  # 將更新後的內容寫入新檔案
  echo "$content" > "$new_name"

  # 如果與新檔案不同，則刪除舊檔案
  [ "$task_file" != "$new_name" ] && rm "$task_file"

  # 更新 frontmatter 中的 github 欄位
  # 將 GitHub URL 添加到 frontmatter
  repo=$(gh repo view --json nameWithOwner -q .nameWithOwner)
  github_url="https://github.com/$repo/issues/$task_number"

  # 使用 GitHub URL 和當前時間戳更新 frontmatter
  current_date=$(date -u +"%Y-%m-%dT%H:%M:%SZ")

  # 使用 sed 更新 github 和 updated 欄位
  sed -i.bak "/^github:/c\github: $github_url" "$new_name"
  sed -i.bak "/^updated:/c\updated: $current_date" "$new_name"
  rm "${new_name}.bak"
done < /tmp/task-mapping.txt
```

### 4. 使用任務列表更新 Epic (僅後備)

如果不使用 gh-sub-issue，則將任務列表添加到 epic：

```bash
if [ "$use_subissues" = false ]; then
  # 獲取當前的 epic 內容
  gh issue view {epic_number} --json body -q .body > /tmp/epic-body.md

  # 附加任務列表
  cat >> /tmp/epic-body.md << 'EOF'

  ## Tasks
  - [ ] #{task1_number} {task1_name}
  - [ ] #{task2_number} {task2_name}
  - [ ] #{task3_number} {task3_name}
  EOF

  # 更新 epic issue
  gh issue edit {epic_number} --body-file /tmp/epic-body.md
fi
```

使用 gh-sub-issue，這是自動的！

### 5. 更新 Epic 檔案

使用 GitHub URL、時間戳和真實的任務 ID 更新 epic 檔案：

#### 5a. 更新 Frontmatter
```bash
# 獲取 repo 資訊
repo=$(gh repo view --json nameWithOwner -q .nameWithOwner)
epic_url="https://github.com/$repo/issues/$epic_number"
current_date=$(date -u +"%Y-%m-%dT%H:%M:%SZ")

# 更新 epic frontmatter
sed -i.bak "/^github:/c\github: $epic_url" .claude/epics/$ARGUMENTS/epic.md
sed -i.bak "/^updated:/c\updated: $current_date" .claude/epics/$ARGUMENTS/epic.md
rm .claude/epics/$ARGUMENTS/epic.md.bak
```

#### 5b. 更新「已創建的任務」區塊
```bash
# 創建一個帶有更新後「已創建的任務」區塊的暫存檔
cat > /tmp/tasks-section.md << 'EOF'
## Tasks Created
EOF

# 添加每個任務及其真實的 issue 編號
for task_file in .claude/epics/$ARGUMENTS/[0-9]*.md; do
  [ -f "$task_file" ] || continue

  # 獲取 issue 編號 (不含 .md 的檔名)
  issue_num=$(basename "$task_file" .md)

  # 從 frontmatter 獲取任務名稱
  task_name=$(grep '^name:' "$task_file" | sed 's/^name: *//')

  # 獲取並行狀態
  parallel=$(grep '^parallel:' "$task_file" | sed 's/^parallel: *//')

  # 添加到 tasks 區塊
  echo "- [ ] #${issue_num} - ${task_name} (parallel: ${parallel})" >> /tmp/tasks-section.md
done

# 添加摘要統計
total_count=$(ls .claude/epics/$ARGUMENTS/[0-9]*.md 2>/dev/null | wc -l)
parallel_count=$(grep -l '^parallel: true' .claude/epics/$ARGUMENTS/[0-9]*.md 2>/dev/null | wc -l)
sequential_count=$((total_count - parallel_count))

cat >> /tmp/tasks-section.md << EOF

Total tasks: ${total_count}
Parallel tasks: ${parallel_count}
Sequential tasks: ${sequential_count}
EOF

# 替換 epic.md 中的「已創建的任務」區塊
# 首先，創建備份
cp .claude/epics/$ARGUMENTS/epic.md .claude/epics/$ARGUMENTS/epic.md.backup

# 使用 awk 替換區塊
awk '
  /^## Tasks Created/ {
    skip=1
    while ((getline line < "/tmp/tasks-section.md") > 0) print line
    close("/tmp/tasks-section.md")
  }
  /^## / && !/^## Tasks Created/ { skip=0 }
  !skip && !/^## Tasks Created/ { print }
' .claude/epics/$ARGUMENTS/epic.md.backup > .claude/epics/$ARGUMENTS/epic.md

# 清理
rm .claude/epics/$ARGUMENTS/epic.md.backup
rm /tmp/tasks-section.md
```

### 6. 創建映射檔案

創建 `.claude/epics/$ARGUMENTS/github-mapping.md`：
```bash
# 創建映射檔案
cat > .claude/epics/$ARGUMENTS/github-mapping.md << EOF
# GitHub Issue Mapping

Epic: #${epic_number} - https://github.com/${repo}/issues/${epic_number}

Tasks:
EOF

# 添加每個任務映射
for task_file in .claude/epics/$ARGUMENTS/[0-9]*.md; do
  [ -f "$task_file" ] || continue

  issue_num=$(basename "$task_file" .md)
  task_name=$(grep '^name:' "$task_file" | sed 's/^name: *//')

  echo "- #${issue_num}: ${task_name} - https://github.com/${repo}/issues/${issue_num}" >> .claude/epics/$ARGUMENTS/github-mapping.md
done

# 添加同步時間戳
echo "" >> .claude/epics/$ARGUMENTS/github-mapping.md
echo "Synced: $(date -u +"%Y-%m-%dT%H:%M:%SZ")" >> .claude/epics/$ARGUMENTS/github-mapping.md
```

### 7. 創建 Worktree

遵循 `/rules/worktree-operations.md` 創建開發 worktree：

```bash
# 確保 main 是最新的
git checkout main
git pull origin main

# 為 epic 創建 worktree
git worktree add ../epic-$ARGUMENTS -b epic/$ARGUMENTS

echo "✅ Created worktree: ../epic-$ARGUMENTS"
```

### 8. 輸出

```
✅ 已同步到 GitHub
  - Epic: #{epic_number} - {epic_title}
  - 任務：已創建 {count} 個子 issue
  - 已應用的標籤：epic, task, epic:{name}
  - 已重命名的檔案：001.md → {issue_id}.md
  - 已更新的參考：depends_on/conflicts_with 現在使用 issue ID
  - Worktree: ../epic-$ARGUMENTS

下一步：
  - 開始並行執行：/pm:epic-start $ARGUMENTS
  - 或處理單一 issue：/pm:issue-start {issue_number}
  - 查看 epic：https://github.com/{owner}/{repo}/issues/{epic_number}
```

## 錯誤處理

遵循 `/rules/github-operations.md` 處理 GitHub CLI 錯誤。

如果任何 issue 創建失敗：
-   報告成功的部分
-   註明失敗的部分
-   不要嘗試回滾（部分同步是可以的）

## 重要筆記

-   信任 GitHub CLI 的身份驗證
-   不要預先檢查重複項
-   僅在成功創建後更新 frontmatter
-   保持操作簡單和原子性
