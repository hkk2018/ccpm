---
allowed-tools: Bash, Read, Write, LS
---

# 同步 Issue (Issue Sync)

將本地更新作為 GitHub issue 評論推送，以提供透明的審計追蹤。

## 用法 (Usage)
```
/pm:issue-sync <issue_number>
```

## 必要規則

**重要：** 在執行此命令之前，請閱讀並遵守：
- `.claude/rules/datetime.md` - 用於獲取真實的當前日期/時間

## 飛行前檢查清單

在繼續之前，請完成這些驗證步驟。
不要用飛行前檢查的進度來打擾使用者（例如說「我將不會...」）。只需執行它們然後繼續。

0. **儲存庫保護檢查：**
   遵循 `/rules/github-operations.md` - 檢查遠端 origin：
   ```bash
   remote_url=$(git remote get-url origin 2>/dev/null || echo "")
   if [[ "$remote_url" == *"automazeio/ccpm"* ]]; then
     echo "❌ 錯誤：無法同步到 CCPM 樣板儲存庫！"
     echo "更新您的遠端：git remote set-url origin https://github.com/YOUR_USERNAME/YOUR_REPO.git"
     exit 1
   fi
   ```

1. **GitHub 認證：**
   - 運行：`gh auth status`
   - 如果未認證，告知使用者：「❌ GitHub CLI 未認證。請運行：gh auth login」

2. **Issue 驗證：**
   - 運行：`gh issue view $ARGUMENTS --json state`
   - 如果 issue 不存在，告知使用者：「❌ 找不到 Issue #$ARGUMENTS」
   - 如果 issue 已關閉但完成度 < 100%，警告：「⚠️ Issue 已關閉但工作未完成」

3. **本地更新檢查：**
   - 檢查 `.claude/epics/*/updates/$ARGUMENTS/` 目錄是否存在
   - 如果找不到，告知使用者：「❌ 找不到 issue #$ARGUMENTS 的本地更新。請運行：/pm:issue-start $ARGUMENTS」
   - 檢查 progress.md 是否存在
   - 如果不存在，告知使用者：「❌ 找不到進度追蹤。請使用 /pm:issue-start $ARGUMENTS 進行初始化」

4. **檢查上次同步：**
   - 從 progress.md frontmatter 讀取 `last_sync`
   - 如果最近同步過（< 5 分鐘），詢問：「⚠️ 最近已同步。是否仍要強制同步？(是/否)」
   - 計算自上次同步以來的新內容

5. **驗證變更：**
   - 檢查是否有實際的更新需要同步
   - 如果沒有變更，告知使用者：「ℹ️ 自 {last_sync} 以來沒有新的更新需要同步」
   - 如果無內容可同步，則優雅地退出

## 指示 (Instructions)

您正在為 **Issue #$ARGUMENTS** 將本地開發進度同步到 GitHub 作為 issue 評論。

### 1. 收集本地更新
收集該 issue 的所有本地更新：
- 從 `.claude/epics/{epic_name}/updates/$ARGUMENTS/` 讀取
- 檢查以下檔案中的新內容：
  - `progress.md` - 開發進度
  - `notes.md` - 技術筆記和決策
  - `commits.md` - 最近的提交和變更
  - 任何其他更新檔案

### 2. 更新進度追蹤的 Frontmatter
獲取當前日期時間：`date -u +"%Y-%m-%dT%H:%M:%SZ"`

更新 progress.md 檔案的 frontmatter：
```yaml
---
issue: $ARGUMENTS
started: [保留現有日期]
last_sync: [使用上面命令的真實日期時間]
completion: [計算出的百分比 0-100%]
---
```

### 3. 確定新增內容
與上次同步比較以識別新內容：
- 尋找同步時間戳標記
- 識別新的區塊或更新
- 僅收集自上次同步以來的增量變更

### 4. 格式化更新評論
創建全面的更新評論：

```markdown
## 🔄 進度更新 - {current_date}

### ✅ 已完成的工作
{list_completed_items}

### 🔄 進行中
{current_work_items}

### 📝 技術筆記
{key_technical_decisions}

### 📊 驗收標準狀態
- ✅ {completed_criterion}
- 🔄 {in_progress_criterion}
- ⏸️ {blocked_criterion}
- □ {pending_criterion}

### 🚀 下一步
{planned_next_actions}

### ⚠️ 阻礙
{any_current_blockers}

### 💻 最近的提交
{commit_summaries}

---
*進度: {completion}% | 於 {timestamp} 從本地更新同步*
```

### 5. 發佈到 GitHub
使用 GitHub CLI 新增評論：
```bash
gh issue comment #$ARGUMENTS --body-file {temp_comment_file}
```

### 6. 更新本地任務檔案
獲取當前日期時間：`date -u +"%Y-%m-%dT%H:%M:%SZ"`

使用同步資訊更新任務檔案的 frontmatter：
```yaml
---
name: [Task Title]
status: open
created: [保留現有日期]
updated: [使用上面命令的真實日期時間]
github: https://github.com/{org}/{repo}/issues/$ARGUMENTS
---
```

### 7. 處理完成情況
如果任務已完成，更新所有相關的 frontmatter：

**任務檔案 frontmatter**：
```yaml
---
name: [Task Title]
status: closed
created: [existing date]
updated: [current date/time]
github: https://github.com/{org}/{repo}/issues/$ARGUMENTS
---
```

**進度檔案 frontmatter**：
```yaml
---
issue: $ARGUMENTS
started: [existing date]
last_sync: [current date/time]
completion: 100%
---
```

**Epic 進度更新**：根據已完成的任務重新計算 epic 進度，並更新 epic frontmatter：
```yaml
---
name: [Epic Name]
status: in-progress
created: [existing date]
progress: [根據已完成任務計算的百分比]%
prd: [existing path]
github: [existing URL]
---
```

### 8. 完成評論
如果任務已完成：
```markdown
## ✅ 任務完成 - {current_date}

### 🎯 所有驗收標準均已滿足
- ✅ {criterion_1}
- ✅ {criterion_2}
- ✅ {criterion_3}

### 📦 可交付成果
- {deliverable_1}
- {deliverable_2}

### 🧪 測試
- 單元測試：✅ 通過
- 整合測試：✅ 通過
- 手動測試：✅ 完成

### 📚 文件
- 程式碼文件：✅ 已更新
- README 更新：✅ 完成

此任務已準備好進行審查並可關閉。

---
*任務完成: 100% | 於 {timestamp} 同步*
```

### 9. 輸出摘要
```
☁️ 已將更新同步到 GitHub Issue #$ARGUMENTS

📝 更新摘要：
   進度項目：{progress_count}
   技術筆記：{notes_count}
   引用的提交數：{commit_count}

📊 當前狀態：
   任務完成度：{task_completion}%
   Epic 進度：{epic_progress}%
   已完成標準：{completed}/{total}

🔗 查看更新：gh issue view #$ARGUMENTS --comments
```

### 10. Frontmatter 維護
- 始終使用當前時間戳更新任務檔案的 frontmatter
- 在進度檔案中追蹤完成百分比
- 當任務完成時更新 epic 進度
- 維護同步時間戳以供審計追蹤

### 11. 增量同步檢測

**防止重複評論：**
1. 每次同步後在本地檔案中新增同步標記：
   ```markdown
   <!-- SYNCED: 2024-01-15T10:30:00Z -->
   ```
2. 僅同步最後一個標記後新增的內容
3. 如果沒有新內容，則跳過同步並顯示訊息：「自上次同步以來無更新」

### 12. 評論大小管理

**處理 GitHub 的評論限制：**
- 最大評論大小：65,536 個字元
- 如果更新超出限制：
  1. 分割成多條評論
  2. 或總結並附上完整詳細資訊的連結
  3. 警告使用者：「⚠️ 由於大小限制，更新被截斷。完整詳細資訊在本地檔案中。」

### 13. 錯誤處理

**常見問題與恢復：**

1. **網路錯誤：**
   - 訊息：「❌ 發佈評論失敗：網路錯誤」
   - 解決方案：「請檢查網路連線並重試」
   - 保持本地更新完整以便重試

2. **速率限制：**
   - 訊息：「❌ 超出 GitHub 速率限制」
   - 解決方案：「請等待 {minutes} 分鐘或使用不同的 token」
   - 將評論保存在本地以便稍後同步

3. **權限被拒絕：**
   - 訊息：「❌ 無法在 issue 上評論（權限被拒絕）」
   - 解決方案：「請檢查儲存庫存取權限」

4. **Issue 已鎖定：**
   - 訊息：「⚠️ Issue 已被鎖定，無法評論」
   - 解決方案：「請聯繫儲存庫管理員解鎖」

### 14. Epic 進度計算

更新 epic 進度時：
1. 計算 epic 目錄中的總任務數
2. 計算 frontmatter 中 `status: closed` 的任務數
3. 計算：`progress = (closed_tasks / total_tasks) * 100`
4. 四捨五入到最接近的整數
5. 僅在百分比變更時更新 epic frontmatter

### 15. 同步後驗證

成功同步後：
- [ ] 驗證評論已發佈到 GitHub
- [ ] 確認 frontmatter 已用同步時間戳更新
- [ ] 如果任務完成，檢查 epic 進度是否已更新
- [ ] 驗證本地檔案無資料損壞

這為利害關係人創建了一個透明的開發進度審計追蹤，他們可以即時關注 Issue #$ARGUMENTS 的進展，同時在所有專案檔案中保持準確的 frontmatter。
