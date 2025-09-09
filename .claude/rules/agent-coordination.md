# 代理協調 (Agent Coordination)

多個代理在同一個 epic worktree 中並行工作的規則。

## 並行執行原則 (Parallel Execution Principles)

1.  **檔案級並行 (File-level parallelism)** - 在不同檔案上工作的代理絕不會衝突。
2.  **明確協調 (Explicit coordination)** - 當需要同一個檔案時，進行明確協調。
3.  **快速失敗 (Fail fast)** - 立即揭露衝突，不要試圖耍小聰明。
4.  **人類解決 (Human resolution)** - 衝突由人類解決，而非代理。

## 工作流分配 (Work Stream Assignment)

每個代理都會從問題分析中被分配一個工作流：
```yaml
# 來自 {issue}-analysis.md
工作流 A: 資料庫層 (Database Layer)
  檔案: src/db/*, migrations/*
  代理: 後端專家 (backend-specialist)

工作流 B: API 層 (API Layer)
  檔案: src/api/*
  代理: API 專家 (api-specialist)
```

代理應僅修改其被分配模式中的檔案。

## 檔案存取協調 (File Access Coordination)

### 修改前檢查 (Check Before Modify)
在修改共享檔案之前：
```bash
# 檢查檔案是否正在被修改
git status {file}

# 如果被另一個代理修改，則等待
if [[ $(git status --porcelain {file}) ]]; then
  echo "正在等待 {file} 變為可用..."
  sleep 30
  # 重試
fi
```

### 原子性提交 (Atomic Commits)
使提交保持原子性和專注：
```bash
# 好的 - 單一目的的提交
git add src/api/users.ts src/api/users.test.ts
git commit -m "Issue #1234: 新增使用者 CRUD 端點"

# 不好的 - 混合關注點
git add src/api/* src/db/* src/ui/*
git commit -m "Issue #1234: 多項變更"
```

## 代理之間的溝通 (Communication Between Agents)

### 透過提交 (Through Commits)
代理透過提交看到彼此的工作：
```bash
# 代理檢查其他人做了什麼
git log --oneline -10

# 代理拉取最新的變更
git pull origin epic/{name}
```

### 透過進度檔案 (Through Progress Files)
每個工作流維護進度：
```markdown
# .claude/epics/{epic}/updates/{issue}/stream-A.md
---
stream: 資料庫層 (Database Layer)
agent: 後端專家 (backend-specialist)
started: {datetime}
status: 進行中 (in_progress)
---

## 已完成 (Completed)
- 建立了使用者資料表結構
- 新增了遷移檔案

## 進行中 (Working On)
- 新增索引

## 受阻 (Blocked)
- 無
```

### 透過分析檔案 (Through Analysis Files)
分析檔案即是合約：
```yaml
# 代理讀取此檔案以了解邊界
Stream A:
  Files: src/db/*  # 代理 A 只接觸這些
Stream B:
  Files: src/api/* # 代理 B 只接觸這些
```

## 處理衝突 (Handling Conflicts)

### 衝突偵測 (Conflict Detection)
```bash
# 如果提交因衝突而失敗
git commit -m "Issue #1234: 更新"
# 錯誤: 存在衝突

# 代理應報告並等待
echo "❌ 在 {files} 中偵測到衝突"
echo "需要人類介入"
```

### 衝突解決 (Conflict Resolution)
永遠交由人類處理：
1.  代理偵測到衝突。
2.  代理報告問題。
3.  代理暫停工作。
4.  人類解決。
5.  代理繼續。

絕不嘗試自動合併解決。

## 同步點 (Synchronization Points)

### 自然同步點 (Natural Sync Points)
-   每次提交後。
-   開始新檔案前。
-   切換工作流時。
-   每工作 30 分鐘。

### 明確同步 (Explicit Sync)
```bash
# 拉取最新變更
git pull --rebase origin epic/{name}

# 如果有衝突，停止並報告
if [[ $? -ne 0 ]]; then
  echo "❌ 同步失敗 - 需要人類協助"
  exit 1
fi
```

## 代理溝通協議 (Agent Communication Protocol)

### 狀態更新 (Status Updates)
代理應定期更新其狀態：
```bash
# 每完成一個重要步驟就更新進度檔案
echo "✅ 已完成: 資料庫結構" >> stream-A.md
git add stream-A.md
git commit -m "進度: 工作流 A - 結構完成"
```

### 協調請求 (Coordination Requests)
當代理需要協調時：
```markdown
# 在 stream-A.md 中
## 需要協調 (Coordination Needed)
- 需要更新 src/types/index.ts
- 將在工作流 B 提交後進行修改
- 預計時間 (ETA): 10 分鐘
```

## 並行提交策略 (Parallel Commit Strategy)

### 不可能發生衝突時 (No Conflicts Possible)
當在完全不同的檔案上工作時：
```bash
# 這些可以同時發生
Agent-A: git commit -m "Issue #1234: 更新資料庫"
Agent-B: git commit -m "Issue #1235: 更新 UI"
Agent-C: git commit -m "Issue #1236: 新增測試"
```

### 需要時循序進行 (Sequential When Needed)
當接觸共享資源時：
```bash
# 代理 A 先提交
git add src/types/index.ts
git commit -m "Issue #1234: 更新類型定義"

# 代理 B 等待，然後繼續
# (在 A 提交後)
git pull
git add src/api/users.ts
git commit -m "Issue #1235: 使用新的類型"
```

## 最佳實踐 (Best Practices)

1.  **及早且頻繁地提交 (Commit early and often)** - 較小的提交 = 較少的衝突。
2.  **待在自己的車道上 (Stay in your lane)** - 只修改分配的檔案。
3.  **溝通變更 (Communicate changes)** - 更新進度檔案。
4.  **頻繁拉取 (Pull frequently)** - 與其他代理保持同步。
5.  **大聲失敗 (Fail loudly)** - 立即報告問題。
6.  **絕不強制 (Never force)** - 永遠不要使用 `--force` 旗標。

## 常見模式 (Common Patterns)

### 開始工作 (Starting Work)
```bash
1. cd ../epic-{name}
2. git pull
3. 檢查 {issue}-analysis.md 以了解分配的任務
4. 用 "started" 更新 stream-{X}.md
5. 開始在分配的檔案上工作
```

### 工作期間 (During Work)
```bash
1. 對分配的檔案進行變更
2. 用清晰的訊息提交
3. 更新進度檔案
4. 檢查來自他人的新提交
5. 根據需要繼續或協調
```

### 完成工作 (Completing Work)
```bash
1. 對工作流進行最終提交
2. 用 "completed" 更新 stream-{X}.md
3. 檢查其他工作流是否需要幫助
4. 報告完成
```
