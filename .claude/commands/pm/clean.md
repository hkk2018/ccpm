---
allowed-tools: Bash, Read, Write, LS
---

# 清理 (Clean)

清理已完成的工作並封存舊的史詩任務 (epics)。

## 用法 (Usage)
```
/pm:clean [--dry-run]
```

選項:
- `--dry-run` - 顯示將被清理的內容，但不實際執行。

## 指示 (Instructions)

### 1. 識別已完成的 Epics

尋找符合以下條件的 epics：
-   frontmatter 中 `status: completed`
-   所有任務都已關閉
-   最後更新於 30 天前

### 2. 識別過時的工作

尋找：
-   已關閉 issue 的進度檔案
-   已完成工作的更新目錄
-   孤立的任務檔案（epic 已被刪除）
-   空目錄

### 3. 顯示清理計畫

```
🧹 清理計畫

待封存的已完成 Epics：
  {epic_name} - 完成於 {days} 天前
  {epic_name} - 完成於 {days} 天前
  
待移除的過時進度：
  {count} 個已關閉 issue 的進度檔案
  
空目錄：
  {list_of_empty_dirs}
  
可恢復空間：約 {size}KB

{如果使用 --dry-run}: 這是模擬運行，不會做任何變更。
{否則}: 是否繼續清理？(是/否)
```

### 4. 執行清理

如果使用者確認：

**封存 Epics：**
```bash
mkdir -p .claude/epics/.archived
mv .claude/epics/{completed_epic} .claude/epics/.archived/
```

**移除過時檔案：**
-   刪除超過 30 天的已關閉 issue 的進度檔案。
-   移除空的更新目錄。
-   清理孤立的檔案。

**建立封存日誌：**
建立 `.claude/epics/.archived/archive-log.md`：
```markdown
# 封存日誌

## {current_date}
- 已封存：{epic_name} (完成於 {date})
- 已移除：{count} 個過時的進度檔案
- 已清理：{count} 個空目錄
```

### 5. 輸出

```
✅ 清理完成

已封存：
  {count} 個已完成的 epics
  
已移除：
  {count} 個過時的檔案
  {count} 個空目錄
  
已恢復空間：{size}KB

系統現已乾淨整潔。
```

## 重要筆記 (Important Notes)

-   始終提供 `--dry-run` 以預覽變更。
-   絕不刪除 PRD 或未完成的工作。
-   保留封存日誌以供歷史查詢。