# 命令的標準模式

此檔案定義了所有命令應遵循的通用模式，以保持一致性和簡潔性。

## 核心原則 (Core Principles)

1.  **快速失敗 (Fail Fast)** - 檢查關鍵的先決條件，然後繼續。
2.  **信任系統 (Trust the System)** - 不要過度驗證那些很少失敗的東西。
3.  **清晰的錯誤 (Clear Errors)** - 當某事失敗時，準確說明是什麼以及如何修復它。
4.  **最簡輸出 (Minimal Output)** - 顯示重要的內容，省略裝飾。

## 標準驗證 (Standard Validations)

### 最小化的飛行前檢查 (Minimal Preflight)
只檢查絕對必要的東西：
```markdown
## 快速檢查 (Quick Check)
1. 如果命令需要特定的目錄/檔案：
   - 檢查它是否存在：`test -f {file} || echo "❌ {file} 未找到"`
   - 如果缺失，告訴使用者修復它的確切命令。
2. 如果命令需要 GitHub：
   - 假設 `gh` 已通過身份驗證（通常是這樣）。
   - 僅在實際失敗時檢查。
```

### 日期時間處理 (DateTime Handling)
```markdown
獲取當前日期時間：`date -u +"%Y-%m-%dT%H:%M:%SZ"`
```
不要重複完整的說明——只需引用一次 `/rules/datetime.md`。

### 錯誤訊息 (Error Messages)
保持簡短且可操作：
```markdown
❌ {什麼失敗了}: {確切的解決方案}
範例: "❌ Epic 未找到：請運行 /pm:prd-parse feature-name"
```

## 標準輸出格式 (Standard Output Formats)

### 成功輸出 (Success Output)
```markdown
✅ {動作} 完成
  - {關鍵結果 1}
  - {關鍵結果 2}
下一步: {單一建議的動作}
```

### 列表輸出 (List Output)
```markdown
找到 {數量} 個 {項目}:
- {項目 1}: {關鍵細節}
- {項目 2}: {關鍵細節}
```

### 進度輸出 (Progress Output)
```markdown
{動作}... {目前}/{總數}
```

## 檔案操作 (File Operations)

### 檢查並創建 (Check and Create)
```markdown
# 不要請求許可，直接創建需要的東西
mkdir -p .claude/{directory} 2>/dev/null
```

### 帶後備方案的讀取 (Read with Fallback)
```markdown
# 嘗試讀取，如果缺失則繼續
if [ -f {file} ]; then
  # 讀取並使用檔案
else
  # 使用合理的預設值
fi
```

## GitHub 操作 (GitHub Operations)

### 信任 gh CLI (Trust gh CLI)
```markdown
# 不要預先檢查認證，直接嘗試操作
gh {command} || echo "❌ GitHub CLI 失敗。請運行：gh auth login"
```

### 簡單的 Issue 操作 (Simple Issue Operations)
```markdown
# 一次呼叫獲取你需要的東西
gh issue view {number} --json state,title,body
```

## 應避免的常見模式 (Common Patterns to Avoid)

### 不要：過度驗證 (DON'T: Over-validate)
```markdown
# 不好 - 太多檢查
1. 檢查目錄是否存在
2. 檢查權限
3. 檢查 git 狀態
4. 檢查 GitHub 認證
5. 檢查速率限制
6. 驗證每個欄位
```

### 要做：檢查必需品 (DO: Check essentials)
```markdown
# 好 - 只檢查需要的
1. 檢查目標是否存在
2. 嘗試操作
3. 清晰地處理失敗
```

### 不要：詳細輸出 (DON'T: Verbose output)
```markdown
# 不好 - 太多資訊
🎯 開始操作...
📋 正在驗證先決條件...
✅ 步驟 1 完成
✅ 步驟 2 完成
📊 統計數據: ...
💡 提示: ...
```

### 要做：簡潔輸出 (DO: Concise output)
```markdown
# 好 - 只有結果
✅ 完成：已創建 3 個檔案
失敗：auth.test.js (語法錯誤 - 第 42 行)
```

### 不要：問太多問題 (DON'T: Ask too many questions)
```markdown
# 不好 - 太多互動
"繼續嗎？ (是/否)"
"覆蓋嗎？ (是/否)"
"你確定嗎？ (是/否)"
```

### 要做：智慧型預設 (DO: Smart defaults)
```markdown
# 好 - 使用合理的預設值繼續
# 僅在具有破壞性或模棱兩可時才詢問
"這將刪除 10 個檔案。繼續嗎？ (是/否)"
```

## 快速參考 (Quick Reference)

### 僅限基本工具 (Essential Tools Only)
-   讀取/列表操作：`Read, LS`
-   檔案創建：`Read, Write, LS`
-   GitHub 操作：新增 `Bash`
-   複雜分析：新增 `Task` (謹慎使用)

### 狀態指示器 (Status Indicators)
-   ✅ 成功 (謹慎使用)
-   ❌ 錯誤 (總是附帶解決方案)
-   ⚠️ 警告 (僅在需要採取行動時)
-   正常輸出無表情符號

### 退出策略 (Exit Strategies)
-   成功：簡短確認
-   失敗：清晰的錯誤 + 確切的修復方法
-   部分成功：顯示哪些成功了，哪些失敗了

## 請記住 (Remember)

**簡單不等於簡陋** - 我們仍然妥善處理錯誤，只是不試圖預防每一個可能的邊界情況。我們相信：
-   檔案系統通常能正常工作。
-   GitHub CLI 通常已通過身份驗證。
-   Git 儲存庫通常是有效的。
-   使用者知道他們在做什麼。

專注於理想路徑 (happy path)，並在出錯時優雅地失敗。