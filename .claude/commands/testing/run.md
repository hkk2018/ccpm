---
allowed-tools: Bash, Read, Write, LS, Task
---

# 運行測試 (Run Tests)

使用已設定的測試執行代理來執行測試。

## 用法 (Usage)
```
/testing:run [test_target]
```

其中 `test_target` 可以是：
-   空（運行所有測試）
-   測試檔案路徑
-   測試模式
-   測試套件名稱

## 快速檢查 (Quick Check)

```bash
# 檢查測試是否已設定
test -f .claude/testing-config.md || echo "❌ 測試尚未設定。請先運行 /testing:prime"
```

如果提供了測試目標，驗證其是否存在：
```bash
# 對於檔案目標
test -f "$ARGUMENTS" || echo "⚠️ 找不到測試檔案：$ARGUMENTS"
```

## 指示 (Instructions)

### 1. 決定測試命令

根據 testing-config.md 和目標：
-   無參數 → 從設定檔運行完整測試套件
-   檔案路徑 → 運行特定的測試檔案
-   模式 → 運行匹配模式的測試

### 2. 執行測試

使用 `.claude/agents/test-runner.md` 中的測試執行代理：

```markdown
為 $ARGUMENTS 執行測試 (如果為空則為 "all")

要求：
-   以詳細輸出運行以進行除錯
-   不使用模擬 - 使用真實服務
-   捕獲包括堆疊追蹤在內的完整輸出
-   如果測試失敗，在假設程式碼問題之前檢查測試結構
```

### 3. 監控執行

-   顯示測試進度
-   捕獲 stdout 和 stderr
-   記錄執行時間

### 4. 報告結果

**成功：**
```
✅ 所有測試通過 ({count} 個測試，耗時 {time}s)
```

**失敗：**
```
❌ 測試失敗：{total_count} 中的 {failed_count} 個

{test_name} - {file}:{line}
  錯誤 (Error): {error_message}
  可能原因 (Likely): {測試問題 | 程式碼問題}
  修復建議 (Fix): {suggestion}

運行更詳細的測試：/testing:run {specific_test}
```

**混合：**
```
測試完成：{passed} 個通過，{failed} 個失敗，{skipped} 個跳過

失敗：
- {test_1}: {簡要原因}
- {test_2}: {簡要原因}
```

### 5. 清理

```bash
# 終止任何懸掛的測試進程
pkill -f "jest|mocha|pytest" 2>/dev/null || true
```

## 錯誤處理

-   測試命令失敗 → "❌ 測試執行失敗：{error}。請檢查測試框架是否已安裝。"
-   超時 → 終止進程並報告："❌ 測試在 {time}s 後超時"
-   找不到測試 → "❌ 找不到匹配的測試：$ARGUMENTS"

## 重要筆記 (Important Notes)

-   始終使用測試執行代理進行分析
-   不模擬 - 僅使用真實服務
-   如果發生失敗，檢查測試結構
-   將輸出重點放在失敗上