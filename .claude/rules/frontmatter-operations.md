# Frontmatter 操作規則

處理 Markdown 檔案中 YAML frontmatter 的標準模式。

## 讀取 Frontmatter (Reading Frontmatter)

從任何 Markdown 檔案中提取 frontmatter：
1.  尋找檔案開頭 `---` 標記之間的內容。
2.  將其解析為 YAML。
3.  如果無效或缺失，使用合理的預設值。

## 更新 Frontmatter (Updating Frontmatter)

更新現有檔案時：
1.  保留所有現有欄位。
2.  僅更新指定的欄位。
3.  始終使用當前日期時間更新 `updated` 欄位（參見 `/rules/datetime.md`）。

## 標準欄位 (Standard Fields)

### 所有檔案
```yaml
---
name: {識別碼}
created: {ISO 日期時間}      # 創建後絕不更改
updated: {ISO 日期時間}      # 任何修改時更新
---
```

### 狀態值 (Status Values)
-   PRDs: `backlog`, `in-progress`, `complete`
-   Epics: `backlog`, `in-progress`, `completed`
-   Tasks: `open`, `in-progress`, `closed`

### 進度追蹤 (Progress Tracking)
```yaml
progress: {0-100}%           # 用於 epics
completion: {0-100}%         # 用於進度檔案
```

## 創建新檔案 (Creating New Files)

創建 Markdown 檔案時，務必包含 frontmatter：
```yaml
---
name: {來自參數或上下文}
status: {初始狀態}
created: {當前日期時間}
updated: {當前日期時間}
---
```

## 重要筆記 (Important Notes)

-   初次創建後，絕不修改 `created` 欄位。
-   始終使用來自系統的真實日期時間（參見 `/rules/datetime.md`）。
-   在嘗試解析之前，驗證 frontmatter 是否存在。
-   在所有檔案中使用一致的欄位名稱。