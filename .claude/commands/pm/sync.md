---
allowed-tools: Bash, Read, Write, LS
---

# 同步 (Sync)

在本地和 GitHub 之間進行完全的雙向同步。

## 用法 (Usage)
```
/pm:sync [epic_name]
```

如果提供了 epic_name，則僅同步該 epic。否則同步所有。

## 指示 (Instructions)

### 1. 從 GitHub 拉取

獲取所有 issues 的當前狀態：
```bash
# 獲取所有的 epic 和 task issues
gh issue list --label "epic" --limit 1000 --json number,title,state,body,labels,updatedAt
gh issue list --label "task" --limit 1000 --json number,title,state,body,labels,updatedAt
```

### 2. 從 GitHub 更新本地

對於每個 GitHub issue：
-   按 issue 編號尋找對應的本地檔案
-   比較狀態：
    -   如果 GitHub 狀態較新 (updatedAt > 本地 updated)，則更新本地
    -   如果 GitHub 已關閉但本地為開啟，則關閉本地
    -   如果 GitHub 已重新開啟但本地為關閉，則重新開啟本地
-   更新 frontmatter 以匹配 GitHub 狀態

### 3. 將本地推送到 GitHub

對於每個本地的 task/epic：
-   如果有 GitHub URL 但找不到 GitHub issue，表示它已被刪除 - 將本地標記為已封存
-   如果沒有 GitHub URL，則創建新的 issue (類似 epic-sync)
-   如果本地 updated > GitHub updatedAt，則推送變更：
    ```bash
    gh issue edit {number} --body-file {local_file}
    ```

### 4. 處理衝突

如果兩邊都已變更 (自上次同步以來本地和 GitHub 都已更新)：
-   顯示兩個版本
-   詢問使用者：「本地和 GitHub 都已變更。要保留哪個版本？(本地/github/合併)？」
-   應用使用者的選擇

### 5. 更新同步時間戳

用 last_sync 時間戳更新所有已同步的檔案。

### 6. 輸出

```
🔄 同步完成

從 GitHub 拉取：
  已更新：{count} 個檔案
  已關閉：{count} 個 issues
  
推送到 GitHub：
  已更新：{count} 個 issues
  已創建：{count} 個新 issues
  
已解決的衝突：{count}

狀態：
  ✅ 所有檔案已同步
  {或列出任何同步失敗的項目}
```

## 重要筆記 (Important Notes)

-   遵循 `/rules/github-operations.md` 執行 GitHub 命令。
-   遵循 `/rules/frontmatter-operations.md` 進行本地更新。
-   在同步前始終備份以防萬一。