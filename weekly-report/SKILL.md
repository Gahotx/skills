---
name: weekly-report
description: "生成工作周报。当用户提到'周报'、'工作周报'、'weekly report'、'写周报'、'生成周报'等关键词时必须使用此技能。该技能会从git项目中提取commit记录，自动生成结构化的周报文档。"
metadata:
  author: gahotx
  version: "1.0"
---

# 周报生成器 (Weekly Report Generator)

从 Git 项目中提取 commit 记录，自动生成结构化的工作周报。

## 使用场景

当用户请求以下内容时激活此 skill：
- "生成周报"、"写周报"、"本周工作总结"
- "汇总这周的提交"、"总结代码变更"
- "生成工作报告"、"导出开发日志"

## 参考文档

| 文档 | 说明 |
|------|------|
| [参数配置](./references/parameters.md) | 必要参数和可选参数说明 |
| [执行流程](./references/execution-flow.md) | 数据收集和分析步骤 |
| [输出格式](./references/output-format.md) | 周报 Markdown 模板 |
| [自定义选项](./references/custom-options.md) | 用户确认项和可选配置 |
| [高级功能](./references/advanced-features.md) | 多仓库支持和智能增强 |

## 交互流程

生成周报的完整流程共 7 步，详细配置选项见 [自定义选项](./references/custom-options.md)：

1. **确认日期范围** - 默认最近5天，或选择最近一周，用户可自定义
2. **确认项目列表** - 默认当前目录，支持多项目
3. **选择格式类型** - 默认按项目分类，或按commit类型分类
4. **确认保存位置** - 默认 `桌面/周报/`
5. **生成周报** - 提取 commit 记录并生成
6. **补充内容** - 询问额外事项和下周计划
7. **保存文件** - 按结束日期命名

## 注意事项

- 所有与用户的交互必须使用中文
- 日期格式统一使用 `YYYY-MM-DD`，周报标题格式为 `YYYY.MM.DD - YYYY.MM.DD`
- 保存 markdown 文件后，额外输出纯文本版本方便复制（详见 [输出格式](./references/output-format.md)）
- 空内容处理规则详见 [高级功能](./references/advanced-features.md)
- **按项目分类时**：项目下直接列工作内容，不要加"功能开发"等子标题（详见 [输出格式](./references/output-format.md)）
- **Commit 精炼**：不要直接罗列 commit，需合并相似提交、提炼核心内容（详见 [执行流程](./references/execution-flow.md)）
- **多项目计划**：多个项目时分别询问每个项目的下周计划（详见 [自定义选项](./references/custom-options.md)）
