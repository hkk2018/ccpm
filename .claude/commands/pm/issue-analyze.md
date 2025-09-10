---
allowed-tools: Bash, Read, Write, LS
---

# 分析 Issue (Issue Analyze)

分析一個 issue 以識別並行工作流，從而實現最大效率。

## 用法 (Usage)
```
/pm:issue-analyze <issue_number>
```

## 快速檢查 (Quick Check)

1. **尋找本地任務檔案：**
   -   首先檢查 `.claude/epics/*/$ARGUMENTS.md` 是否存在（新命名慣例）
   -   如果找不到，則在 frontmatter 中搜索包含 `github:.*issues/$ARGUMENTS` 的檔案（舊命名方式）
   -   如果找不到：「❌ 找不到 issue #$ARGUMENTS 的本地任務。請先運行 /pm:import」

2. **檢查現有的分析：**
   ```bash
   test -f .claude/epics/*/$ARGUMENTS-analysis.md && echo "⚠️ 分析已存在。是否覆蓋？(是/否)"
   ```

## 指示 (Instructions)

### 1. 讀取 Issue 上下文

從 GitHub 獲取 issue 詳細資訊：
```bash
gh issue view $ARGUMENTS --json title,body,labels
```

讀取本地任務檔案以了解：
-   技術需求
-   驗收標準
-   依賴項
-   工時估計

### 2. 識別並行工作流

分析 issue 以識別可以並行運行的獨立工作：

**常見模式：**
-   **資料庫層 (Database Layer)**：結構、遷移、模型
-   **服務層 (Service Layer)**：業務邏輯、資料存取
-   **API 層 (API Layer)**：端點、驗證、中介軟體
-   **UI 層 (UI Layer)**：元件、頁面、樣式
-   **測試層 (Test Layer)**：單元測試、整合測試
-   **文件 (Documentation)**：API 文件、README 更新

**關鍵問題：**
-   將會創建/修改哪些檔案？
-   哪些變更可以獨立進行？
-   變更之間有什麼依賴關係？
-   哪裡可能會發生衝突？

### 3. 創建分析檔案

獲取當前日期時間：`date -u +"%Y-%m-%dT%H:%M:%SZ"`

創建 `.claude/epics/{epic_name}/$ARGUMENTS-analysis.md`：

```markdown
---
issue: $ARGUMENTS
title: {issue_title}
analyzed: {current_datetime}
estimated_hours: {total_hours}
parallelization_factor: {1.0-5.0}
---

# 並行工作分析：Issue #$ARGUMENTS

## 概覽 (Overview)
{需要做什麼的簡要描述}

## 並行工作流 (Parallel Streams)

### 工作流 A: {Stream Name}
**範圍 (Scope)**: {此工作流處理的內容}
**檔案 (Files)**:
- {file_pattern_1}
- {file_pattern_2}
**代理類型 (Agent Type)**: {backend|frontend|fullstack|database}-specialist
**可開始時間 (Can Start)**: 立即
**估計工時 (Estimated Hours)**: {hours}
**依賴項 (Dependencies)**: 無

### 工作流 B: {Stream Name}
**範圍 (Scope)**: {此工作流處理的內容}
**檔案 (Files)**:
- {file_pattern_1}
- {file_pattern_2}
**代理類型 (Agent Type)**: {agent_type}
**可開始時間 (Can Start)**: 立即
**估計工時 (Estimated Hours)**: {hours}
**依賴項 (Dependencies)**: 無

### 工作流 C: {Stream Name}
**範圍 (Scope)**: {此工作流處理的內容}
**檔案 (Files)**:
- {file_pattern_1}
**代理類型 (Agent Type)**: {agent_type}
**可開始時間 (Can Start)**: 工作流 A 完成後
**估計工時 (Estimated Hours)**: {hours}
**依賴項 (Dependencies)**: 工作流 A

## 協調點 (Coordination Points)

### 共享檔案 (Shared Files)
{列出多個工作流需要修改的任何檔案}：
- `src/types/index.ts` - 工作流 A & B (協調類型更新)
- `package.json` - 工作流 B (新增依賴項)

### 循序需求 (Sequential Requirements)
{列出必須按順序發生的事情}：
1. 資料庫結構先於 API 端點
2. API 類型先於 UI 元件
3. 核心邏輯先於測試

## 衝突風險評估 (Conflict Risk Assessment)
-   **低風險 (Low Risk)**：工作流在不同的目錄中工作
-   **中風險 (Medium Risk)**：一些共享的類型檔案，可透過協調管理
-   **高風險 (High Risk)**：多個工作流修改相同的核心檔案

## 並行化策略 (Parallelization Strategy)

**建議方法 (Recommended Approach)**: {循序|並行|混合}

{如果並行}: 同時啟動工作流 A, B。當 A 完成時啟動 C。
{如果循序}: 完成工作流 A，然後是 B，然後是 C。
{如果混合}: 一起啟動 A & B，C 依賴於 A，D 依賴於 B & C。

## 預期時間軸 (Expected Timeline)

使用並行執行：
-   實際耗時 (Wall time): {max_stream_hours} 小時
-   總工作量 (Total work): {sum_all_hours} 小時
-   效率增益 (Efficiency gain): {percentage}%

不使用並行執行：
-   實際耗時 (Wall time): {sum_all_hours} 小時

## 筆記 (Notes)
{任何特殊考量、警告或建議}
```

### 4. 驗證分析

確保：
-   所有主要工作都由工作流涵蓋
-   檔案模式沒有不必要的重疊
-   依賴關係是合乎邏輯的
-   代理類型與工作類型匹配
-   時間估計是合理的

### 5. 輸出

```
✅ Issue #$ARGUMENTS 分析完成

已識別 {count} 個並行工作流：
  工作流 A: {name} ({hours}h)
  工作流 B: {name} ({hours}h)
  工作流 C: {name} ({hours}h)
  
並行化潛力：{factor}x 加速
  循序時間：{total}h
  並行時間：{reduced}h

有衝突風險的檔案：
  {列出任何共享檔案}

下一步：使用 /pm:issue-start $ARGUMENTS 開始工作
```

## 重要筆記 (Important Notes)

-   分析僅在本地進行 - 不會同步到 GitHub。
-   專注於實際的並行化，而非理論上的最大值。
-   在分配工作流時考慮代理的專業知識。
-   在估算中考慮協調的開銷。
-   優先選擇清晰的分離，而不是最大程度的並行化。