# Worktree 操作

Git worktree 透過允許同一個儲存庫擁有多個工作目錄，來實現並行開發。

## 建立 Worktree (Creating Worktrees)

永遠從一個乾淨的 `main` 分支建立 worktree：
```bash
# 確保 main 是最新的
git checkout main
git pull origin main

# 為 epic 建立 worktree
git worktree add ../epic-{name} -b epic/{name}
```

Worktree 將會被建立為一個同層級的目錄，以保持乾淨的分離。

## 在 Worktree 中工作 (Working in Worktrees)

### 代理提交 (Agent Commits)
-   代理直接向 worktree 提交。
-   使用小而專注的提交。
-   提交訊息格式：`Issue #{number}: {description}`
-   範例：`Issue #1234: 新增使用者認證結構`

### 檔案操作 (File Operations)
```bash
# 工作目錄就是 worktree
cd ../epic-{name}

# 可使用正常的 git 操作
git add {files}
git commit -m "Issue #{number}: {change}"

# 查看 worktree 狀態
git status
```

## 在同一個 Worktree 中並行工作 (Parallel Work in Same Worktree)

如果多個代理接觸不同的檔案，它們可以在同一個 worktree 中工作：
```bash
# 代理 A 處理 API
git add src/api/*
git commit -m "Issue #1234: 新增使用者端點"

# 代理 B 處理 UI (無衝突！)
git add src/ui/*
git commit -m "Issue #1235: 新增儀表板元件"
```

## 合併 Worktree (Merging Worktrees)

當 epic 完成後，將其合併回 `main`：
```bash
# 從主儲存庫 (不是 worktree)
cd {main-repo}
git checkout main
git pull origin main

# 合併 epic 分支
git merge epic/{name}

# 如果成功，進行清理
git worktree remove ../epic-{name}
git branch -d epic/{name}
```

## 處理衝突 (Handling Conflicts)

如果發生合併衝突：
```bash
# 衝突將會顯示
git status

# 由人類解決衝突
# 然後繼續合併
git add {resolved-files}
git commit
```

## Worktree 管理 (Worktree Management)

### 列出活動的 Worktree (List Active Worktrees)
```bash
git worktree list
```

### 移除過時的 Worktree (Remove Stale Worktree)
```bash
# 如果 worktree 目錄已被刪除
git worktree prune

# 強制移除 worktree
git worktree remove --force ../epic-{name}
```

### 檢查 Worktree 狀態 (Check Worktree Status)
```bash
# 從主儲存庫
cd ../epic-{name} && git status && cd -
```

## 最佳實踐 (Best Practices)

1.  **每個 epic 一個 worktree** - 而非每個 issue 一個。
2.  **建立前保持乾淨** - 永遠從更新的 `main` 開始。
3.  **頻繁提交** - 小的提交更容易合併。
4.  **合併後刪除** - 不要留下過時的 worktree。
5.  **使用描述性的分支名稱** - `epic/feature-name` 而不是 `feature`。

## 常見問題 (Common Issues)

### Worktree 已存在 (Worktree Already Exists)
```bash
# 先移除舊的 worktree
git worktree remove ../epic-{name}
# 然後再建立新的
```

### 分支已存在 (Branch Already Exists)
```bash
# 刪除舊分支
git branch -D epic/{name}
# 或使用現有分支
git worktree add ../epic-{name} epic/{name}
```

### 無法移除 Worktree (Cannot Remove Worktree)
```bash
# 強制移除
git worktree remove --force ../epic-{name}
# 清理參考
git worktree prune
```