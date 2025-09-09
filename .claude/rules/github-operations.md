# GitHub 操作規則

所有命令中 GitHub CLI 操作的標準模式。

## 關鍵：儲存庫保護 (CRITICAL: Repository Protection)

**在執行任何創建/修改 issue 或 PR 的 GitHub 操作之前：**

```bash
# 檢查遠端 origin 是否為 CCPM 樣板儲存庫
remote_url=$(git remote get-url origin 2>/dev/null || echo "")
if [[ "$remote_url" == *"automazeio/ccpm"* ]] || [[ "$remote_url" == *"automazeio/ccpm.git"* ]]; then
  echo "❌ 錯誤：您正試圖與 CCPM 樣板儲存庫同步！"
  echo ""
  echo "此儲存庫 (automazeio/ccpm) 是一個供他人使用的樣板。"
  echo "您不應該在此處創建 issue 或 PR。"
  echo ""
  echo "要修正此問題："
  echo "1. 將此儲存庫 fork 到您自己的 GitHub 帳戶"
  echo "2. 更新您的遠端 origin："
  echo "   git remote set-url origin https://github.com/YOUR_USERNAME/YOUR_REPO.git"
  echo ""
  echo "或者如果這是一個新專案："
  echo "1. 在 GitHub 上創建一個新的儲存庫"
  echo "2. 更新您的遠端 origin："
  echo "   git remote set-url origin https://github.com/YOUR_USERNAME/YOUR_REPO.git"
  echo ""
  echo "目前的遠端：$remote_url"
  exit 1
fi
```

此檢查**必須**在所有執行以下操作的命令中執行：
-   創建 issue (`gh issue create`)
-   編輯 issue (`gh issue edit`)
-   在 issue 上留言 (`gh issue comment`)
-   創建 PR (`gh pr create`)
-   任何其他修改 GitHub 儲存庫的操作

## 認證 (Authentication)

**不要預先檢查認證。** 只需運行命令並處理失敗：

```bash
gh {command} || echo "❌ GitHub CLI 失敗。請運行：gh auth login"
```

## 常見操作 (Common Operations)

### 獲取 Issue 詳細資訊
```bash
gh issue view {number} --json state,title,labels,body
```

### 創建 Issue
```bash
# 務必先檢查遠端 origin！
gh issue create --title "{title}" --body-file {file} --label "{labels}"
```

### 更新 Issue
```bash
# 務必先檢查遠端 origin！
gh issue edit {number} --add-label "{label}" --add-assignee @me
```

### 新增留言
```bash
# 務必先檢查遠端 origin！
gh issue comment {number} --body-file {file}
```

## 錯誤處理 (Error Handling)

如果任何 `gh` 命令失敗：
1.  顯示清晰的錯誤：「❌ GitHub 操作失敗：{command}」
2.  建議修復方法：「運行：gh auth login」或檢查 issue 編號
3.  不要自動重試

## 重要筆記 (Important Notes)

-   在對 GitHub 進行**任何**寫入操作之前，**務必**檢查遠端 origin。
-   相信 `gh` CLI 已安裝並通過認證。
-   在解析時使用 `--json` 以獲得結構化輸出。
-   保持操作的原子性——每個動作一個 `gh` 命令。
-   不要預先檢查速率限制。
