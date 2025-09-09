# 測試執行規則

適用於所有測試命令的標準測試運行模式。

## 核心原則 (Core Principles)

1.  **始終使用 `test-runner` 代理**，定義於 `.claude/agents/test-runner.md`。
2.  **禁止模擬 (No mocking)** - 使用真實服務以獲得準確結果。
3.  **詳細輸出 (Verbose output)** - 捕獲所有資訊以便除錯。
4.  **先檢查測試結構** - 在假設程式碼有錯誤之前。

## 執行模式 (Execution Pattern)

```markdown
為 {target} 執行測試

要求:
- 以詳細模式運行輸出
- 不使用模擬服務
- 捕獲完整的堆疊追蹤
- 如果發生失敗，分析測試結構
```

## 輸出焦點 (Output Focus)

### 成功 (Success)
保持簡單：
```
✅ 所有測試通過 ({count} 個測試，耗時 {time}s)
```

### 失敗 (Failure)
專注於失敗的部分：
```
❌ 測試失敗: {count}

{test_name} - {file}:{line}
  錯誤 (Error): {message}
  修復建議 (Fix): {suggestion}
```

## 常見問題 (Common Issues)

-   找不到測試 → 檢查檔案路徑。
-   超時 → 終止進程，報告未完成。
-   缺少框架 → 安裝依賴項。

## 清理 (Cleanup)

測試後務必進行清理：
```bash
pkill -f "jest|mocha|pytest" 2>/dev/null || true
```

## 重要筆記 (Important Notes)

-   不要並行化測試（以避免衝突）。
-   讓每個測試完全完成。
-   報告失敗時附帶可行的修復建議。
-   將輸出重點放在失敗上，而不是成功上。