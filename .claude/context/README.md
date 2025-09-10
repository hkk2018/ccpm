# 上下文目錄 (Context Directory)

此目錄包含專案上下文文件，提供有關您專案當前狀態、結構和方向的全面資訊。這些上下文檔案作為知識庫，供 AI 代理和團隊成員快速理解和貢獻專案。

## 目的 (Purpose)

上下文系統能夠：
-   **快速的代理上手 (Fast Agent Onboarding)**：新的 AI 代理可以透過標準化文件快速了解專案。
-   **專案連續性 (Project Continuity)**：在不同的開發會話和團隊變動中保持知識。
-   **一致的理解 (Consistent Understanding)**：確保所有貢獻者都能存取相同的專案資訊。
-   **動態文件 (Living Documentation)**：保持專案知識的即時性和可操作性。

## 核心上下文檔案 (Core Context Files)

完全初始化後，此目錄包含：

### 專案基礎 (Project Foundation)
-   **`project-brief.md`** - 專案範圍、目標和關鍵指標
-   **`project-vision.md`** - 長期願景和戰略方向
-   **`project-overview.md`** - 功能和能力的高層次摘要
-   **`progress.md`** - 當前專案狀態、已完成的工作和接下來的步驟

### 技術上下文 (Technical Context)
-   **`tech-context.md`** - 依賴項、技術和開發工具
-   **`project-structure.md`** - 目錄結構和檔案組織
-   **`system-patterns.md`** - 架構模式和設計決策
-   **`project-style-guide.md`** - 編碼標準、慣例和風格偏好

### 產品上下文 (Product Context)
-   **`product-context.md`** - 產品需求、目標使用者和核心功能

## 上下文命令 (Context Commands)

使用這些命令來管理您的專案上下文：

### 初始化上下文
```bash
/context:create
```
分析您的專案並創建初始上下文文件。在以下情況使用：
-   開始一個新專案
-   為現有專案新增上下文
-   重大的專案重組

### 載入上下文
```bash
/context:prime
```
為新的代理會話載入所有上下文資訊。在以下情況使用：
-   開始一個新的開發會話
-   新團隊成員上手
-   快速了解專案狀態

### 更新上下文
```bash
/context:update
```
更新上下文文件以反映當前的專案狀態。在以下情況使用：
-   開發會話結束時
-   完成主要功能後
-   當專案方向改變時
-   架構變更後

## 上下文工作流程 (Context Workflow)

1.  **專案開始**: 運行 `/context:create` 建立基準文件
2.  **會話開始**: 運行 `/context:prime` 載入當前上下文
3.  **開發**: 在完全的上下文感知下進行專案工作
4.  **會話結束**: 運行 `/context:update` 捕獲變更和進度

## 好處 (Benefits)

-   **減少上手時間 (Reduced Onboarding Time)**：新的貢獻者能快速了解專案
-   **維持專案記憶 (Maintained Project Memory)**：會話之間不會遺失任何資訊
-   **一致的架構 (Consistent Architecture)**：決策被記錄並遵循
-   **清晰的進度追蹤 (Clear Progress Tracking)**：始終知道已完成什麼和下一步是什麼
-   **增強的 AI 協作 (Enhanced AI Collaboration)**：AI 代理對專案有充分的理解

## 最佳實踐 (Best Practices)

-   **保持最新**: 定期更新上下文，特別是在重大變更後
-   **力求簡潔**: 專注於有助於理解的基本資訊
-   **保持一致**: 遵循已建立的格式和結構
-   **記錄決策**: 捕獲架構和設計決策
-   **追蹤進度**: 維護準確的狀態和下一步驟

## 整合 (Integration)

上下文系統與以下內容整合：
-   **專案管理 (Project Management)**：與 PRD、epics 和任務追蹤連結
-   **開發工作流程 (Development Workflow)**：支援連續的開發會話
-   **文件 (Documentation)**：補充現有的專案文件
-   **團隊協作 (Team Collaboration)**：在貢獻者之間提供共享的理解

從 `/context:create` 開始，初始化您專案的知識庫！
