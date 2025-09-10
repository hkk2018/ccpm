---
allowed-tools: Bash, Read, Write, LS, Task
---

# 開始 Issue (Issue Start)

根據工作流分析，使用並行代理開始處理一個 GitHub issue。

## 用法 (Usage)
```
/pm:issue-start <issue_number>
```

## 快速檢查 (Quick Check)

1. **獲取 issue 詳細資訊：**
   ```bash
   gh issue view $ARGUMENTS --json state,title,labels,body
   ```
   如果失敗：「❌ 無法存取 issue #$ARGUMENTS。請檢查編號或運行：gh auth login」

2. **尋找本地任務檔案：**
   -   首先檢查 `.claude/epics/*/$ARGUMENTS.md` 是否存在（新命名方式）
   -   如果找不到，則在 frontmatter 中搜索包含 `github:.*issues/$ARGUMENTS` 的檔案（舊命名方式）
   -   如果找不到：「❌ 找不到 issue #$ARGUMENTS 的本地任務。此 issue 可能是在 PM 系統之外創建的。」

3. **檢查分析：**
   ```bash
   test -f .claude/epics/*/$ARGUMENTS-analysis.md || echo "❌ 找不到 issue #$ARGUMENTS 的分析
   
   請先運行：/pm:issue-analyze $ARGUMENTS
   或者：/pm:issue-start $ARGUMENTS --analyze 以同時進行兩者」
   ```
   如果不存在分析且沒有 --analyze 旗標，則停止執行。

## 指示 (Instructions)

### 1. 確保 Worktree 存在

檢查 epic worktree 是否存在：
```bash
# 從任務檔案中尋找 epic 名稱
epic_name={extracted_from_path}

# 檢查 worktree
if ! git worktree list | grep -q "epic-$epic_name"; then
  echo "❌ 找不到 epic 的 worktree。請運行：/pm:epic-start $epic_name"
  exit 1
fi
```

### 2. 讀取分析

讀取 `.claude/epics/{epic_name}/$ARGUMENTS-analysis.md`：
-   解析並行工作流
-   識別哪些可以立即開始
-   注意工作流之間的依賴關係

### 3. 設定進度追蹤

獲取當前日期時間：`date -u +"%Y-%m-%dT%H:%M:%SZ"`

創建工作區結構：
```bash
mkdir -p .claude/epics/{epic_name}/updates/$ARGUMENTS
```

使用當前日期時間更新任務檔案 frontmatter 的 `updated` 欄位。

### 4. 啟動並行代理

對於每個可以立即開始的工作流：

創建 `.claude/epics/{epic_name}/updates/$ARGUMENTS/stream-{X}.md`：
```markdown
---
issue: $ARGUMENTS
stream: {stream_name}
agent: {agent_type}
started: {current_datetime}
status: in_progress
---

# 工作流 {X}: {stream_name}

## 範圍 (Scope)
{stream_description}

## 檔案 (Files)
{file_patterns}

## 進度 (Progress)
- 正在開始實作
```

使用 Task 工具啟動代理：
```yaml
Task:
  description: "Issue #$ARGUMENTS 工作流 {X}"
  subagent_type: "{agent_type}"
  prompt: |
    您正在 epic worktree 中處理 Issue #$ARGUMENTS。
    
    Worktree 位置：../epic-{epic_name}/
    您的工作流：{stream_name}
    
    您的工作範圍：
    - 要修改的檔案：{file_patterns}
    - 要完成的工作：{stream_description}
    
    要求：
    1. 從以下位置讀取完整任務：.claude/epics/{epic_name}/{task_file}
    2. 僅在您被分配的檔案中工作
    3. 使用以下格式頻繁提交："Issue #$ARGUMENTS: {specific change}"
    4. 在以下位置更新進度：.claude/epics/{epic_name}/updates/$ARGUMENTS/stream-{X}.md
    5. 遵循 /rules/agent-coordination.md 中的協調規則
    
    如果您需要修改超出您範圍的檔案：
    - 檢查是否有另一個工作流擁有它們
    - 如有必要，請等待
    - 用協調筆記更新您的進度檔案
    
    完成您工作流的工作，並在完成時標記為完成。
```

### 5. GitHub 指派

```bash
# 指派給自己並標記為進行中
gh issue edit $ARGUMENTS --add-assignee @me --add-label "in-progress"
```

### 6. 輸出

```
✅ 已開始對 issue #$ARGUMENTS 進行並行工作

Epic: {epic_name}
Worktree: ../epic-{epic_name}/

正在啟動 {count} 個並行代理：
  工作流 A: {name} (代理-1) ✓ 已啟動
  工作流 B: {name} (代理-2) ✓ 已啟動
  工作流 C: {name} - 等待中 (依賴於 A)

進度追蹤：
  .claude/epics/{epic_name}/updates/$ARGUMENTS/

使用 /pm:epic-status {epic_name} 進行監控
使用 /pm:issue-sync $ARGUMENTS 同步更新
```

## 錯誤處理

如果任何步驟失敗，請清楚地報告：
- "❌ {什麼失敗了}: {如何修復}"
- 繼續進行可能的部分
- 絕不留下部分狀態

## 重要筆記

-   遵循 `/rules/datetime.md` 處理時間戳。
-   保持簡單——相信 GitHub 和檔案系統能正常工作。