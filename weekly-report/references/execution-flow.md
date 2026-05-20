# 执行流程

## 第一步：收集用户配置

使用 AskUserQuestion 工具收集以下信息（所有交互必须用中文）：

### 1.1 确认周报日期范围

**选项**：
- 默认：最近5天（含今天，往前推4天）
- 最近一周：上一个完整工作周（上周一到上周五）
- 自定义：用户指定开始和结束日期

**询问示例**：
> 请选择周报日期范围：
> - 默认：最近5天（2026.05.15 - 2026.05.19）
> - 最近一周：上一个完整工作周（2026.05.11 - 2026.05.15）
> - 自定义日期

**计算逻辑**：
```bash
# 默认日期范围（含今天，往前推4天，共5天）
end_date=$(date +%Y-%m-%d)
start_date=$(date -d "4 days ago" +%Y-%m-%d)

# 最近一周（周一到周五）
# 始终返回上一个完整的工作周（上周一到上周五）
day_of_week=$(date +%u)  # 1=周一, 7=周日
# 上周一 = 今天 - (day_of_week + 6) 天
start_date=$(date -d "$((day_of_week + 6)) days ago" +%Y-%m-%d)
# 上周五 = 上周一 + 4 天
end_date=$(date -d "$((day_of_week + 2)) days ago" +%Y-%m-%d)
```

### 1.2 确认项目列表

**询问示例**：
> 项目路径：默认使用当前目录，还有其他项目需要加入周报吗？

**支持格式**：
- 单个项目：直接输入路径
- 多个项目：用逗号分隔
- 示例：`E:/Projects/frontend, E:/Projects/backend, E:/Projects/admin`

### 1.3 确认周报格式

**询问示例**：
> 周报格式选择：
> - 按commit类型分类（功能开发、Bug修复等）
> - 按项目分类（每个项目单独一节）

### 1.4 确认保存位置

**询问示例**：
> 保存位置：默认保存到 docs/weekly-report/ 目录，需要修改吗？

**.gitignore 规则**：
- 仅当保存位置是**相对路径**（如 `docs/weekly-report/`、`./docs/report`）时，询问是否添加到 .gitignore
- 当保存位置是**绝对路径**（如 `E:\工作文档\周报`）时，不询问 .gitignore

## 第二步：获取项目名称

对于每个项目，按以下优先级获取显示名称：

### 2.1 从 .env 文件读取

检查项目根目录的以下文件：
- `.env`
- `.env.local`
- `.env.development`
- `.env.production`

查找以下变量：
```bash
# 常见的应用名称变量
APP_TITLE=xxx
PROJECT_NAME=xxx
VITE_APP_TITLE=xxx
REACT_APP_NAME=xxx
SYSTEM_TITLE=xxx
APP_NAME=xxx
```

### 2.2 使用目录名称

如果 .env 中没有找到，使用项目根目录文件夹名称：
```bash
# 提取目录名称
project_name=$(basename /path/to/project)
```

### 2.3 英文翻译为中文

如果项目名称是英文，翻译为中文：

| 英文名称 | 中文翻译 |
|----------|----------|
| hospital_management | 康养管理端 |
| ai_assistant | AI助理 |
| admin_panel | 管理后台 |
| user_portal | 用户端 |
| backend_api | 后端服务 |
| frontend | 前端 |
| backend | 后端 |
| mobile_app | 移动端 |

**翻译规则**：
- 移除下划线，转为空格
- 根据上下文语义翻译
- 常见技术词汇保留英文或使用行业通用翻译

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
