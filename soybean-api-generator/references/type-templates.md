# 类型声明模板参考

## 通用类型定义位置

以下类型定义在 `src/typings/api.d.ts` 的 `Api.Common` 命名空间中：

```typescript
namespace Common {
  /** 通用响应包装 */
  interface PublicResponse<T = any> {
    result?: T;
  }

  /** 分页参数 */
  interface PaginatingCommonParams {
    page: number;
    size: number;
    total: number;
  }

  /** 分页查询参数 */
  type CommonSearchParams = Pick<PaginatingCommonParams, 'page' | 'size'>;

  /** 分页查询响应 */
  interface PaginatingQueryRecord<T = any> extends PaginatingCommonParams {
    result?: T[];
  }

  /** 带ID的记录类型 */
  type CommonRecord<T = any> = {
    id: string;
  } & T;
}
```

## 常用工具类型

定义在 `src/typings/common.d.ts` 的 `CommonType` 命名空间中：

```typescript
namespace CommonType {
  /** 将所有属性变为可选且允许null */
  type RecordNullable<T> = {
    [K in keyof T]?: T[K] | null;
  };

  /** 选项类型（用于下拉框等） */
  type Option<V = string> = {
    label: string;
    value: V;
  };
}
```

## 命名空间模板

```typescript
// src/typings/api.d.ts

declare namespace Api {
  // ... 其他命名空间 ...

  /** 模块中文名 */
  namespace ModuleName {
    /** 接口描述 - 请求参数 */
    type GetSomethingReq = {
      /** 字段说明 */
      field_name: string;
    };

    /** 接口描述 - 响应数据 */
    type GetSomethingRes = Common.PublicResponse<{
      /** 字段说明 */
      result_field: string;
    }>;

    // 如果是分页接口
    /** 分页查询 - 请求参数 */
    type GetListReq = Common.CommonSearchParams &
      CommonType.RecordNullable<{
        /** 搜索关键词 */
        keyword?: string;
        /** 状态筛选 */
        status?: number;
      }>;

    /** 分页查询 - 响应数据 */
    type GetListRes = Common.PaginatingQueryRecord<{
      /** ID */
      id: string;
      /** 名称 */
      name: string;
    }>;
  }
}
```

## API函数模板

```typescript
// src/service/api/module-name.ts
// {baseUrl常量名} 从项目的 src/constants/base-url.ts 中读取

import { {baseUrl常量名} } from '@/constants/base-url';
import { request } from '../request';

/** 接口中文描述 */
export function fetchGetSomething(data: Api.ModuleName.GetSomethingReq) {
  return request<Api.ModuleName.GetSomethingRes>({
    url: `${{baseUrl常量名}}/api/path`,
    method: 'post',
    data
  });
}

// 分页查询示例
/** 分页查询列表 */
export function fetchGetList(data: Api.ModuleName.GetListReq) {
  return request<Api.ModuleName.GetListRes>({
    url: `${{baseUrl常量名}}/api/list`,
    method: 'post',
    data
  });
}

// 带额外配置示例
/** 下载文件 */
export function fetchDownloadFile(data: Api.ModuleName.DownloadReq) {
  return request<Api.ModuleName.DownloadRes>({
    url: `${{baseUrl常量名}}/api/download`,
    method: 'post',
    data,
    responseType: 'blob',
    timeout: 60000
  });
}
```

## 项目文件结构

```
src/
├── service/
│   ├── api/
│   │   ├── index.ts              # 统一导出
│   │   ├── auth.ts               # 认证相关
│   │   ├── cluster-situation.ts  # 人群态势
│   │   ├── user-manage.ts        # 用户管理
│   │   └── ...
│   └── request/
│       └── index.ts              # request工具
├── typings/
│   ├── api.d.ts                  # API类型定义
│   └── common.d.ts               # 通用工具类型
└── constants/
    └── base-url.ts               # Base URL常量
```
