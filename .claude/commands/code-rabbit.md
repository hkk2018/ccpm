---
allowed-tools: Task, Read, Edit, MultiEdit, Write, LS, Grep
---

# CodeRabbit 審查處理程序

以具備上下文感知能力的判斷力，處理 CodeRabbit 的審查評論。

## 用法 (Usage)
```
/code-rabbit
```

然後貼上一條或多條 CodeRabbit 評論。

## 指示 (Instructions)

### 1. 初始上下文 (Initial Context)

告知使用者：
```
我會謹慎地審查 CodeRabbit 的評論，因為 CodeRabbit 無法存取整個程式碼庫，可能不了解完整的上下文。

對於每條評論，我會：
- 根據我們的程式碼庫上下文評估其是否有效。
- 接受能改善程式碼品質的建議。
- 忽略不適用於我們架構的建議。
- 解釋我接受/忽略決定的原因。
```

### 2. 處理評論 (Process Comments)

#### 單一檔案評論 (Single File Comments)
如果所有評論都與單一檔案相關：
-   讀取該檔案以了解上下文。
-   評估每條建議。
-   使用 MultiEdit 批次應用已接受的變更。
-   報告哪些建議被接受/忽略及其原因。

#### 多檔案評論 (Multiple File Comments)
如果評論橫跨多個檔案：

使用 Task 工具啟動並行子代理：
```yaml
Task:
  description: "針對 {filename} 的 CodeRabbit 修復"
  subagent_type: "general-purpose"
  prompt: |
    審查並應用針對 {filename} 的 CodeRabbit 建議。
    
    要評估的評論：
    {relevant_comments_for_this_file}
    
    指示：
    1. 讀取檔案以了解上下文。
    2. 對於每條建議：
       - 根據程式碼庫的模式評估其有效性。
       - 如果能改善品質/正確性，則接受。
       - 如果不適用，則忽略。
    3. 使用 Edit/MultiEdit 應用已接受的變更。
    4. 返回摘要：
       - 已接受 (Accepted): {附帶原因的列表}
       - 已忽略 (Ignored): {附帶原因的列表}
       - 已做的變更 (Changes made): {簡要描述}
    
    請謹慎行事——CodeRabbit 缺乏完整的上下文。
```

### 3. 整合結果 (Consolidate Results)

所有子代理完成後：
```
📋 CodeRabbit 審查摘要

已處理檔案數 (Files Processed): {count}

已接受的建議 (Accepted Suggestions):
  {file}: {changes_made}
  
已忽略的建議 (Ignored Suggestions):
  {file}: {reason_ignored}

總體 (Overall): {X}/{Y} 條建議已應用
```

### 4. 常見的忽略模式 (Common Patterns to Ignore)

-   與專案慣例衝突的**風格偏好**。
-   不適用於我們特定用例的**通用最佳實踐**。
-   針對非性能關鍵程式碼的**性能優化**。
-   針對內部工具的**無障礙性建議**。
-   針對已驗證模式的**安全警告**。
-   會破壞我們結構的**導入重組**。

### 5. 常見的接受模式 (Common Patterns to Accept)

-   **實際的錯誤**（空值檢查、錯誤處理）。
-   **安全漏洞**（除非是誤報）。
-   **資源洩漏**（未關閉的連線、記憶體洩漏）。
-   **類型安全問題**（TypeScript/類型提示）。
-   **邏輯錯誤**（差一錯誤、不正確的條件）。
-   **缺少錯誤處理**。

## 決策框架 (Decision Framework)

對於每條建議，請考量：
1.  **它是否正確？** - 問題是否確實存在？
2.  **它是否相關？** - 它是否適用於我們的用例？
3.  **它是否有益？** - 修復它會改善程式碼嗎？
4.  **它是否安全？** - 這個變更會引入問題嗎？

只有當所有答案都是「是」，或者好處明顯大於風險時才應用。

## 重要筆記 (Important Notes)

-   CodeRabbit 很有幫助，但缺乏上下文。
-   相信您對程式碼庫的理解勝過通用的建議。
-   簡要解釋決定以保持審計追蹤。
-   為提高效率，批次處理相關的變更。
-   對多檔案審查使用並行代理以節省時間。