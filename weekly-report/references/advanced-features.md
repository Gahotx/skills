# 高级功能

## 多仓库支持

支持从多个项目中提取 commit 记录，合并生成一份周报。

### 使用场景

- 同时参与多个项目的开发
- 前后端分离项目
- 多个微服务项目

### 路径格式规范

支持以下路径格式：

| 格式 | 示例 | 说明 |
|------|------|------|
| 绝对路径 | `E:/Projects/frontend` | 完整路径 |
| 绝对路径（反斜杠） | `E:\Projects\backend` | Windows 风格 |
| 相对路径 | `./frontend` | 相对于当前目录 |
| 多个项目 | `E:/Projects/frontend, E:/Projects/backend` | 逗号分隔 |

### 项目名称解析

对于每个项目，按以下优先级获取显示名称：

1. **从 .env 文件读取**
   - 检查 `.env`、`.env.local`、`.env.development` 等文件
   - 查找 `APP_TITLE`、`PROJECT_NAME`、`VITE_APP_TITLE` 等变量

2. **使用目录名称**
   - 如果 .env 中没有找到，使用项目根目录文件夹名称

3. **英文翻译为中文**
   - `hospital_management` → 康养管理端
   - `ai_assistant` → AI助理
   - `admin_panel` → 管理后台

---

## Commit Message 智能处理

### 1. 移除类型前缀

自动识别并移除 conventional commit 前缀：
```
feat: 新增功能 → 新增功能
fix(scope): 修复bug → 修复bug
feat(auth): implement JWT → implement JWT
```

### 2. 翻译英文 Commit（可选）

将英文 commit message 翻译为中文：
```
implement user login → 实现用户登录
fix pagination bug → 修复分页bug
```

### 3. 合并相似 Commit（可选）

将同一功能的多个 commit 合并为一条工作项：
```
# 原始 commits
feat: add login form
feat: add login validation
feat: add login API integration

# 合并后
- 实现用户登录功能（包含表单、验证、API集成）
```

---

## 空项目处理

如果某个项目在指定日期范围内没有 commit 记录：

- **按项目分类**：不显示该项目
- **按类型分类**：不显示该项目的条目

如果所有项目都没有 commit 记录：
1. 提示用户确认日期范围是否正确
2. 提示用户确认项目路径是否正确
3. 提示用户确认是否在正确的分支上

---

## 错误处理

### 1. 非 Git 项目

如果项目目录不是 Git 仓库：
- 跳过该项目
- 提示用户该项目不是 Git 仓库
- 继续处理其他项目

### 2. 路径不存在

如果用户指定的项目路径不存在：
- 提示用户路径不存在
- 询问是否重新输入或跳过该项目

### 3. 日期格式错误

如果用户输入的日期格式不正确：
- 提示正确的日期格式（YYYY-MM-DD）
- 询问是否重新输入或使用默认日期

### 4. 权限问题

如果无法读取项目目录或写入周报文件：
- 提示权限不足
- 建议用户检查文件权限

---

## 扩展建议

### 1. 自定义 Commit 类型映射

允许用户自定义 commit 前缀到分类的映射关系：
```json
{
  "feat": "功能开发",
  "fix": "Bug修复",
  "custom_prefix": "自定义分类"
}
```

### 2. 周报模板自定义

允许用户自定义周报模板格式：
- 修改标题格式
- 添加/删除章节
- 调整内容组织方式

### 3. 历史周报对比

对比本周和上周的周报：
- 统计工作量变化
- 识别持续进行的任务
- 分析工作重点转移

### 4. 自动发送

将生成的周报自动发送到指定邮箱或团队协作工具。
