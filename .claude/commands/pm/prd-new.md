---
allowed-tools: Bash, Read, Write, LS
---

# 新建 PRD (PRD New)

為新的產品需求文件啟動腦力激盪。

## 用法 (Usage)
```
/pm:prd-new <feature_name>
```

## 必要規則

**重要：** 在執行此命令之前，請閱讀並遵守：
- `.claude/rules/datetime.md` - 用於獲取真實的當前日期/時間

## 飛行前檢查清單

在繼續之前，請完成這些驗證步驟。
不要用飛行前檢查的進度來打擾使用者（例如說「我將不會...」）。只需執行它們然後繼續。

### 輸入驗證
1. **驗證功能名稱格式：**
   -   必須只包含小寫字母、數字和連字號
   -   必須以字母開頭
   -   不允許空格或特殊字元
   -   如果無效，告知使用者：「❌ 功能名稱必須是 kebab-case（僅限小寫字母、數字、連字號）。範例：user-auth、payment-v2、notification-system」

2. **檢查現有的 PRD：**
   -   檢查 `.claude/prds/$ARGUMENTS.md` 是否已存在
   -   如果存在，詢問使用者：「⚠️ PRD '$ARGUMENTS' 已存在。您要覆蓋它嗎？(是/否)」
   -   僅在得到明確的「是」確認後才繼續
   -   如果使用者說「否」，建議：「使用不同的名稱或運行 /pm:prd-parse $ARGUMENTS 從現有的 PRD 創建一個 epic」

3. **驗證目錄結構：**
   -   檢查 `.claude/prds/` 目錄是否存在
   -   如果不存在，先創建它
   -   如果無法創建，告知使用者：「❌ 無法創建 PRD 目錄。請手動創建：.claude/prds/」

## 指示 (Instructions)

您是一位產品經理，正在為 **$ARGUMENTS** 創建一份全面的產品需求文件 (PRD)。

遵循此結構化方法：

### 1. 探索與情境 (Discovery & Context)
-   詢問關於功能/產品 "$ARGUMENTS" 的澄清問題
-   理解正在解決的問題
-   識別目標使用者和使用案例
-   收集限制和需求

### 2. PRD 結構
創建一份包含以下區塊的綜合 PRD：

#### 執行摘要 (Executive Summary)
-   簡要概述和價值主張

#### 問題陳述 (Problem Statement)
-   我們正在解決什麼問題？
-   為什麼現在這很重要？

#### 使用者故事 (User Stories)
-   主要使用者畫像
-   詳細的使用者旅程
-   正在解決的痛點

#### 需求 (Requirements)
**功能性需求**
-   核心功能和能力
-   使用者互動和流程

**非功能性需求**
-   性能預期
-   安全考量
-   可擴展性需求

#### 成功標準 (Success Criteria)
-   可衡量的成果
-   關鍵指標和 KPI

#### 限制與假設 (Constraints & Assumptions)
-   技術限制
-   時間軸限制
-   資源限制

#### 範圍外 (Out of Scope)
-   我們明確**不**構建的內容

#### 依賴項 (Dependencies)
-   外部依賴
-   內部團隊依賴

### 3. 帶有 Frontmatter 的檔案格式
將完成的 PRD 儲存到：`.claude/prds/$ARGUMENTS.md`，並使用此確切結構：

```markdown
---
name: $ARGUMENTS
description: [此 PRD 的簡短單行描述]
status: backlog
created: [當前 ISO 日期/時間]
---

# PRD: $ARGUMENTS

## Executive Summary
[內容...]

## Problem Statement
[內容...]

[繼續所有區塊...]
```

### 4. Frontmatter 指南
- **name**: 使用確切的功能名稱 (與 $ARGUMENTS 相同)
- **description**: 撰寫此 PRD 涵蓋內容的簡潔單行摘要
- **status**: 新 PRD 始終以 "backlog" 開始
- **created**: 運行 `date -u +"%Y-%m-%dT%H:%M:%SZ"` 獲取真實的當前日期時間
  - 絕不使用預留位置文字
  - 必須是 ISO 8601 格式的實際系統時間

### 5. 品質檢查

儲存 PRD 之前，請驗證：
- [ ] 所有區塊都已完成 (沒有預留位置文字)
- [ ] 使用者故事包含驗收標準
- [ ] 成功標準是可衡量的
- [ ] 依賴項已明確識別
- [ ] 範圍外的項目已明確列出

### 6. 創建後

成功創建 PRD 後：
1. 確認：「✅ PRD 已創建：.claude/prds/$ARGUMENTS.md」
2. 顯示捕獲內容的簡要摘要
3. 建議下一步：「準備好創建實作 epic 了嗎？運行：/pm:prd-parse $ARGUMENTS」

## 錯誤恢復

如果任何步驟失敗：
-   清楚地解釋出了什麼問題
-   提供具體的步驟來修復問題
-   絕不留下部分或損壞的檔案

在撰寫 PRD 之前進行徹底的腦力激盪。提出問題，探索邊界情況，並確保全面涵蓋 "$ARGUMENTS" 的功能需求。
