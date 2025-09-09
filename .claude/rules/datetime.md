# 日期時間規則 (DateTime Rule)

## 獲取當前日期與時間

當任何命令需要當前日期/時間（用於 frontmatter、時間戳或日誌）時，您**必須**從系統獲取**真實的**當前日期/時間，而不是估計或使用預留位置的值。

### 如何獲取當前日期時間

使用 `date` 命令獲取當前 ISO 8601 格式的日期時間：

```bash
# 獲取 ISO 8601 格式的當前日期時間（適用於 Linux/Mac）
date -u +"%Y-%m-%dT%H:%M:%SZ"

# 支援此功能的系統的替代方案
date --iso-8601=seconds

# 對於 Windows（如果使用 PowerShell）
Get-Date -Format "yyyy-MM-ddTHH:mm:ssZ"
```

### 必要格式

frontmatter 中的所有日期**必須**使用帶有 UTC 時區的 ISO 8601 格式：
-   格式：`YYYY-MM-DDTHH:MM:SSZ`
-   範例：`2024-01-15T14:30:45Z`

### 在 Frontmatter 中的使用

在任何檔案（PRD、Epic、Task、Progress）中建立或更新 frontmatter 時，請始終使用真實的當前日期時間：

```yaml
---
name: feature-name
created: 2024-01-15T14:30:45Z  # 使用 date 命令的實際輸出
updated: 2024-01-15T14:30:45Z  # 使用 date 命令的實際輸出
---
```

### 實作說明

1.  **在寫入任何帶有 frontmatter 的檔案之前：**
    -   運行：`date -u +"%Y-%m-%dT%H:%M:%SZ"`
    -   儲存輸出
    -   在 frontmatter 中使用這個確切的值

2.  **對於創建檔案的命令：**
    -   PRD 創建：對 `created` 欄位使用真實日期。
    -   Epic 創建：對 `created` 欄位使用真實日期。
    -   Task 創建：對 `created` 和 `updated` 欄位都使用真實日期。
    -   進度追蹤：對 `started` 和 `last_sync` 欄位使用真實日期。

3.  **對於更新檔案的命令：**
    -   始終用當前的真實日期時間更新 `updated` 欄位。
    -   保留原始的 `created` 欄位。
    -   對於同步操作，用真實日期時間更新 `last_sync`。

### 範例

**創建一個新的 PRD：**
```bash
# 首先，獲取當前日期時間
CURRENT_DATE=$(date -u +"%Y-%m-%dT%H:%M:%SZ")
# 輸出：2024-01-15T14:30:45Z

# 然後在 frontmatter 中使用：
---
name: user-authentication
description: 使用者認證與授權系統
status: backlog
created: 2024-01-15T14:30:45Z  # 使用實際的 $CURRENT_DATE 值
---
```

**更新一個現有的 task：**
```bash
# 獲取當前日期時間以進行更新
UPDATE_DATE=$(date -u +"%Y-%m-%dT%H:%M:%SZ")

# 只更新 'updated' 欄位：
---
name: implement-login-api
status: in-progress
created: 2024-01-10T09:15:30Z  # 保留原始值
updated: 2024-01-15T14:30:45Z  # 使用新的 $UPDATE_DATE 值
---
```

### 重要筆記

-   **絕不使用預留位置日期**，如 `[Current ISO date/time]` 或 `YYYY-MM-DD`。
-   **絕不估計日期** - 始終獲取實際的系統時間。
-   **始終使用 UTC**（`Z` 後綴），以確保跨時區的一致性。
-   **保持時區一致性** - 系統中的所有日期都使用 UTC。

### 跨平台相容性

如果您需要確保在不同系統間的相容性：

```bash
# 首先嘗試主要方法
date -u +"%Y-%m-%dT%H:%M:%SZ" 2>/dev/null || \
# 為沒有 -u 旗標的系統提供後備方案
date +"%Y-%m-%dT%H:%M:%SZ" 2>/dev/null || \
# 最後的手段：如果可用，則使用 Python
python3 -c "from datetime import datetime; print(datetime.utcnow().strftime('%Y-%m-%dT%H:%M:%SZ'))" 2>/dev/null || \
python -c "from datetime import datetime; print(datetime.utcnow().strftime('%Y-%m-%dT%H:%M:%SZ'))" 2>/dev/null
```

## 規則優先級

此規則具有**最高優先級**，所有執行以下操作的命令都必須遵守：
-   創建帶有 frontmatter 的新檔案。
-   更新帶有 frontmatter 的現有檔案。
-   追蹤時間戳或進度。
-   記錄任何基於時間的資訊。

受影響的命令：`prd-new`, `prd-parse`, `epic-decompose`, `epic-sync`, `issue-start`, `issue-sync`，以及任何其他寫入時間戳的命令。