# Gahotx's Skills

个人 Claude Code Skills 集合，用于提升开发效率。

## 一键安装

```bash
npx skills add Gahotx/skills --skill='*' -g
```

## 包含的 Skills

### [git-commit-cn](./git-commit-cn/)

```bash
npx skills add Gahotx/skills --skill git-commit-cn
```

使用约定式提交规范生成中文 git commit 信息。

- 根据代码变更自动检测提交类型和范围
- 生成符合 Conventional Commits 的中文提交信息
- 支持自定义类型、范围和描述

### [weekly-report](./weekly-report/)

```bash
npx skills add Gahotx/skills --skill weekly-report
```

从 Git 项目提取 commit 记录，自动生成结构化的工作周报。

- 支持多项目批量提取
- 按项目或按类型分类输出
- 自动生成 Markdown 格式周报

## 使用

在 Claude Code 中直接描述需求即可触发对应的 skill：

- **git-commit-cn**: "帮我提交代码"、"生成 commit 信息"
- **weekly-report**: "写周报"、"生成本周工作周报"

## 许可

MIT
