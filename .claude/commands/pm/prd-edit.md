---
allowed-tools: Read, Write, LS
---

# 編輯 PRD (PRD Edit)

編輯現有的產品需求文件 (Product Requirements Document)。

## 用法 (Usage)
```
/pm:prd-edit <feature_name>
```

## 指示 (Instructions)

### 1. 讀取目前的 PRD

讀取 `.claude/prds/$ARGUMENTS.md`：
-   解析 frontmatter
-   讀取所有區塊

### 2. 互動式編輯

詢問使用者要編輯哪些區塊：
-   執行摘要 (Executive Summary)
-   問題陳述 (Problem Statement)
-   使用者故事 (User Stories)
-   需求 (功能性/非功能性) (Requirements (Functional/Non-Functional))
-   成功標準 (Success Criteria)
-   限制與假設 (Constraints & Assumptions)
-   範圍外 (Out of Scope)
-   依賴項 (Dependencies)

### 3. 更新 PRD

獲取當前日期時間：`date -u +"%Y-%m-%dT%H:%M:%SZ"`

更新 PRD 檔案：
-   保留除了 `updated` 欄位之外的所有 frontmatter
-   將使用者的編輯應用到選定的區塊
-   使用當前日期時間更新 `updated` 欄位

### 4. 檢查對 Epic 的影響

如果 PRD 有關聯的 epic：
-   通知使用者：「此 PRD 有對應的 epic: {epic_name}」
-   詢問：「Epic 可能需要根據 PRD 的變更進行更新。是否要審查 epic？(是/否)」
-   如果是，顯示：「使用 /pm:epic-edit {epic_name} 進行審查」

### 5. 輸出

```
✅ 已更新 PRD: $ARGUMENTS
  已編輯的區塊: {list_of_sections}
  
{如果有 epic}: ⚠️ Epic 可能需要審查: {epic_name}

下一步: /pm:prd-parse $ARGUMENTS 以更新 epic
```

## 重要筆記 (Important Notes)

-   保留原始的創建日期。
-   如果需要，在 frontmatter 中保留版本歷史。
-   遵循 `/rules/frontmatter-operations.md`。