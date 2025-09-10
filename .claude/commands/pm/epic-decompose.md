---
allowed-tools: Bash, Read, Write, LS, Task
---

# 分解 Epic (Epic Decompose)

將 epic 分解為具體、可操作的任務。

## 用法 (Usage)
```
/pm:epic-decompose <feature_name>
```

## 必要規則

**重要：** 在執行此命令之前，請閱讀並遵守：
- `.claude/rules/datetime.md` - 用於獲取真實的當前日期/時間

## 飛行前檢查清單

在繼續之前，請完成這些驗證步驟。
不要用飛行前檢查的進度來打擾使用者（例如說「我將不會...」）。只需執行它們然後繼續。

1. **驗證 epic 是否存在：**
   - 檢查 `.claude/epics/$ARGUMENTS/epic.md` 是否存在
   - 如果找不到，告知使用者：「❌ Epic 未找到：$ARGUMENTS。請先使用 /pm:prd-parse $ARGUMENTS 創建它」
   - 如果 epic 不存在，則停止執行

2. **檢查現有任務：**
   - 檢查 `.claude/epics/$ARGUMENTS/` 中是否已存在任何編號的任務檔案（001.md, 002.md 等）
   - 如果任務已存在，列出它們並詢問：「⚠️ 發現 {count} 個現有任務。是否刪除並重新創建所有任務？(是/否)」
   - 僅在得到明確的「是」確認後才繼續
   - 如果使用者說「否」，建議：「使用 /pm:epic-show $ARGUMENTS 查看現有任務」

3. **驗證 epic 的 frontmatter：**
   - 驗證 epic 是否有包含 name, status, created, prd 的有效 frontmatter
   - 如果無效，告知使用者：「❌ 無效的 epic frontmatter。請檢查：.claude/epics/$ARGUMENTS/epic.md」

4. **檢查 epic 狀態：**
   - 如果 epic 狀態已為 "completed"，警告使用者：「⚠️ Epic 已標記為完成。您確定要再次分解它嗎？」

## 指示 (Instructions)

您正在為 **$ARGUMENTS** 將一個 epic 分解為具體、可操作的任務。

### 1. 讀取 Epic
- 從 `.claude/epics/$ARGUMENTS/epic.md` 載入 epic
- 理解技術方法和需求
- 審查任務分解預覽

### 2. 分析以進行並行創建

確定任務是否可以並行創建：
- 如果任務大多是獨立的：使用 Task 代理並行創建
- 如果任務有複雜的依賴關係：循序創建
- 為獲得最佳結果：將獨立的任務分組以進行並行創建

### 3. 並行創建任務 (如果可能)

如果任務可以並行創建，則生成子代理：

```yaml
Task:
  description: "創建任務檔案批次 {X}"
  subagent_type: "general-purpose"
  prompt: |
    為 epic 創建任務檔案：$ARGUMENTS

    要創建的任務：
    - {此批次的 3-4 個任務列表}

    對於每個任務：
    1. 創建檔案：.claude/epics/$ARGUMENTS/{number}.md
    2. 使用帶有 frontmatter 和所有區塊的確切格式
    3. 遵循 epic 中的任務分解
    4. 適當地設定 parallel/depends_on 欄位
    5. 循序編號 (001.md, 002.md 等)

    返回：已創建的檔案列表
```

### 4. 帶有 Frontmatter 的任務檔案格式
對於每個任務，使用此確切結構創建一個檔案：

```markdown
---
name: [任務標題]
status: open
created: [當前 ISO 日期/時間]
updated: [當前 ISO 日期/時間]
github: [同步到 GitHub 時將會更新]
depends_on: []  # 此任務依賴的任務編號列表，例如 [001, 002]
parallel: true  # 此任務是否可以與其他任務並行運行？
conflicts_with: []  # 修改相同檔案的任務，例如 [003, 004]
---

# 任務：[任務標題]

## 描述 (Description)
清晰、簡潔地描述需要做什麼

## 驗收標準 (Acceptance Criteria)
- [ ] 具體標準 1
- [ ] 具體標準 2
- [ ] 具體標準 3

## 技術細節 (Technical Details)
- 實作方法
- 關鍵考量
- 受影響的程式碼位置/檔案

## 依賴項 (Dependencies)
- [ ] 任務/Issue 依賴
- [ ] 外部依賴

## 工時估計 (Effort Estimate)
- 大小：XS/S/M/L/XL
- 小時：估計小時數
- 並行：是/否 (可與其他任務並行運行)

## 完成的定義 (Definition of Done)
- [ ] 程式碼已實作
- [ ] 測試已編寫並通過
- [ ] 文件已更新
- [ ] 程式碼已審查
- [ ] 已部署到預備環境
```

### 3. 任務命名慣例
將任務另存為：`.claude/epics/$ARGUMENTS/{task_number}.md`
- 使用循序編號：001.md, 002.md 等
- 保持任務標題簡短但具描述性

### 4. Frontmatter 指南
- **name**: 使用具描述性的任務標題 (不含 "Task:" 前綴)
- **status**: 新任務始終以 "open" 開始
- **created**: 運行 `date -u +"%Y-%m-%dT%H:%M:%SZ"` 獲取真實的當前日期時間
- **updated**: 對於新任務，使用與 created 相同的真實日期時間
- **github**: 留下預留位置文字 - 將在同步期間更新
- **depends_on**: 列出在此任務開始前必須完成的任務編號 (例如, [001, 002])
- **parallel**: 如果此任務可以與其他任務無衝突地同時運行，則設為 true
- **conflicts_with**: 列出修改相同檔案的任務編號 (有助於協調)

### 5. 要考慮的任務類型
- **設定任務**: 環境、依賴項、腳手架
- **資料任務**: 模型、結構、遷移
- **API 任務**: 端點、服務、整合
- **UI 任務**: 元件、頁面、樣式
- **測試任務**: 單元測試、整合測試
- **文件任務**: README、API 文件
- **部署任務**: CI/CD、基礎設施

### 6. 並行化
如果任務可以無衝突地同時進行，則用 `parallel: true` 標記它們。

### 7. 執行策略

根據任務數量和複雜性選擇：

**小型 Epic (< 5 個任務)**: 為簡單起見，循序創建

**中型 Epic (5-10 個任務)**:
- 分成 2-3 個批次
- 為每個批次生成代理
- 整合結果

**大型 Epic (> 10 個任務)**:
- 首先分析依賴關係
- 將獨立的任務分組
- 啟動並行代理 (最多 5 個並發)
- 在先決條件完成後創建依賴的任務

並行執行的範例：
```markdown
正在為並行任務創建生成 3 個代理：
- 代理 1：創建任務 001-003 (資料庫層)
- 代理 2：創建任務 004-006 (API 層)
- 代理 3：創建任務 007-009 (UI 層)
```

### 8. 任務依賴驗證

創建帶有依賴關係的任務時：
- 確保引用的依賴項存在 (例如，如果任務 003 依賴於任務 002，驗證 002 已被創建)
- 檢查循環依賴 (任務 A → 任務 B → 任務 A)
- 如果發現依賴問題，發出警告但繼續：「⚠️ 任務依賴警告：{details}」

### 9. 用任務摘要更新 Epic
創建所有任務後，通過添加此部分來更新 epic 檔案：
```markdown
## 已創建的任務
- [ ] 001.md - {任務標題} (並行: 是/否)
- [ ] 002.md - {任務標題} (並行: 是/否)
- etc.

總任務數：{count}
並行任務數：{parallel_count}
循序任務數：{sequential_count}
估計總工時：{sum of hours}
```

如果需要，也更新 epic 的 frontmatter 進度 (在任務實際開始前仍為 0%)。

### 9. 品質驗證

在最終確定任務之前，驗證：
- [ ] 所有任務都有明確的驗收標準
- [ ] 任務大小合理 (每個 1-3 天)
- [ ] 依賴關係合乎邏輯且可實現
- [ ] 並行任務彼此不衝突
- [ ] 合併的任務涵蓋所有 epic 需求

### 10. 分解後續

成功創建任務後：
1. 確認：「✅ 已為 epic 創建 {count} 個任務：$ARGUMENTS」
2. 顯示摘要：
   - 創建的總任務數
   - 並行與循序的細分
   - 總估計工時
3. 建議下一步：「準備好同步到 GitHub 了嗎？運行：/pm:epic-sync $ARGUMENTS」

## 錯誤恢復

如果任何步驟失敗：
- 如果任務創建部分完成，列出已創建的任務
- 提供清理部分任務的選項
- 絕不讓 epic 處於不一致的狀態

目標是使每個任務都能在 1-3 天內完成。為 "$ARGUMENTS" epic 將較大的任務分解為更小、更易於管理的部分。
