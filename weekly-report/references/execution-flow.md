# 执行流程

## 第一步：收集用户配置

使用 AskUserQuestion 工具收集配置信息，详见 [自定义选项](./custom-options.md)：

1. 确认日期范围
2. 确认项目列表
3. 确认周报格式
4. 确认保存位置

**日期计算逻辑**：
```bash
# 默认日期范围（含今天，往前推4天，共5天）
end_date=$(date +%Y-%m-%d)
start_date=$(date -d "4 days ago" +%Y-%m-%d)

# 最近一周（周一到周五）
day_of_week=$(date +%u)  # 1=周一, 7=周日
start_date=$(date -d "$((day_of_week + 6)) days ago" +%Y-%m-%d)
end_date=$(date -d "$((day_of_week + 2)) days ago" +%Y-%m-%d)
```

## 第二步：获取项目名称

按优先级获取显示名称，详见 [高级功能 - 项目名称解析](./advanced-features.md#项目名称解析)：

1. 从 `.env` 文件读取（APP_TITLE、PROJECT_NAME 等）
2. 使用目录名称
3. 英文翻译为中文

## 第三步：提取 Git 提交记录

对于每个项目，使用以下命令提取指定日期范围内的 commit 记录：

### 3.1 获取 commit 记录

```bash
# 获取指定日期范围内的commit记录（按日期排序）
git -C <project_path> log \
  --after="<start_date>" \
  --before="<end_date + 1 day>" \
  --pretty=format:"%h|%s|%ai" \
  --no-merges
```

**输出格式说明**：
- `%h`: commit hash（简短）
- `%s`: commit message
- `%ai`: commit日期时间

### 3.2 解析 commit 信息

将每条 commit 解析为：
```json
{
  "hash": "abc1234",
  "message": "feat: 新增用户登录功能",
  "date": "2026-05-15 10:30:00 +0800",
  "type": "feat"
}
```

## 第四步：分类 Commit 记录

### 4.1 按类型分类

根据 commit message 前缀分类：

| Commit 前缀 | 分类名称 | 说明 |
|------------|----------|------|
| feat | 功能开发 | 新功能 |
| fix | Bug修复 | 问题修复 |
| refactor | 代码重构 | 重构优化 |
| docs | 文档更新 | 文档变更 |
| style | 样式调整 | UI/CSS |
| test | 测试相关 | 测试代码 |
| chore | 构建/工具 | 构建配置 |
| perf | 性能优化 | 性能改进 |
| ci | CI/CD | 持续集成 |
| revert | 回滚 | 版本回退 |
| 其他 | 其他 | 无前缀或未识别 |

### 4.2 Commit Message 处理规则

1. **移除类型前缀**：
   - `feat: 新增功能` → `新增功能`
   - `fix(scope): 修复bug` → `修复bug`
   - `feat(auth): implement JWT` → `implement JWT`

2. **保留有意义的描述**：
   - 移除前缀后保留核心内容
   - 如果 message 过长，适当精简但保留关键信息

3. **翻译英文 commit**（可选）：
   - 将英文 commit message 翻译为中文
   - 保留技术术语

## 第五步：生成周报

根据用户选择的格式，按照 [输出格式](./output-format.md) 生成周报内容。
