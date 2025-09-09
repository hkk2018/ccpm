# 分支操作 (Branch Operations)

Git 分支允許多個開發者在同一個儲存庫中進行隔離的變更，從而實現並行開發。

## 建立分支 (Creating Branches)

永遠從一個乾淨的 `main` 分支建立新分支：
```bash
# 確保 main 是最新的
git checkout main
git pull origin main

# 為 epic 建立分支
git checkout -b epic/{name}
git push -u origin epic/{name}
```

這樣會建立分支並將其推送到遠端 (origin)，並設定上游追蹤。

## 在分支中工作 (Working in Branches)

### 代理提交 (Agent Commits)
-   代理直接向分支提交。
-   使用小而專注的提交。
-   提交訊息格式：`Issue #{number}: {description}`
-   範例：`Issue #1234: 新增使用者認證結構`

### 檔案操作 (File Operations)
```bash
# 工作目錄就是當前目錄
# (不像 worktrees 那樣需要切換目錄)

# 可使用正常的 git 操作
git add {files}
git commit -m "Issue #{number}: {change}"

# 查看分支狀態
git status
git log --oneline -5
```

## 在同一分支中並行工作 (Parallel Work in Same Branch)

如果多個代理協調檔案存取，它們可以在同一個分支中工作：
```bash
# 代理 A 處理 API
git add src/api/*
git commit -m "Issue #1234: 新增使用者端點"

# 代理 B 處理 UI (需協調以避免衝突！)
git pull origin epic/{name}  # 獲取最新變更
git add src/ui/*
git commit -m "Issue #1235: 新增儀表板元件"
```

## 合併分支 (Merging Branches)

當 epic 完成後，將其合併回 `main`：
```bash
# 從主儲存庫
git checkout main
git pull origin main

# 合併 epic 分支
git merge epic/{name}

# 如果成功，進行清理
git branch -d epic/{name}
git push origin --delete epic/{name}
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

## 分支管理 (Branch Management)

### 列出活動分支 (List Active Branches)
```bash
git branch -a
```

### 移除過時分支 (Remove Stale Branch)
```bash
# 刪除本地分支
git branch -d epic/{name}

# 刪除遠端分支
git push origin --delete epic/{name}
```

### 檢查分支狀態 (Check Branch Status)
```bash
# 當前分支資訊
git branch -v

# 與 main 比較
git log --oneline main..epic/{name}
```

## 最佳實踐 (Best Practices)

1.  **每個 epic 一個分支** - 而非每個 issue 一個。
2.  **建立前保持乾淨** - 永遠從更新的 `main` 開始。
3.  **頻繁提交** - 小的提交更容易合併。
4.  **推送前先拉取** - 獲取最新變更以避免衝突。
5.  **使用描述性的分支名稱** - `epic/feature-name` 而不是 `feature`。

## 常見問題 (Common Issues)

### 分支已存在 (Branch Already Exists)
```bash
# 先刪除舊分支
git branch -D epic/{name}
git push origin --delete epic/{name}
# 然後再建立新的
```

### 無法推送分支 (Cannot Push Branch)
```bash
# 檢查遠端是否存在該分支
git ls-remote origin epic/{name}

# 帶上游推送
git push -u origin epic/{name}
```

### 拉取時發生合併衝突 (Merge Conflicts During Pull)
```bash
# 如有需要，暫存變更
git stash

# 拉取並變基
git pull --rebase origin epic/{name}

# 還原變更
git stash pop
```
