# Cursor 代理的 AST-Grep 整合協定

## 何時使用 AST-Grep

在以下情況下，使用 `ast-grep`（如果已安裝）而非普通的正則表達式或文本搜索：

-   涉及**結構化程式碼模式**時（例如，尋找所有函數呼叫、類別定義或方法實作）
-   需要**語言感知重構**時（例如，重命名變數、更新函數簽名或更改導入）
-   需要**複雜的程式碼分析**時（例如，在不同的語法上下文中尋找模式的所有用法）
-   需要**跨語言搜索**時（例如，在單一儲存庫中同時處理 Ruby 和 TypeScript）
-   **語意程式碼理解**很重要時（例如，基於程式碼結構而非僅僅是文本來尋找模式）

## AST-Grep 命令模式

### 基本搜索範本：
```sh
ast-grep --pattern '$PATTERN' --lang $LANGUAGE $PATH
```

### 常見用例

-   **尋找函數呼叫：**
    `ast-grep --pattern 'functionName($$$)' --lang javascript .`
-   **尋找類別定義：**
    `ast-grep --pattern 'class $NAME { $$$ }' --lang typescript .`
-   **尋找變數賦值：**
    `ast-grep --pattern '$VAR = $$$' --lang ruby .`
-   **尋找導入語句：**
    `ast-grep --pattern 'import { $$$ } from "$MODULE"' --lang javascript .`
-   **尋找物件上的方法呼叫：**
    `ast-grep --pattern '$OBJ.$METHOD($$$)' --lang typescript .`
-   **尋找 React hooks：**
    `ast-grep --pattern 'const [$STATE, $SETTER] = useState($$$)' --lang typescript .`
-   **尋找 Ruby 類別定義：**
    `ast-grep --pattern 'class $NAME < $$$; $$$; end' --lang ruby .`

## 模式語法參考

-   `$VAR` — 匹配任何單一節點並捕獲它
-   `$$$` — 匹配零個或多個節點（通配符）
-   `$$` — 匹配一個或多個節點
-   字面程式碼 — 完全按照書寫的方式匹配

## 支援的語言

-   javascript, typescript, ruby, python, go, rust, java, c, cpp, html, css, yaml, json 等等

## 整合工作流程

### 使用 ast-grep 之前：
1.  **檢查 ast-grep 是否已安裝：**
    如果未安裝，則跳過並退回到正則表達式/語意搜索。
    ```sh
    command -v ast-grep >/dev/null 2>&1 || echo "ast-grep 未安裝，跳過 AST 搜索"
    ```
2.  **識別**任務是否涉及結構化程式碼模式或語言感知重構。
3.  **確定**要搜索的適當語言。
4.  使用 ast-grep 語法**建構**模式。
5.  **運行** ast-grep 以收集精確的結構資訊。
6.  **使用**結果來為程式碼編輯、重構或進一步分析提供資訊。

### 範例工作流程

當被要求「尋找所有呼叫 `perform` 的 Ruby 服務物件」時：

1.  **檢查 ast-grep：**
    ```sh
    command -v ast-grep >/dev/null 2>&1 && ast-grep --pattern 'perform($$$)' --lang ruby app/services/
    ```
2.  結構化地**分析**結果。
3.  如果需要，使用 codebase 語意搜索以獲取額外的上下文。
4.  基於結構性理解**進行**明智的編輯。

### 將 ast-grep 與內部工具結合

-   **codebase_search** 用於語意上下文和文件
-   **read_file** 用於檢查 ast-grep 找到的特定檔案
-   **edit_file** 用於進行精確、具上下文感知的程式碼變更

### 進階用法
-   **用於程式化處理的 JSON 輸出：**
    `ast-grep --pattern '$PATTERN' --lang $LANG $PATH --json`
-   **替換模式：**
    `ast-grep --pattern '$OLD_PATTERN' --rewrite '$NEW_PATTERN' --lang $LANG $PATH`
-   **互動模式：**
    `ast-grep --pattern '$PATTERN' --lang $LANG $PATH --interactive`

## 相較於正則表達式的主要優點

1.  **語言感知** — 理解語法和語意
2.  **結構化匹配** — 無論格式如何都能找到模式
3.  **跨語言** — 在不同語言中一致地工作
4.  **精確重構** — 安全地進行結構性變更
5.  **上下文感知** — 理解程式碼層次結構和範圍

## 決策矩陣：何時使用各種工具

| 任務類型 | 工具選擇 | 原因 |
|---|---|---|
| 尋找文本模式 | grep_search | 簡單的文本匹配 |
| 尋找程式碼結構 | ast-grep | 語法感知的搜索 |
| 理解語意 | codebase_search | AI 驅動的上下文 |
| 進行編輯 | edit_file | 精確的檔案編輯 |
| 結構性重構 | ast-grep + edit_file | 結構 + 精確度 |

**對於程式碼結構分析，始終優先選擇 ast-grep 而非基於正則表達式的方法，但前提是它已安裝且可用。**
