---
allowed-tools: Bash, Read, Write, LS, Task
---

# 啟動 Epic (Epic Start)

啟動並行代理，在共享的分支中處理 epic 任務。

## 用法 (Usage)
```
/pm:epic-start <epic_name>
```

## 快速檢查 (Quick Check)

1. **驗證 epic 是否存在：**
   ```bash
   test -f .claude/epics/$ARGUMENTS/epic.md || echo "❌ 找不到 Epic。請運行 /pm:prd-parse $ARGUMENTS"
   ```

2. **檢查 GitHub 同步狀態：**
   在 epic frontmatter 中尋找 `github:` 欄位。
   如果缺失：「❌ Epic 尚未同步。請先運行 /pm:epic-sync $ARGUMENTS」

3. **檢查分支：**
   ```bash
   git branch -a | grep "epic/$ARGUMENTS"
   ```

4. **檢查未提交的變更：**
   ```bash
   git status --porcelain
   ```
   如果輸出不為空：「❌ 您有未提交的變更。請在啟動 epic 前提交或儲藏它們」

## 指示 (Instructions)

### 1. 創建或進入分支

遵循 `/rules/branch-operations.md`：

```bash
# 檢查未提交的變更
if [ -n "$(git status --porcelain)" ]; then
  echo "❌ 您有未提交的變更。請在啟動 epic 前提交或儲藏它們。"
  exit 1
fi

# 如果分支不存在，則創建它
if ! git branch -a | grep -q "epic/$ARGUMENTS"; then
  git checkout main
  git pull origin main
  git checkout -b epic/$ARGUMENTS
  git push -u origin epic/$ARGUMENTS
  echo "✅ 已創建分支: epic/$ARGUMENTS"
else
  git checkout epic/$ARGUMENTS
  git pull origin epic/$ARGUMENTS
  echo "✅ 正在使用現有的分支: epic/$ARGUMENTS"
fi
```

### 2. 識別就緒的 Issues

讀取 `.claude/epics/$ARGUMENTS/` 中的所有任務檔案：
-   解析 frontmatter 中的 `status`, `depends_on`, `parallel` 欄位
-   如果需要，檢查 GitHub issue 狀態
-   建立依賴關係圖

將 issues 分類：
-   **就緒 (Ready)**：沒有未滿足的依賴項，尚未開始
-   **受阻 (Blocked)**：有未滿足的依賴項
-   **進行中 (In Progress)**：已在處理中
-   **完成 (Complete)**：已結束

### 3. 分析就緒的 Issues

對於每個尚未分析的就緒 issue：
```bash
# 檢查是否有分析
if ! test -f .claude/epics/$ARGUMENTS/{issue}-analysis.md; then
  echo "正在分析 issue #{issue}..."
  # 運行分析 (內聯或通過 Task 工具)
fi
```

### 4. 啟動並行代理

對於每個已分析的就緒 issue：

```markdown
## 正在啟動 Issue #{issue}: {title}

正在讀取分析...
發現 {count} 個並行工作流：
  - 工作流 A: {description} (代理-{id})
  - 工作流 B: {description} (代理-{id})

正在分支中啟動代理：epic/$ARGUMENTS
```

使用 Task 工具為每個工作流啟動代理：
```yaml
Task:
  description: "Issue #{issue} 工作流 {X}"
  subagent_type: "{agent_type}"
  prompt: |
    在分支中工作：epic/$ARGUMENTS
    Issue: #{issue} - {title}
    工作流 (Stream): {stream_name}

    您的工作範圍：
    - 檔案 (Files): {file_patterns}
    - 工作 (Work): {stream_description}

    從以下位置讀取完整需求：
    - .claude/epics/$ARGUMENTS/{task_file}
    - .claude/epics/$ARGUMENTS/{issue}-analysis.md

    遵循 /rules/agent-coordination.md 中的協調規則

    使用以下格式頻繁提交：
    "Issue #{issue}: {specific change}"

    在以下位置更新進度：
    .claude/epics/$ARGUMENTS/updates/{issue}/stream-{X}.md
```

### 5. 追蹤活動的代理

創建/更新 `.claude/epics/$ARGUMENTS/execution-status.md`：

```markdown
---
started: {datetime}
branch: epic/$ARGUMENTS
---

# 執行狀態 (Execution Status)

## 活動的代理 (Active Agents)
- 代理-1: Issue #1234 工作流 A (資料庫) - 已於 {time} 啟動
- 代理-2: Issue #1234 工作流 B (API) - 已於 {time} 啟動
- 代理-3: Issue #1235 工作流 A (UI) - 已於 {time} 啟動

## 排隊中的 Issues (Queued Issues)
- Issue #1236 - 等待 #1234
- Issue #1237 - 等待 #1235

## 已完成 (Completed)
- {尚無}
```

### 6. 監控與協調

設定監控：
```bash
echo "
代理已成功啟動！

監控進度：
  /pm:epic-status $ARGUMENTS

查看分支變更：
  git status

停止所有代理：
  /pm:epic-stop $ARGUMENTS

完成後合併：
  /pm:epic-merge $ARGUMENTS
"
```

### 7. 處理依賴關係

當代理完成工作流時：
-   檢查是否有任何受阻的 issue 現在已就緒
-   為新就緒的工作啟動新的代理
-   更新 execution-status.md

## 輸出格式

```
🚀 Epic 執行已啟動: $ARGUMENTS

分支: epic/$ARGUMENTS

正在為 {issue_count} 個 issues 啟動 {total} 個代理：

Issue #1234: 資料庫結構 (Database Schema)
  ├─ 工作流 A: 結構創建 (代理-1) ✓ 已啟動
  └─ 工作流 B: 遷移 (代理-2) ✓ 已啟動

Issue #1235: API 端點 (API Endpoints)
  ├─ 工作流 A: 使用者端點 (代理-3) ✓ 已啟動
  ├─ 工作流 B: 貼文端點 (代理-4) ✓ 已啟動
  └─ 工作流 C: 測試 (代理-5) ⏸ 等待 A & B

受阻的 Issues (2):
  - #1236: UI 元件 (依賴於 #1234)
  - #1237: 整合 (依賴於 #1235, #1236)

使用 /pm:epic-status $ARGUMENTS 進行監控
```

## 錯誤處理

如果代理啟動失敗：
```
❌ 啟動代理-{id} 失敗
  Issue: #{issue}
  工作流: {stream}
  錯誤: {reason}

是否繼續其他代理？(是/否)
```

如果發現未提交的變更：
```
❌ 您有未提交的變更。請在啟動 epic 前提交或儲藏它們。

要提交變更：
  git add .
  git commit -m "您的提交訊息"

要儲藏變更：
  git stash push -m "進行中的工作"
  # (稍後使用 git stash pop 還原)
```

如果分支創建失敗：
```
❌ 無法創建分支
  {git 錯誤訊息}

嘗試：git branch -d epic/$ARGUMENTS
或：使用 git branch -a 檢查現有分支
```

## 重要筆記

-   遵循 `/rules/branch-operations.md` 進行 git 操作
-   遵循 `/rules/agent-coordination.md` 進行並行工作
-   代理在**同一個**分支中工作（不是分開的）
-   並行代理的最大數量應合理（例如，5-10）
-   如果啟動許多代理，請監控系統資源
