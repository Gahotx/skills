# 现有代码示例参考

> **重要**：所有命名必须遵循语义化规则，参考 SKILL.md 中的 4.2 节。
> 不要机械拼接 URL 路径，要根据接口的业务语义来命名。

> **注意**：以下示例中的 base URL 常量（如 `humanBaseUrl`、`dispatchBaseUrl`）和命名空间名（如 `ClusterSituation`、`ApplicationManage`）仅作为代码风格参考。实际生成时，必须先读取目标项目的 `src/constants/base-url.ts` 获取真实的 base URL 常量。

---

## 命名正误对照

### 列表查询接口

接口路径：`/app/search/main`，业务：查询主应用列表

```typescript
// ❌ 错误命名：机械拼接 URL 路径中的 search + main
export function fetchSearchMainApp(data: ...) { ... }
type SearchMainAppReq = { ... };

// ✅ 正确命名：语义化，列表查询用 List 后缀
export function fetchGetAppList(data: Api.ApplicationManage.GetAppListReq) { ... }
type GetAppListReq = { ... };
type GetAppListRes = Common.PaginatingQueryRecord<{ ... }>;
```

### 详情查询接口

接口路径：`/app/query`，业务：查询应用详情

```typescript
// ❌ 错误命名：Query 不是语义化动词
export function fetchQueryApp(data: ...) { ... }
type QueryAppReq = { ... };

// ✅ 正确命名：详情查询用 Detail 后缀
export function fetchGetAppDetail(data: Api.ApplicationManage.GetAppDetailReq) { ... }
type GetAppDetailReq = { ... };
type GetAppDetailRes = Common.PublicResponse<{ ... }>;
```

### 编辑/更新接口

接口路径：`/app/edit/main`，业务：修改主应用基本信息

```typescript
// ❌ 错误命名：缺少 Info 后缀，且冗余的 Main
export function fetchEditMainApp(data: ...) { ... }
type EditMainAppReq = { ... };

// ✅ 正确命名：编辑信息用 Info 后缀，不重复模块名
export function fetchEditAppInfo(data: Api.ApplicationManage.EditAppInfoReq) { ... }
type EditAppInfoReq = { ... };
type EditAppInfoRes = Common.PublicResponse<boolean>;
```

---

## 示例1：人群态势查询（非分页接口）

> **命名说明**：命名空间已经是 `ClusterSituation`，所以函数名和类型名中省略模块前缀，只保留核心语义 `GetQuestionnaireData`。

### API函数 (`src/service/api/cluster-situation.ts`)

```typescript
import { humanBaseUrl } from '@/constants/base-url';
import { request } from '../request';

/** 人群态势查询-问卷信息 */
export function fetchGetQuestionnaireData(
  data: Api.ClusterSituation.GetQuestionnaireDataReq
) {
  return request<Api.ClusterSituation.GetQuestionnaireDataRes>({
    url: `${humanBaseUrl}/human/attribute/crowd/situation/questionnaire`,
    method: 'post',
    data
  });
}
```

### 类型声明 (`src/typings/api.d.ts`)

```typescript
/** 人群态势 */
namespace ClusterSituation {
  /** 人群态势查询-问卷信息 - 请求参数 */
  type GetQuestionnaireDataReq = {
    /** 时间格式 2006-01-02 */
    end_date: string;
    /** 时间格式 2006-01-02 */
    start_date: string;
    /** 时间纬度，根据不同纬度其统计逻辑会不同 */
    time_dimension: 'year' | 'quarter' | 'month';
  };

  /** 人群态势查询-问卷信息 - 响应数据 */
  type GetQuestionnaireDataRes = Common.PublicResponse<{
    /** 最新数据时间 */
    latest_data_time?: string;
    /** 心理指标数据 */
    psychological_indicator_data?: PsychologicalIndicatorData;
    /** 量表覆盖概况数据 */
    questionnaire_coverage_overview_data?: QuestionnaireCoverageOverviewData;
    // ... 其他字段
  }>;

  /** 心理指标数据 */
  type PsychologicalIndicatorData = {
    /** 堆叠系列数据 */
    xAxisDataList: XAxisDataList[];
    /** 量表名称纵坐标 */
    yAxisData: string[];
  };

  /** 堆叠系列数据项 */
  type XAxisDataList = {
    /** 颜色 */
    color: string;
    /** 各量表指标对应的数值 */
    data: number[];
    /** 风险等级（高风险/中风险/低风险/无风险） */
    name: string;
  };

  // ... 其他嵌套类型
}
```

---

## 示例2：用户管理（分页 + 非分页）

> 展示分页查询和新增操作的典型模式，其他CRUD操作（详情/编辑/删除）结构类似。

### API函数 (`src/service/api/user-manage.ts`)

```typescript
import { userBaseUrl } from '@/constants/base-url';
import { request } from '../request';

/** 获取用户列表（分页） */
export function fetchGetUserList(data: Api.UserManage.GetUserListReq) {
  return request<Api.UserManage.GetUserListRes>({
    url: `${userBaseUrl}/user/list`,
    method: 'post',
    data
  });
}

/** 添加用户 */
export function fetchAddUser(data: Api.UserManage.AddUserReq) {
  return request<Api.UserManage.AddUserRes>({
    url: `${userBaseUrl}/user/add`,
    method: 'post',
    data
  });
}
```

### 类型声明 (`src/typings/api.d.ts`)

```typescript
/** 用户管理 */
namespace UserManage {
  /** 获取用户列表 - 请求参数（分页） */
  type GetUserListReq = Common.CommonSearchParams &
    CommonType.RecordNullable<{
      /** 搜索关键词 */
      search_keyword?: string;
      /** 用户状态 */
      status?: number;
    }>;

  /** 获取用户列表 - 响应数据 */
  type GetUserListRes = Common.PaginatingQueryRecord<{
    /** 用户ID */
    id: string;
    /** 用户名 */
    username: string;
    /** 手机号 */
    phone: string;
    /** 状态 1-启用 0-禁用 */
    status: number;
  }>;

  /** 添加用户 - 请求参数 */
  type AddUserReq = {
    /** 用户名 */
    username: string;
    /** 手机号 */
    phone: string;
    /** 邮箱 */
    email?: string;
  };

  /** 添加用户 - 响应数据 */
  type AddUserRes = Common.PublicResponse<{
    /** 新增用户ID */
    id: string;
  }>;
}
```

---

## 示例3：带文件下载的接口

### API函数

```typescript
/** 导出用户数据 */
export function fetchExportUsers(data: Api.UserManage.ExportUsersReq) {
  return request<Api.UserManage.ExportUsersRes>({
    url: `${userBaseUrl}/user/export`,
    method: 'post',
    data,
    responseType: 'blob',
    timeout: 120000
  });
}
```

### 类型声明

```typescript
/** 导出用户数据 - 请求参数 */
type ExportUsersReq = CommonType.RecordNullable<{
  /** 搜索关键词 */
  keyword?: string;
  /** 状态筛选 */
  status?: number;
  /** 导出格式 */
  format: 'xlsx' | 'csv';
}>;

/** 导出用户数据 - 响应数据（Blob） */
type ExportUsersRes = Blob;
```

---

## 示例4：应用管理（CRUD 全套，语义化命名）

> **重点**：模块命名空间 `ApplicationManage` 已包含"管理"含义，函数名和类型名中只需用 `App` 作为实体名，不加 `Main`、不加 `Manage`。

### API函数 (`src/service/api/application-manage.ts`)

```typescript
import { dispatchBaseUrl } from '@/constants/base-url';
import { request } from '../request';

/** 主应用列表 */
export function fetchGetAppList(data: Api.ApplicationManage.GetAppListReq) {
  return request<Api.ApplicationManage.GetAppListRes>({
    url: `${dispatchBaseUrl}/app/search`,
    method: 'post',
    data
  });
}

/** 应用详情查询 */
export function fetchGetAppDetail(data: Api.ApplicationManage.GetAppDetailReq) {
  return request<Api.ApplicationManage.GetAppDetailRes>({
    url: `${dispatchBaseUrl}/app/query`,
    method: 'post',
    data
  });
}

/** 修改主应用基本信息 */
export function fetchEditAppInfo(data: Api.ApplicationManage.EditAppInfoReq) {
  return request<Api.ApplicationManage.EditAppInfoRes>({
    url: `${dispatchBaseUrl}/app/update`,
    method: 'post',
    data
  });
}
```

### 类型声明 (`src/typings/api.d.ts`)

```typescript
/** 应用管理 */
namespace ApplicationManage {
  /** 应用版本项 */
  type AppVersionItem = {
    /** 应用版本ID */
    app_version_id: string;
    /** 版本logo URL */
    logo_url: string;
    /** 资源数量 */
    resource_count: number;
    /** 版本名称 */
    version_name: string;
  };

  /** 主应用列表项 */
  type AppListItem = {
    /** 应用名称 */
    app_name: string;
    /** 应用URL */
    app_url: string;
    /** 创建时间 */
    created_at: string;
    /** 应用logo URL */
    logo_url: string;
    /** 主应用ID */
    main_app_id: string;
    /** 版本号 */
    version_code: string;
    /** 应用版本列表 */
    versions: AppVersionItem[];
  };

  /** 主应用列表 - 请求参数 */
  type GetAppListReq = {
    /** 应用名称关键字 */
    keyword: string;
  };

  /** 主应用列表 - 响应数据 */
  type GetAppListRes = Common.PublicResponse<AppListItem[]>;

  /** 应用详情查询 - 请求参数 */
  type GetAppDetailReq = {
    /** 主应用ID */
    main_app_id: string;
  };

  /** 应用详情查询 - 响应数据 */
  type GetAppDetailRes = Common.PublicResponse<{
    /** 应用名称 */
    app_name: string;
    /** 应用URL */
    app_url: string;
    /** 应用描述 */
    description: string;
    /** 应用logo URL */
    logo_url: string;
    /** 应用版本列表 */
    versions: AppVersionItem[];
  }>;

  /** 修改主应用基本信息 - 请求参数 */
  type EditAppInfoReq = {
    /** 应用名称 */
    app_name: string;
    /** 应用描述 */
    description?: string;
    /** 帮助文档URL */
    help_doc_url: string;
    /** 应用logo URL */
    logo_url: string;
  };

  /** 修改主应用基本信息 - 响应数据 */
  type EditAppInfoRes = Common.PublicResponse<null>;
}
```
