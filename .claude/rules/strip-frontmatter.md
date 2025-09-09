# 移除 Frontmatter

在將內容發送到 GitHub 之前移除 YAML frontmatter 的標準方法。

## 問題所在 (The Problem)

YAML frontmatter 包含不應出現在 GitHub issue 中的內部元數據：
-   `status`, `created`, `updated` 欄位
-   內部參考和 ID
-   本地檔案路徑

## 解決方案 (The Solution)

使用 `sed` 從任何 markdown 檔案中移除 frontmatter：

```bash
# 移除 frontmatter（位於前兩個 --- 行之間的所有內容）
sed '1,/^---$/d; 1,/^---$/d' input.md > output.md
```

這會移除：
1.  開頭的 `---` 行
2.  所有的 YAML 內容
3.  結尾的 `---` 行

## 何時移除 Frontmatter (When to Strip Frontmatter)

在以下情況下務必移除 frontmatter：
-   從 markdown 檔案創建 GitHub issue 時
-   將檔案內容作為留言發布時
-   向外部使用者顯示內容時
-   與任何外部系統同步時

## 範例 (Examples)

### 從檔案創建 issue
```bash
# 不好 - 包含 frontmatter
gh issue create --body-file task.md

# 好 - 移除了 frontmatter
sed '1,/^---$/d; 1,/^---$/d' task.md > /tmp/clean.md
gh issue create --body-file /tmp/clean.md
```

### 發布留言
```bash
# 發布前移除 frontmatter
sed '1,/^---$/d; 1,/^---$/d' progress.md > /tmp/comment.md
gh issue comment 123 --body-file /tmp/comment.md
```

### 在迴圈中
```bash
for file in *.md; do
  # 從每個檔案中移除 frontmatter
  sed '1,/^---$/d; 1,/^---$/d' "$file" > "/tmp/$(basename $file)"
  # 使用乾淨的版本
done
```

## 替代方法 (Alternative Approaches)

如果 `sed` 不可用或您需要更多控制：

```bash
# 使用 awk
awk 'BEGIN{fm=0} /^---$/{fm++; next} fm==2{print}' input.md > output.md

# 使用 grep 搭配行號
grep -n "^---$" input.md | head -2 | tail -1 | cut -d: -f1 | xargs -I {} tail -n +$(({}+1)) input.md
```

## 重要筆記 (Important Notes)

-   務必先用範例檔案進行測試。
-   保持原始檔案完整。
-   對清理後的內容使用暫存檔案。
-   有些檔案可能沒有 frontmatter——此命令能優雅地處理這種情況。