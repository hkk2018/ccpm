---
allowed-tools: Bash, Read, Write
---

# 測試參考更新 (Test Reference Update)

測試 epic-sync 中使用的任務參考更新邏輯。

## 用法 (Usage)
```
/pm:test-reference-update
```

## 指示 (Instructions)

### 1. 創建測試檔案

創建帶有參考的測試任務檔案：
```bash
mkdir -p /tmp/test-refs
cd /tmp/test-refs

# 創建任務 001
cat > 001.md << 'EOF'
---
name: Task One
status: open
depends_on: []
parallel: true
conflicts_with: [002, 003]
---
# Task One
This is task 001.
EOF

# 創建任務 002
cat > 002.md << 'EOF'
---
name: Task Two
status: open
depends_on: [001]
parallel: false
conflicts_with: [003]
---
# Task Two
This is task 002, depends on 001.
EOF

# 創建任務 003
cat > 003.md << 'EOF'
---
name: Task Three
status: open
depends_on: [001, 002]
parallel: false
conflicts_with: []
---
# Task Three
This is task 003, depends on 001 and 002.
EOF
```

### 2. 創建映射

模擬 issue 創建映射：
```bash
# 模擬 任務 -> issue 編號 映射
cat > /tmp/task-mapping.txt << 'EOF'
001.md:42
002.md:43
003.md:44
EOF

# 創建 舊 -> 新 ID 映射
> /tmp/id-mapping.txt
while IFS=: read -r task_file task_number; do
  old_num=$(basename "$task_file" .md)
  echo "$old_num:$task_number" >> /tmp/id-mapping.txt
done < /tmp/task-mapping.txt

echo "ID Mapping:"
cat /tmp/id-mapping.txt
```

### 3. 更新參考

處理每個檔案並更新參考：
```bash
while IFS=: read -r task_file task_number; do
  echo "正在處理: $task_file -> $task_number.md"
  
  # 讀取檔案內容
  content=$(cat "$task_file")
  
  # 更新參考
  while IFS=: read -r old_num new_num; do
    content=$(echo "$content" | sed "s/\b$old_num\b/$new_num/g")
  done < /tmp/id-mapping.txt
  
  # 寫入新檔案
  new_name="${task_number}.md"
  echo "$content" > "$new_name"
  
  echo "更新內容預覽:"
  grep -E "depends_on:|conflicts_with:" "$new_name"
  echo "---"
done < /tmp/task-mapping.txt
```

### 4. 驗證結果

檢查參考是否已正確更新：
```bash
echo "=== 最終結果 ==="
for file in 42.md 43.md 44.md; do
  echo "檔案: $file"
  grep -E "name:|depends_on:|conflicts_with:" "$file"
  echo ""
done
```

預期輸出：
-   42.md 應有 conflicts_with: [43, 44]
-   43.md 應有 depends_on: [42] 和 conflicts_with: [44]
-   44.md 應有 depends_on: [42, 43]

### 5. 清理

```bash
cd -
rm -rf /tmp/test-refs
rm -f /tmp/task-mapping.txt /tmp/id-mapping.txt
echo "✅ 測試完成並已清理"
```