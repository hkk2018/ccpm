---
allowed-tools: Bash, Read, Write, LS
---

# 解析 PRD (PRD Parse)

將 PRD 轉換為技術實作的 epic。

## 用法 (Usage)
```
/pm:prd-parse <feature_name>
```

## 必要規則

**重要：** 在執行此命令之前，請閱讀並遵守：
- `.claude/rules/datetime.md` - 用於獲取真實的當前日期/時間

## 飛行前檢查清單

在繼續之前，請完成這些驗證步驟。
不要用飛行前檢查的進度來打擾使用者（例如說「我將不會...」）。只需執行它們然後繼續。

### 驗證步驟
1. **驗證 <feature_name> 是否已作為參數提供：**
   - 如果沒有，告知使用者：「❌ 未提供 <feature_name> 作為參數。請運行：/pm:prd-parse <feature_name>」
   - 如果未提供 <feature_name>，則停止執行

2. **驗證 PRD 是否存在：**
   - 檢查 `.claude/prds/$ARGUMENTS.md` 是否存在
   - 如果找不到，告知使用者：「❌ 找不到 PRD：$ARGUMENTS。請先使用 /pm:prd-new $ARGUMENTS 創建它」
   - 如果 PRD 不存在，則停止執行

3. **驗證 PRD 的 frontmatter：**
   - 驗證 PRD 是否有包含 name, description, status, created 的有效 frontmatter
   - 如果 frontmatter 無效或缺失，告知使用者：「❌ 無效的 PRD frontmatter。請檢查：.claude/prds/$ARGUMENTS.md」
   - 顯示缺失或無效的內容

4. **檢查現有的 epic：**
   - 檢查 `.claude/epics/$ARGUMENTS/epic.md` 是否已存在
   - 如果存在，詢問使用者：「⚠️ Epic '$ARGUMENTS' 已存在。是否覆蓋？(是/否)」
   - 僅在得到明確的「是」確認後才繼續
   - 如果使用者說「否」，建議：「使用 /pm:epic-show $ARGUMENTS 查看現有的 epic」

5. **驗證目錄權限：**
   - 確保 `.claude/epics/` 目錄存在或可以被創建
   - 如果無法創建，告知使用者：「❌ 無法創建 epic 目錄。請檢查權限。」

## 指示 (Instructions)

您是一位技術主管，正在為 **$ARGUMENTS** 將一份產品需求文件轉換為詳細的實作 epic。

### 1. 讀取 PRD
- 從 `.claude/prds/$ARGUMENTS.md` 載入 PRD
- 分析所有需求和限制
- 理解使用者故事和成功標準
- 從 frontmatter 提取 PRD 描述

### 2. 技術分析
- 識別所需的架構決策
- 決定技術棧和方法
- 將功能性需求映射到技術元件
- 識別整合點和依賴項

### 3. 帶有 Frontmatter 的檔案格式
在 `.claude/epics/$ARGUMENTS/epic.md` 創建 epic 檔案，並使用此確切結構：

```markdown
---
name: $ARGUMENTS
status: backlog
created: [當前 ISO 日期/時間]
progress: 0%
prd: .claude/prds/$ARGUMENTS.md
github: [同步到 GitHub 時將會更新]
---

# Epic: $ARGUMENTS

## 概覽 (Overview)
實作方法的簡要技術摘要

## 架構決策 (Architecture Decisions)
- 關鍵技術決策及其理由
- 技術選擇
- 要使用的設計模式

## 技術方法 (Technical Approach)
### 前端元件 (Frontend Components)
- 所需的 UI 元件
- 狀態管理方法
- 使用者互動模式

### 後端服務 (Backend Services)
- 所需的 API 端點
- 資料模型和結構
- 業務邏輯元件

### 基礎設施 (Infrastructure)
- 部署考量
- 擴展需求
- 監控和可觀察性

## 實作策略 (Implementation Strategy)
- 開發階段
- 風險緩解
- 測試方法

## 任務分解預覽 (Task Breakdown Preview)
將創建的高層次任務類別：
- [ ] 類別 1: 描述
- [ ] 類別 2: 描述
- [ ] 等等...

## 依賴項 (Dependencies)
- 外部服務依賴
- 內部團隊依賴
- 先決工作

## 成功標準 (技術) (Success Criteria (Technical))
- 性能基準
- 品質門檻
- 驗收標準

## 工時估計 (Estimated Effort)
- 整體時間軸估計
- 資源需求
- 關鍵路徑項目
```

### 4. Frontmatter 指南
- **name**: 使用確切的功能名稱 (與 $ARGUMENTS 相同)
- **status**: 新 epics 始終以 "backlog" 開始
- **created**: 運行 `date -u +"%Y-%m-%dT%H:%M:%SZ"` 獲取真實的當前日期時間
- **progress**: 新 epics 始終以 "0%" 開始
- **prd**: 引用來源 PRD 檔案路徑
- **github**: 留下預留位置文字 - 將在同步期間更新

### 5. 輸出位置
如果目錄結構不存在，則創建它：
- `.claude/epics/$ARGUMENTS/` (目錄)
- `.claude/epics/$ARGUMENTS/epic.md` (epic 檔案)

### 6. 品質驗證

儲存 epic 之前，請驗證：
- [ ] 所有 PRD 需求都在技術方法中得到解決
- [ ] 任務分解類別涵蓋所有實作領域
- [ ] 依賴關係在技術上是準確的
- [ ] 工時估計是現實的
- [ ] 架構決策是合理的

### 7. 創建後

成功創建 epic 後：
1. 確認：「✅ Epic 已創建：.claude/epics/$ARGUMENTS/epic.md」
2. 顯示摘要：
   - 識別的任務類別數量
   - 關鍵架構決策
   - 估計工時
3. 建議下一步：「準備好分解成任務了嗎？運行：/pm:epic-decompose $ARGUMENTS」

## 錯誤恢復

如果任何步驟失敗：
-   清楚地解釋出了什麼問題
-   如果 PRD 不完整，列出具體缺失的區塊
-   如果技術方法不清楚，識別需要澄清的內容
-   絕不創建資訊不完整的 epic

專注於為 "$ARGUMENTS" 創建一個技術上合理、解決所有 PRD 需求，同時又實用且可實現的實作計畫。

## 重要：
- 目標是盡可能少的任務，並將總任務數限制在 10 個或以下。
- 創建 epic 時，找出簡化和改進它的方法。盡可能利用現有功能，而不是創建更多程式碼。
