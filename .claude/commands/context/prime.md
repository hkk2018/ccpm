---
allowed-tools: Bash, Read, LS
---

# 載入上下文 (Prime Context)

此命令透過讀取專案上下文文件和理解程式碼庫結構，為新的代理會話載入必要的上下文。

## 飛行前檢查清單

在繼續之前，請完成這些驗證步驟。
不要用飛行前檢查的進度來打擾使用者（例如說「我將不會...」）。只需執行它們然後繼續。

### 1. 上下文可用性檢查
- 運行：`ls -la .claude/context/ 2>/dev/null`
- 如果目錄不存在或為空：
  - 告知使用者：「❌ 找不到上下文。請先運行 /context:create 來建立專案上下文。」
  - 優雅地退出
- 計算可用的上下文檔案數量：`ls -1 .claude/context/*.md 2>/dev/null | wc -l`
- 報告：「📁 發現 {count} 個要載入的上下文檔案」

### 2. 檔案完整性檢查
- 對於找到的每個上下文檔案：
  - 驗證檔案是否可讀：`test -r ".claude/context/{file}" && echo "readable"`
  - 檢查檔案是否有內容：`test -s ".claude/context/{file}" && echo "has content"`
  - 檢查是否存在有效的 frontmatter（應以 `---` 開頭）
- 報告任何問題：
  - 空檔案：「⚠️ {filename} 是空的（跳過）」
  - 不可讀的檔案：「⚠️ 無法讀取 {filename}（權限問題）」
  - 缺少 frontmatter：「⚠️ {filename} 缺少 frontmatter（可能已損壞）」

### 3. 專案狀態檢查
- 運行：`git status --short 2>/dev/null` 以查看當前狀態
- 運行：`git branch --show-current 2>/dev/null` 以獲取當前分支
- 注意是否不在 git 儲存庫中（上下文可能不太完整）

## 指示

### 1. 上下文載入順序

按優先級順序載入上下文檔案，以實現最佳理解：

**優先級 1 - 必要上下文 (最先載入):**
1. `project-overview.md` - 對專案的高層次理解
2. `project-brief.md` - 核心目的和目標
3. `tech-context.md` - 技術棧和依賴項

**優先級 2 - 當前狀態 (其次載入):**
4. `progress.md` - 當前狀態和最近的工作
5. `project-structure.md` - 目錄和檔案組織

**優先級 3 - 深度上下文 (第三載入):**
6. `system-patterns.md` - 架構和設計模式
7. `product-context.md` - 使用者需求和要求
8. `project-style-guide.md` - 編碼慣例
9. `project-vision.md` - 長期方向

### 2. 載入期間的驗證

對於載入的每個檔案：
- 檢查 frontmatter 是否存在並解析：
  - `created` 日期應有效
  - `last_updated` 應大於等於 `created` 日期
  - `version` 應存在
- 如果 frontmatter 無效，則註明但繼續載入內容
- 追蹤哪些檔案成功載入，哪些失敗

### 3. 補充資訊

載入上下文檔案後：
- 運行：`git ls-files --others --exclude-standard | head -20` 以查看未追蹤的檔案
- 如果存在，則讀取 `README.md` 以獲取額外的專案資訊
- 檢查 `.env.example` 或類似檔案以了解環境設定需求

### 4. 錯誤恢復

**如果關鍵檔案缺失：**
- `project-overview.md` 缺失：嘗試從 README.md 理解
- `tech-context.md` 缺失：直接分析 package.json/requirements.txt
- `progress.md` 缺失：檢查最近的 git 提交以了解狀態

**如果上下文不完整：**
- 告知使用者哪些檔案缺失
- 建議運行 `/context:update` 來刷新上下文
- 繼續使用部分上下文，但註明其局限性

### 5. 載入摘要

在載入後提供全面的摘要：

```
🧠 上下文載入成功

📖 已載入的上下文檔案：
  ✅ 必要上下文：{count}/3 個檔案
  ✅ 當前狀態：{count}/2 個檔案
  ✅ 深度上下文：{count}/4 個檔案

🔍 專案理解：
  - 名稱：{project_name}
  - 類型：{project_type}
  - 語言：{primary_language}
  - 狀態：{來自 progress.md 的當前狀態}
  - 分支：{git_branch}

📊 關鍵指標：
  - 最後更新：{most_recent_update}
  - 上下文版本：{version}
  - 已載入檔案：{success_count}/{total_count}

⚠️ 警告：
  {列出任何缺失的檔案或問題}

🎯 準備就緒狀態：
  ✅ 專案上下文已載入
  ✅ 當前狀態已理解
  ✅ 準備好進行開發工作

💡 專案摘要：
  {2-3 句話總結專案是什麼以及當前狀態}
```

### 6. 部分上下文處理

如果某些檔案載入失敗：
- 繼續使用可用的上下文
- 清楚地註明缺失的內容
- 建議補救措施：
  - 「缺少技術上下文 - 運行 /context:create 以重建」
  - 「進度檔案已損壞 - 運行 /context:update 以刷新」

### 7. 性能優化

對於大型上下文：
- 盡可能並行載入檔案
- 顯示進度指示器：「正在載入上下文檔案... {current}/{total}」
- 跳過極大的檔案（>10000 行）並發出警告
- 緩存已解析的 frontmatter 以加快後續載入速度

## 重要筆記

- 在嘗試讀取之前，**始終驗證**檔案
- **按優先級順序載入**以首先獲取必要的上下文
- **優雅地處理缺失的檔案** - 不要完全失敗
- **提供**所載入內容和專案狀態的**清晰摘要**
- **註明**任何可能影響開發工作的**問題**
