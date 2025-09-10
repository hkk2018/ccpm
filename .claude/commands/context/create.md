---
allowed-tools: Bash, Read, Write, LS
---

# 創建初始上下文

此命令透過分析當前專案狀態，在 `.claude/context/` 中創建初始專案上下文文件，以建立全面的基準文件。

## 必要規則

**重要：** 在執行此命令之前，請閱讀並遵守：
- `.claude/rules/datetime.md` - 用於獲取真實的當前日期/時間

## 飛行前檢查清單

在繼續之前，請完成這些驗證步驟。
不要用飛行前檢查的進度來打擾使用者（例如說「我將不會...」）。只需執行它們然後繼續。

### 1. 上下文目錄檢查
- 運行：`ls -la .claude/context/ 2>/dev/null`
- 如果目錄存在且有檔案：
  - 計算現有檔案數量：`ls -1 .claude/context/*.md 2>/dev/null | wc -l`
  - 詢問使用者：「⚠️ 發現 {count} 個現有上下文檔案。是否覆蓋所有上下文？(是/否)」
  - 僅在得到明確的「是」確認後才繼續
  - 如果使用者說「否」，建議：「使用 /context:update 來刷新現有上下文」

### 2. 專案類型檢測
- 檢查專案指標：
  - Node.js: `test -f package.json && echo "檢測到 Node.js 專案"`
  - Python: `test -f requirements.txt || test -f pyproject.toml && echo "檢測到 Python 專案"`
  - Rust: `test -f Cargo.toml && echo "檢測到 Rust 專案"`
  - Go: `test -f go.mod && echo "檢測到 Go 專案"`
- 運行：`git status 2>/dev/null` 以確認這是一個 git 儲存庫
- 如果不是 git 儲存庫，詢問：「⚠️ 這不是一個 git 儲存庫。是否仍要繼續？(是/否)」

### 3. 目錄創建
- 如果 `.claude/` 不存在，則創建它：`mkdir -p .claude/context/`
- 驗證寫入權限：`touch .claude/context/.test && rm .claude/context/.test`
- 如果權限被拒絕，告知使用者：「❌ 無法創建上下文目錄。請檢查權限。」

### 4. 獲取當前日期時間
- 運行：`date -u +"%Y-%m-%dT%H:%M:%SZ"`
- 儲存此值以用於所有上下文檔案的 frontmatter

## 指示

### 1. 分析前驗證
- 確認專案根目錄是否正確（是否存在 .git、package.json 等）
- 檢查可為上下文提供資訊的現有文件（README.md、docs/）
- 如果 README.md 不存在，請向使用者索取專案描述

### 2. 系統性專案分析
按此順序收集資訊：

**專案檢測：**
- 運行：`find . -maxdepth 2 -name 'package.json' -o -name 'requirements.txt' -o -name 'Cargo.toml' -o -name 'go.mod' 2>/dev/null`
- 運行：`git remote -v 2>/dev/null` 以獲取儲存庫資訊
- 運行：`git branch --show-current 2>/dev/null` 以獲取當前分支

**程式碼庫分析：**
- 運行：`find . -type f -name '*.js' -o -name '*.py' -o -name '*.rs' -o -name '*.go' 2>/dev/null | head -20`
- 運行：`ls -la` 以查看根目錄結構
- 如果存在，則讀取 README.md

### 3. 創建帶有 Frontmatter 的上下文檔案

每個上下文檔案**必須**包含帶有真實日期時間的 frontmatter：

```yaml
---
created: [使用來自 date 命令的真實日期時間]
last_updated: [使用來自 date 命令的真實日期時間]
version: 1.0
author: Claude Code PM System
---
```

生成以下初始上下文檔案：
  - `progress.md` - 記錄當前專案狀態、已完成的工作和接下來的步驟
    - 包括：當前分支、最近的提交、未完成的變更
  - `project-structure.md` - 描繪目錄結構和檔案組織
    - 包括：關鍵目錄、檔案命名模式、模組組織
  - `tech-context.md` - 編目當前的依賴項、技術和開發工具
    - 包括：語言版本、框架版本、開發依賴項
  - `system-patterns.md` - 識別現有的架構模式和設計決策
    - 包括：觀察到的設計模式、架構風格、資料流
  - `product-context.md` - 定義產品需求、目標使用者和核心功能
    - 包括：使用者畫像、核心功能、使用案例
  - `project-brief.md` - 建立專案範圍、目標和關鍵指標
    - 包括：它的作用、它為何存在、成功標準
  - `project-overview.md` - 提供功能和能力的高層次摘要
    - 包括：功能列表、當前狀態、整合點
  - `project-vision.md` - 闡明長期願景和戰略方向
    - 包括：未來目標、潛在擴展、戰略優先級
  - `project-style-guide.md` - 記錄編碼標準、慣例和風格偏好
    - 包括：命名慣例、檔案結構模式、註釋風格

### 4. 品質驗證

創建每個檔案後：
- 驗證檔案是否成功創建
- 檢查檔案是否為空（最少 10 行內容）
- 確保 frontmatter 存在且有效
- 驗證 markdown 格式是否正確

### 5. 錯誤處理

**常見問題：**
- **無寫入權限：**「❌ 無法寫入 .claude/context/。請檢查權限。」
- **磁碟空間：**「❌ 上下文檔案的磁碟空間不足。」
- **檔案創建失敗：**「❌ 創建 {filename} 失敗。錯誤：{error}」

如果任何檔案創建失敗：
- 報告哪些檔案已成功創建
- 提供使用部分上下文繼續的選項
- 絕不留下損壞或不完整的檔案

### 6. 創建後摘要

提供全面的摘要：
```
📋 上下文創建完成

📁 在此創建上下文：.claude/context/
✅ 已創建檔案：{count}/9

📊 上下文摘要：
  - 專案類型：{detected_type}
  - 語言：{primary_language}
  - Git 狀態：{clean/changes}
  - 依賴項：{count} 個套件

📝 檔案詳細資訊：
  ✅ progress.md ({lines} 行) - 當前狀態和最近的工作
  ✅ project-structure.md ({lines} 行) - 目錄組織
  [... 列出所有檔案及其行數和簡要描述 ...]

⏰ 創建時間：{timestamp}
🔄 下一步：在新會話中使用 /context:prime 來載入上下文
💡 提示：定期運行 /context:update 以保持上下文最新
```

## 上下文收集命令

使用這些命令來收集專案資訊：
- 目標目錄：`.claude/context/`（如果需要則創建）
- 當前 git 狀態：`git status --short`
- 最近的提交：`git log --oneline -10`
- 專案 README：如果存在則讀取 `README.md`
- 套件檔案：檢查 `package.json`、`requirements.txt`、`Cargo.toml`、`go.mod` 等
- 文件掃描：`find . -type f -name '*.md' -path '*/docs/*' 2>/dev/null | head -10`
- 測試檢測：`find . -type d \( -name 'test' -o -name 'tests' -o -name '__tests__' -o -name 'spec' \) 2>/dev/null | head -5`

## 重要筆記

- **始終使用**來自系統時鐘的**真實日期時間**，絕不使用預留位置
- 在覆蓋現有上下文之前**請求確認**
- **驗證每個檔案**是否成功創建
- **提供**所創建內容的**詳細摘要**
- 以具體指導**優雅地處理錯誤**

$ARGUMENTS
