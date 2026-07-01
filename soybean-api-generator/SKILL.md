---
name: soybean-api-generator
description: 根据接口信息自动生成API调用函数和类型声明代码。当用户提供接口名称、接口地址、原始Apifox生成的类型声明，或提供接口文档路径（如.md、.json、.ts文件）时使用此skill。适用于需要新增后端API调用的场景，包括但不限于：新增接口、批量导入接口、对接新模块的API、按照文档生成接口代码。触发词：生成接口、添加API、新建接口、接口类型、API调用、对接接口、按照文档生成。
---

# API Generator Skill

本skill用于根据用户提供的接口信息，自动生成符合项目规范的API调用函数和TypeScript类型声明代码。

## 工作流程

### 1. 收集接口信息

从用户输入中提取：**接口名称**、**接口地址**、**原始类型声明**、**HTTP方法**（默认 `post`）

#### 文档路径处理

当用户提供文档路径时，**必须先读取文档内容**，不能凭记忆生成。

**核心原则：文档是唯一真相来源**
1. **先读取**：使用 Read 工具读取文档完整内容
2. **再解析**：提取接口地址、请求参数、响应字段
3. **后生成**：严格按照文档内容生成，不添加/修改字段

> **HTTP 方法规则**：不要从接口名称推断方法。用户没指定 → **一律 POST**（本项目后端统一用 POST 传参）

### 2. 确定Base URL

读取项目中的 `src/constants/base-url.ts` 获取可用常量列表，展示给用户选择。如果文件不存在，询问用户或使用硬编码前缀。

### 3. 确定目标文件

- **API函数**：`src/service/api/{module-name}.ts`（按模块命名，如 `cluster-situation.ts`）
- **类型声明**：`src/typings/api.d.ts`（所有类型在 `Api` 命名空间下）

### 4. 转换类型声明

将原始类型转换为项目规范格式：

```typescript
// 原始格式 → 项目格式
export interface Request { ... }  →  namespace 模块名 {
export interface Response { ... }     type 接口名Req = { ... };
                                       type 接口名Res = { ... };
                                     }
```

**命名规则**：`fetch` + 动词 + 语义化名词（按业务语义命名，非URL路径直译）

> 详细命名规则和类型模板见 [references/quick-reference.md](references/quick-reference.md) 和 [references/type-templates.md](references/type-templates.md)

### 5. 生成代码

**API函数模板**：
```typescript
import { {baseUrl常量名} } from '@/constants/base-url';
import { request } from '../request';

/** 接口中文描述 */
export function fetchGetSomething(data: Api.ModuleName.GetSomethingReq) {
  return request<Api.ModuleName.GetSomethingRes>({
    url: `${{baseUrl常量名}}/接口路径`,
    method: 'post',
    data
  });
}
```

> 完整代码模板见 [references/type-templates.md](references/type-templates.md)

### 6. 更新导出

如果API函数文件是新建的，在 `src/service/api/index.ts` 中添加：
```typescript
export * from './新模块名';
```

### 7. 验证生成结果

**文档一致性检查**：逐字段核对接口地址、请求参数、响应字段是否与文档一致

**代码质量检查**：类型语法、命名空间层级、导入语句，提示用户运行 `pnpm typecheck`

## 核心规则速查

| 规则 | 说明 |
|------|------|
| 文档优先 | 提供文档路径时，文档是唯一真相来源 |
| HTTP方法 | 默认 POST，除非用户明确指定 GET |
| 命名语义化 | 按业务语义命名，不机械拼接URL路径 |
| 分页处理 | 请求继承 `Common.CommonSearchParams`，响应用 `PaginatingQueryRecord<T>` |
| 类型清理 | 移除索引签名、`code`/`message` 包装字段，保留业务字段 |

## 参考文件

- [quick-reference.md](references/quick-reference.md) — 命名规则速查、文档处理清单
- [type-templates.md](references/type-templates.md) — 通用类型定义、代码模板
- [existing-examples.md](references/existing-examples.md) — 现有代码示例、命名正误对照

## 用户输入格式

**格式1：直接粘贴**
```
接口名称：{中文描述}
接口地址：{URL路径}
原始类型声明：{粘贴代码}
```

**格式2：提供文档路径**
```
按照 {文档路径} 生成接口代码
```

收到文档路径后：读取 → 解析 → 确认 → 生成 → 核对
