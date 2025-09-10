---
allowed-tools: Read, LS
---

# Epic 一步到位 (Epic Oneshot)

在單一操作中將 epic 分解為任務並同步到 GitHub。

## 用法 (Usage)
```
/pm:epic-oneshot <feature_name>
```

## 指示 (Instructions)

### 1. 驗證先決條件

檢查 epic 是否存在且尚未被處理：
```bash
# Epic 必須存在
test -f .claude/epics/$ARGUMENTS/epic.md || echo "❌ 找不到 Epic。請運行：/pm:prd-parse $ARGUMENTS"

# 檢查現有任務
if ls .claude/epics/$ARGUMENTS/[0-9]*.md 2>/dev/null | grep -q .; then
  echo "⚠️ 任務已存在。這將會創建重複的任務。"
  echo "請刪除現有任務或改用 /pm:epic-sync。"
  exit 1
fi

# 檢查是否已同步
if grep -q "github:" .claude/epics/$ARGUMENTS/epic.md; then
  echo "⚠️ Epic 已同步到 GitHub。"
  echo "請使用 /pm:epic-sync 進行更新。"
  exit 1
fi
```

### 2. 執行分解

只需運行分解命令：
```
正在運行：/pm:epic-decompose $ARGUMENTS
```

這將會：
-   讀取 epic
-   創建任務檔案（如果適用，使用並行代理）
-   用任務摘要更新 epic

### 3. 執行同步

立即接著進行同步：
```
正在運行：/pm:epic-sync $ARGUMENTS
```

這將會：
-   在 GitHub 上創建 epic issue
-   創建子 issue（如果適用，使用並行代理）
-   將任務檔案重命名為 issue ID
-   創建 worktree

### 4. 輸出

```
🚀 Epic 一步到位完成：$ARGUMENTS

步驟 1：分解 ✓
  - 已創建任務數：{count}
  
步驟 2：GitHub 同步 ✓
  - Epic: #{number}
  - 已創建子 issue 數：{count}
  - Worktree: ../epic-$ARGUMENTS

準備好進行開發！
  開始工作：/pm:epic-start $ARGUMENTS
  或單一任務：/pm:issue-start {task_number}
```

## 重要筆記 (Important Notes)

這只是一個方便的包裝器，它會運行：
1. `/pm:epic-decompose` 
2. `/pm:epic-sync`

這兩個命令各自處理自己的錯誤檢查、並行執行和驗證。此命令只是按順序協調它們。

當您確信 epic 已準備就緒，並希望一步從 epic 到 GitHub issues 時，請使用此命令。