---
allowed-tools: Read, Write, LS
---

# 編輯 Epic (Epic Edit)

在創建後編輯 epic 的詳細資訊。

## 用法 (Usage)
```
/pm:epic-edit <epic_name>
```

## 指示 (Instructions)

### 1. 讀取目前的 Epic

讀取 `.claude/epics/$ARGUMENTS/epic.md`：
-   解析 frontmatter
-   讀取內容區塊

### 2. 互動式編輯

詢問使用者要編輯什麼：
-   名稱/標題
-   描述/概覽
-   架構決策
-   技術方法
-   依賴項
-   成功標準

### 3. 更新 Epic 檔案

獲取當前日期時間：`date -u +"%Y-%m-%dT%H:%M:%SZ"`

更新 epic.md：
-   保留除了 `updated` 之外的所有 frontmatter
-   將使用者的編輯應用到內容中
-   使用當前日期時間更新 `updated` 欄位

### 4. 更新 GitHub 的選項

如果 epic 的 frontmatter 中有 GitHub URL：
詢問：「是否更新 GitHub issue？(是/否)」

如果是：
```bash
gh issue edit {issue_number} --body-file .claude/epics/$ARGUMENTS/epic.md
```

### 5. 輸出

```
✅ 已更新 epic: $ARGUMENTS
  變更的區塊: {sections_edited}
  
{如果 GitHub 已更新}: GitHub issue 已更新 ✅

查看 epic: /pm:epic-show $ARGUMENTS
```

## 重要筆記 (Important Notes)

-   保留 frontmatter 的歷史記錄（created、github URL 等）。
-   編輯 epic 時不要更改任務檔案。
-   遵循 `/rules/frontmatter-operations.md`。