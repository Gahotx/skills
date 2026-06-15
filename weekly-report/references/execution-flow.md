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

### 4.3 Commit 信息精炼规则 ⭐ 重要

**目标**：将原始的 commit 信息转化为简洁、专业的工作周报内容，而不是简单罗列所有提交。

**处理原则**：

1. **合并相似提交**：
   - 同一天内对同一功能的多次提交应合并为一条
   - 示例：
     ```
     原始提交：
     - fix: 修复登录按钮样式
     - fix: 调整登录按钮间距
     - fix: 登录按钮hover效果优化
     
     精炼后：
     - 优化登录按钮样式和交互效果
     ```

2. **提炼核心内容**：
   - 移除技术细节，保留业务价值
   - 示例：
     ```
     原始：feat(auth): implement JWT token refresh mechanism with sliding window
     精炼：实现Token自动续期功能
     ```

3. **避免重复描述**：
   - 如果多个commit描述同一件事，只保留最准确的一条
   - 示例：
     ```
     原始：
     - 修复用户列表bug
     - 修复用户列表显示问题
     - 解决用户列表异常
     
     精炼：
     - 修复用户列表显示异常
     ```

4. **使用专业表达**：
   - 将口语化表达转为专业工作描述
   - 示例：
     ```
     原始：改了个bug
     精炼：修复XX功能的异常问题
     ```

5. **按项目分类时的特殊处理**：
   - 每个项目下的内容应该是**工作总结**，不是commit列表
   - 每条内容代表一个**完成的工作项**，而非一次代码提交
   - 同一个功能的多次提交应合并为一条工作项

## 第五步：生成周报

根据用户选择的格式，按照 [输出格式](./output-format.md) 生成周报内容。
