# API Generator 快速参考

## 文档处理速查（核心）

**⚠️ 文档是唯一真相来源**

当用户提供文档路径时：
1. **必须先读取文档** — 使用 Read 工具读取完整内容
2. **严格按照文档生成** — 不要凭记忆或猜测修改任何内容
3. **逐字段核对** — 生成后回文档对比接口地址、参数、响应字段

**反面教材**：
- ❌ 没读文档就生成
- ❌ 文档是 `/api/v2/user`，生成时写成 `/api/v1/user`
- ❌ 文档字段是 `user_name`，生成时写成 `username`

## Base URL 速查

**不要硬编码常量列表！** 先读取项目中的 `src/constants/base-url.ts` 文件获取实际常量，再根据接口路径匹配合适的常量。

如果项目中没有 base URL 常量文件，直接使用硬编码的 URL 前缀即可。

## 命名规则速查

| 项目 | 规则 | 示例 |
|------|------|------|
| API函数名 | `fetch` + 动词 + 语义化名词 | `fetchGetAppList` |
| 请求类型 | 动词 + 语义化名词 + `Req` | `GetAppListReq` |
| 响应类型 | 动词 + 语义化名词 + `Res` | `GetAppListRes` |
| 命名空间 | PascalCase模块名 | `ApplicationManage` |
| 文件名 | kebab-case | `application-manage.ts` |

## 语义化命名速查（核心）

**根据接口的业务语义命名，不要机械拼接 URL 路径！**

| 业务语义 | 函数名格式 | 类型名格式 | 示例 |
|---------|-----------|-----------|------|
| 列表查询（分页） | `fetchGet{Name}List` | `Get{Name}ListReq/Res` | `fetchGetAppList` |
| 详情查询（单条） | `fetchGet{Name}Detail` | `Get{Name}DetailReq/Res` | `fetchGetAppDetail` |
| 新增/创建 | `fetchAdd{Name}` | `Add{Name}Req/Res` | `fetchAddApp` |
| 编辑/更新信息 | `fetchEdit{Name}Info` | `Edit{Name}InfoReq/Res` | `fetchEditAppInfo` |
| 删除 | `fetchDelete{Name}` | `Delete{Name}Req/Res` | `fetchDeleteApp` |
| 状态变更 | `fetchUpdate{Name}Status` | `Update{Name}StatusReq/Res` | `fetchUpdateAppStatus` |
| 导出 | `fetchExport{Name}` | `Export{Name}Req/Res` | `fetchExportAppList` |
| 搜索/筛选 | `fetchSearch{Name}` | `Search{Name}Req/Res` | `fetchSearchApp` |
| 统计/汇总 | `fetchGet{Name}Statistics` | `Get{Name}StatisticsReq/Res` | `fetchGetAppStatistics` |

**{Name} 选取原则**：取最简洁的业务实体名，不重复模块名。
- 模块 `ApplicationManage` → `{Name}` 用 `App`，不用 `MainApp`
- 模块 `UserManage` → `{Name}` 用 `User`，不用 `ManageUser`

## 分页 vs 非分页

```typescript
// 分页请求 - 继承 CommonSearchParams
type GetListReq = Common.CommonSearchParams &
  CommonType.RecordNullable<{
    keyword?: string;
  }>;

// 分页响应 - 使用 PaginatingQueryRecord
type GetListRes = Common.PaginatingQueryRecord<{
  id: string;
  name: string;
}>;

// 非分页请求 - 直接定义
type GetDetailReq = {
  id: string;
};

// 非分页响应 - 使用 PublicResponse
type GetDetailRes = Common.PublicResponse<{
  id: string;
  name: string;
}>;
```

## HTTP 方法规则（重要）

**不要从接口名称推断 HTTP 方法！** 接口名称含"获取"、"查询"等词 ≠ GET。

- 用户明确指定 → 按用户说的
- 用户没指定 → **一律 POST**（本项目后端统一用 POST 传参，包括查询接口）

## 代码生成清单

### 文档来源接口（用户提供了文档路径）
- [ ] 读取文档完整内容
- [ ] 解析接口地址、请求参数、响应字段
- [ ] 向用户确认解析结果
- [ ] 严格按照文档生成代码
- [ ] **逐字段核对文档**（接口地址、参数名、字段类型、可选性）
- [ ] 确认base URL
- [ ] 确认目标文件（API函数文件 + 类型声明文件）
- [ ] 转换类型命名（Req/Res后缀）
- [ ] 清理原始类型（移除索引签名、包装字段）
- [ ] 生成API函数代码
- [ ] 生成类型声明代码
- [ ] 更新index.ts导出（如果是新文件）
- [ ] 提示用户运行类型检查

### 直接粘贴接口信息
- [ ] 确认base URL
- [ ] 确认目标文件（API函数文件 + 类型声明文件）
- [ ] 转换类型命名（Req/Res后缀）
- [ ] 清理原始类型（移除索引签名、包装字段）
- [ ] 生成API函数代码
- [ ] 生成类型声明代码
- [ ] 更新index.ts导出（如果是新文件）
- [ ] 提示用户运行类型检查
