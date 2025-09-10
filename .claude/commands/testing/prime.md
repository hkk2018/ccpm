---
allowed-tools: Bash, Read, Write, LS
---

# 準備測試環境 (Prime Testing Environment)

此命令透過偵測測試框架、驗證依賴項以及設定測試執行代理，來準備測試環境以實現最佳的測試執行。

## 飛行前檢查清單

在繼續之前，請完成這些驗證步驟。
不要用飛行前檢查的進度來打擾使用者（例如說「我將不會...」）。只需執行它們然後繼續。

### 1. 測試框架偵測

**JavaScript/Node.js:**
-   檢查 package.json 中的測試腳本：`grep -E '"test"|"spec"|"jest"|"mocha"' package.json 2>/dev/null`
-   尋找測試設定檔：`ls -la jest.config.* mocha.opts .mocharc.* 2>/dev/null`
-   檢查測試目錄：`find . -type d \( -name "test" -o -name "tests" -o -name "__tests__" -o -name "spec" \) -maxdepth 3 2>/dev/null`

**Python:**
-   檢查 pytest：`find . -name "pytest.ini" -o -name "conftest.py" -o -name "setup.cfg" 2>/dev/null | head -5`
-   檢查 unittest：`find . -path "*/test*.py" -o -path "*/test_*.py" 2>/dev/null | head -5`
-   檢查依賴項：`grep -E "pytest|unittest|nose" requirements.txt 2>/dev/null`

**Rust:**
-   檢查 Cargo 測試：`grep -E '\[dev-dependencies\]' Cargo.toml 2>/dev/null`
-   尋找測試模組：`find . -name "*.rs" -exec grep -l "#\[cfg(test)\]" {} \; 2>/dev/null | head -5`

**Go:**
-   檢查測試檔案：`find . -name "*_test.go" 2>/dev/null | head -5`
-   檢查 go.mod 是否存在：`test -f go.mod && echo "Go module found"`

**其他語言：**
-   Ruby: 檢查 RSpec：`find . -name ".rspec" -o -name "spec_helper.rb" 2>/dev/null`
-   Java: 檢查 JUnit：`find . -name "pom.xml" -exec grep -l "junit" {} \; 2>/dev/null`

### 2. 測試環境驗證

如果未偵測到測試框架：
-   告知使用者：「⚠️ 未偵測到測試框架。請指定您的測試設定。」
-   詢問：「我應該使用哪個測試命令？(例如，npm test, pytest, cargo test)」
-   儲存回應以供將來使用

### 3. 依賴項檢查

**對於偵測到的框架：**
-   Node.js: 運行 `npm list --depth=0 2>/dev/null | grep -E "jest|mocha|chai|jasmine"`
-   Python: 運行 `pip list 2>/dev/null | grep -E "pytest|unittest|nose"`
-   驗證測試依賴項是否已安裝

如果缺少依賴項：
-   告知使用者：「❌ 未安裝測試依賴項」
-   建議：「運行：npm install (或 pip install -r requirements.txt)」

## 指示 (Instructions)

### 1. 特定框架的設定

根據偵測到的框架創建測試設定：

#### JavaScript/Node.js (Jest)
```yaml
framework: jest
test_command: npm test
test_directory: __tests__
config_file: jest.config.js
options:
  - --verbose
  - --no-coverage
  - --runInBand
environment:
  NODE_ENV: test
```

#### JavaScript/Node.js (Mocha)
```yaml
framework: mocha
test_command: npm test
test_directory: test
config_file: .mocharc.js
options:
  - --reporter spec
  - --recursive
  - --bail
environment:
  NODE_ENV: test
```

#### Python (Pytest)
```yaml
framework: pytest
test_command: pytest
test_directory: tests
config_file: pytest.ini
options:
  - -v
  - --tb=short
  - --strict-markers
environment:
  PYTHONPATH: .
```

#### Rust
```yaml
framework: cargo
test_command: cargo test
test_directory: tests
config_file: Cargo.toml
options:
  - --verbose
  - --nocapture
environment: {}
```

#### Go
```yaml
framework: go
test_command: go test
test_directory: .
config_file: go.mod
options:
  - -v
  - ./...
environment: {}
```

### 2. 測試探索

掃描測試檔案：
-   計算找到的總測試檔案數
-   識別使用的測試命名模式
-   注意任何測試工具程式或輔助程式
-   檢查測試固件或資料

```bash
# Node.js 範例
find . -path "*/node_modules" -prune -o -name "*.test.js" -o -name "*.spec.js" | wc -l
```

### 3. 創建測試執行器設定

使用探索到的資訊創建 `.claude/testing-config.md`：

```markdown
---
framework: {detected_framework}
test_command: {detected_command}
created: [使用來自: date -u +"%Y-%m-%dT%H:%M:%SZ" 的真實日期時間]
---

# 測試設定 (Testing Configuration)

## 框架 (Framework)
- 類型 (Type): {framework_name}
- 版本 (Version): {framework_version}
- 設定檔 (Config File): {config_file_path}

## 測試結構 (Test Structure)
- 測試目錄 (Test Directory): {test_dir}
- 測試檔案 (Test Files): 找到 {count} 個檔案
- 命名模式 (Naming Pattern): {pattern}

## 命令 (Commands)
- 運行所有測試 (Run All Tests): `{full_test_command}`
- 運行特定測試 (Run Specific Test): `{specific_test_command}`
- 使用除錯模式運行 (Run with Debugging): `{debug_command}`

## 環境 (Environment)
- 必要的環境變數 (Required ENV vars): {list}
- 測試資料庫 (Test Database): {if applicable}
- 測試伺服器 (Test Servers): {if applicable}

## 測試執行代理設定 (Test Runner Agent Configuration)
- 使用詳細輸出以便除錯
- 循序運行測試（非並行）
- 捕獲完整的堆疊追蹤
- 不模擬 - 使用真實的實作
- 等待每個測試完全完成
```

### 4. 設定測試執行代理

根據框架準備代理上下文：

```markdown
# 測試執行代理設定 (Test-Runner Agent Configuration)

## 專案測試設定 (Project Testing Setup)
- 框架 (Framework): {framework}
- 測試位置 (Test Location): {directories}
- 總測試數 (Total Tests): {count}
- 上次運行 (Last Run): 從未

## 執行規則 (Execution Rules)
1. 始終使用 `.claude/agents/test-runner.md` 中的測試執行代理
2. 以最大詳細程度運行以進行除錯
3. 不模擬服務 - 使用真實的實作
4. 循序執行測試 - 非並行執行
5. 捕獲包括堆疊追蹤在內的完整輸出
6. 如果測試失敗，在假設程式碼問題之前分析測試結構
7. 報告帶有上下文的詳細失敗分析

## 測試命令範本 (Test Command Templates)
- 完整套件 (Full Suite): `{full_command}`
- 單一檔案 (Single File): `{single_file_command}`
- 模式匹配 (Pattern Match): `{pattern_command}`
- 觀察模式 (Watch Mode): `{watch_command}` (如果可用)

## 要檢查的常見問題 (Common Issues to Check)
- 環境變數是否已正確設定
- 測試資料庫/服務是否正在運行
- 依賴項是否已安裝
- 適當的檔案權限
- 運行之間是否清理了測試狀態
```

### 5. 驗證步驟

設定後：
-   嘗試運行一個簡單的測試以驗證設定
-   檢查測試命令是否有效：`{test_command} --version` 或等效命令
-   驗證測試檔案是否可被發現
-   確保沒有權限問題

### 6. 輸出摘要

```
🧪 測試環境已準備就緒

🔍 偵測結果：
  ✅ 框架 (Framework): {framework_name} {version}
  ✅ 測試檔案 (Test Files): 在 {directories} 中找到 {count} 個檔案
  ✅ 設定檔 (Config): {config_file}
  ✅ 依賴項 (Dependencies): 全部已安裝

📋 測試結構：
  - 模式 (Pattern): {test_file_pattern}
  - 目錄 (Directories): {test_directories}
  - 工具程式 (Utilities): {test_helpers}

🤖 代理設定：
  ✅ 測試執行代理已設定
  ✅ 詳細輸出已啟用
  ✅ 循序執行已設定
  ✅ 真實服務 (無模擬)

⚡ 就緒的命令：
  - 運行所有測試: /testing:run
  - 運行特定測試: /testing:run {test_file}
  - 運行模式匹配: /testing:run {pattern}

💡 提示：
  - 始終以詳細輸出運行測試
  - 如果測試失敗，請檢查測試結構
  - 使用真實服務，而非模擬
  - 讓每個測試完全完成
```

### 7. 錯誤處理

**常見問題：**

**未偵測到框架：**
-   訊息：「⚠️ 找不到測試框架」
-   解決方案：「請手動指定測試命令」
-   儲存使用者的回應以供將來使用

**缺少依賴項：**
-   訊息：「❌ 未安裝測試框架」
-   解決方案：「請先安裝依賴項：npm install / pip install -r requirements.txt」

**無測試檔案：**
-   訊息：「⚠️ 找不到測試檔案」
-   解決方案：「請先創建測試或檢查測試目錄位置」

**權限問題：**
-   訊息：「❌ 無法存取測試檔案」
-   解決方案：「請檢查檔案權限」

### 8. 儲存設定

如果成功，為將來的會話儲存設定：
-   儲存在 `.claude/testing-config.md`
-   包含所有探索到的設定
-   如果後續運行偵測到變更，則更新

## 重要筆記

-   **始終偵測**而非假設測試框架
-   在聲稱準備就緒之前**驗證依賴項**
-   **為除錯進行設定** - 詳細輸出至關重要
-   **不模擬** - 使用真實服務進行準確測試
-   **循序執行** - 避免並行測試問題
-   **儲存設定**以供將來一致的運行

$ARGUMENTS
