# 增删改查示例

## 指南

接下来，我们将完成一个标准的 CRUD 业务。

### 定义类型

首先，我们要定义数据实体类型，表单类型。

目录如下：

```
├── src
│   ├── types                 # 全局类型
│   │   └── example           # 业务模块
│   │       └── template.ts   # 示例模版类型文件
```

示例模版有 id、title（标题）、description（描述）、created_at（创建时间） 和 updated_at（更新时间） 字段。

#### 实体类型

我们定义示例模版 `ExampleTemplate` 的类型：

```ts
/**
 * 示例模版
 */
type ExampleTemplate = {
  id: number;

  /** 标题 */
  title: string;

  /** 描述 */
  description: string;

  created_at: string;
  updated_at: string;
};
```

#### 表单类型

我们还需一个表单类型，用于创建和编辑：

```ts
/**
 * 示例模版表单
 */
type ExampleTemplateForm = {
  /** 标题 */
  title: string;

  /** 描述 */
  description?: string;
};
```

`description?` 中的 `?` 表示这个字段是可选的。

#### 查询参数类型

除此之外我们还需要一个查询参数类型，我们需要通过 title 字段筛选列表：

```ts
/**
 * 示例模版查询参数
 */
type ExampleTemplateQueryParams = RequestPagination & {
  /** 标题 */
  title?: string; // 查询参数是可选的，所以我们要加上 ?
};
```

这里我们继承了 `RequestPagination` 类型，`RequestPagination` 是我们标准的分页参数类型。

#### 导出类型

最后，通过 `export type` 导出我们的类型：

```ts
export type {ExampleTemplate, ExampleTemplateForm, ExampleTemplateQueryParams};
```

### API 请求

项目使用了 [TanStack Query](https://tanstack.com/query/latest/docs/framework/react/overview) 管理 API
请求以及请求返回的数据。我们需要分别创建**请求方法**和**请求 Hooks**。

目录如下：

```
├── src
│   ├── api                      # api 请求
│   │   ├── query-fn             # 请求方法
│   │   │   └── example          # 业务模块
│   │   │       └── template.ts  # 示例模版请求方法
│   │   └── query-hooks          # 请求 Hooks
│   │       └── example          # 业务模块
│   │           └── template.ts  # 示例模版请求 Hooks
```

#### 请求方法

项目使用了 [Axios](https://axios-http.com) 来封装 Http 请求方法 `request`，它接受一个范型作为响应结果的类型。

```ts
/**
 * 获取 示例模版（分页）
 */
export function getExampleTemplates(params?: ExampleTemplateQueryParams) {
  return request<ApiResponsePage<ExampleTemplate>>({
    url: '/example/template',
    method: 'GET',
    params,
  });
}

/**
 * 获取 单个示例模版
 */
export function getExampleTemplate(id: number) {
  return request<ExampleTemplate>({
    url: `/example/template/${id}`,
    method: 'GET',
  });
}

// ...其他参考代码总览
```

如果我们的响应结果是带分页的，则需要再包裹一个 `ApiResponsePage` 类型：

```ts title="src/types/common/result.ts"
/**
 * api 响应结果(分页)
 */
interface ApiResponsePage<T> {
  /** 数据 */
  data: T[];

  /** 分页信息 */
  meta?: ApiResponseMeta;
}
```

#### 请求 Hooks

TanStack Query 可以帮我们管理响应数据（异步状态管理），我们发送创建请求成功后，将列表数据设为过期，TanStack Query
会自动发送请求更新数据：

```ts
/**
 * 获取 示例模版
 */
export function useExampleTemplates(params?: ExampleTemplateQueryParams) {
  return useQuery({
    queryKey: ['/example/template', params],
    queryFn: () => getExampleTemplates(params),
    select: (data) => data.data,
    placeholderData: keepPreviousData,
  });
}

/**
 * 创建 示例模版
 */
export function useCreateExampleTemplate() {
  const queryClient = useQueryClient();
  return useMutation({
    mutationFn: createExampleTemplate,
    onSuccess: () => {
      // 将列表数据设为过期
      // highlight-start
      queryClient.invalidateQueries({queryKey: ['/example/template']});
      // highlight-end
    },
  });
}

// ...其他参考代码总览
```

### 视图组件

目录如下：

```
├── src
│   ├── views                       
│   │   └── example                  # 业务模块
│   │       └── template            
│   │           ├── components
│   │           │   ├── Creator.tsx  # 创建表单
│   │           │   └── Editor.tsx   # 编辑表单
│   │           └── index.tsx        # 示例模版主视图组件
```

视图组件的名称以 View 结尾，可以防止与类型定义冲突：

```tsx title="src/views/example/template/index.tsx"
function ExampleTemplateView() {
    // ...
}
```

#### 高级表格组件 ProTable

本项目封装了一个高级表格组件 `<ProTable>` ，使用高级表格组件我们只需要定义表格列、筛选栏表单、工具栏即可，无需关注他们的交互及样式，简化了很多样板代码。

```tsx
<ProTable<ExampleTemplate, ExampleTemplateQueryParams>
  tableTitle="示例模版"
  columns={columns} // 表格列
  tableToolBarItems={tableToolBarItems} // 工具栏
  filterBarItems={filterBarItems} // 筛选栏
  useQueryHook={useQueryHook} // 查询方法
/>
```

ProTable 需要我们传入一个查询方法，该方法接受 pagination（分页）、filter（筛选）参数，返回查询 Hook：

```tsx
function useQueryHook({pagination, filter}: QueryDataParams<ExampleTemplateQueryParams>) {
  return useExampleTemplates({
    page: pagination?.current,
    per_page: pagination?.pageSize,
    ...filter,
  });
}
```

表格列定义：

```tsx
const columns: ProTableColumnsType<ExampleTemplate> = [
  {
    title: 'ID',
    dataIndex: 'id',
    key: 'id',
    width: 80,
    hidden: true, // 当 hidden 为 true 时表示默认隐藏
  },
  {
    title: '标题',
    dataIndex: 'title',
    key: 'title',
    width: 300,
  },
  
  // ...
];
```

筛选栏：

```tsx
const filterBarItems: ReactNode[] = [
  <Form.Item<ExampleTemplateQueryParams>
    name="title"
    label="标题"
  >
    <Input
      placeholder="请输入"
      allowClear
    />
  </Form.Item>,
];
```

表格工具栏：

```tsx
const tableToolBarItems: ReactNode = (
  <>
    <Button
      type="primary"
      icon={<PlusOutlined/>}
      onClick={() => creatorRef.current?.open()}
    >
      添加
    </Button>
  </>
);
```

### 路由

最后，我们要把视图组件挂在到路由上，这样我们就能访问页面了。

目录如下：

```
├── src
│   ├── router                # 全局路由
│   │   └── routes            # 路由定义模块
│   │       ├── example.tsx   # 示例路由
│   │       └── index.tsx
```

注意：导入组件时，我们使用 [`lazy()`](https://react.dev/reference/react/lazy) 方法对视图组件进行懒加载处理，可以优化我们项目访问的速度。

```tsx title="src/router/routes/example.tsx"
// highlight-start
const ExampleTemplateView = lazy(() => import('@/views/example/template'));
// highlight-end

const example: AppRoute[] = [
  {
    path: 'example',
    element: <BasicLayout />,
    handle: { title: '配送', icon: <AppstoreOutlined /> },
    children: [
      {
        path: 'template',
        element: <ExampleTemplateView />,
        handle: { title: '运费模版' },
      },
    ],
  },
];
```

然后将定义好的路由导入到 index 路由中：

```tsx title="src/router/routes/index.tsx"
const routes: AppRoute[] = [
    {
        path: '/',
        element: <Root />,
        redirect: 'home',
        children: [
            // ...
            
            // highlight-start
            ...example,
            // highlight-end
            
            // ...
        ],
    },
    
    // ...
];
```

## 代码总览

### type

```ts title="src/types/example/template.ts"
import type {RequestPagination} from '@/types/common/requestParam.ts';

/**
 * 示例模版
 */
type ExampleTemplate = {
  id: number;

  /** 标题 */
  title: string;

  /** 描述 */
  description: string;

  created_at: string;
  updated_at: string;
};

/**
 * 示例模版表单
 */
type ExampleTemplateForm = {
  /** 标题 */
  title: string;

  /** 描述 */
  description?: string;
};

/**
 * 示例模版查询参数
 */
type ExampleTemplateQueryParams = RequestPagination & {
  /** 标题 */
  title?: string;
};

export type {ExampleTemplate, ExampleTemplateForm, ExampleTemplateQueryParams};
```

### query-fn

```ts title="src/api/query-fn/example/template.ts"
import type {ApiResponsePage} from '@/types/common/result.ts';
import type {
  ExampleTemplate,
  ExampleTemplateForm,
  ExampleTemplateQueryParams,
} from '@/types/example/template';
import {request} from '@/common/http/request.ts';

/**
 * 获取 示例模版
 */
export function getExampleTemplates(params?: ExampleTemplateQueryParams) {
  return request<ApiResponsePage<ExampleTemplate>>({
    url: '/example/template',
    method: 'GET',
    params,
  });
}

/**
 * 获取 单个示例模版
 */
export function getExampleTemplate(id: number) {
  return request<ExampleTemplate>({
    url: `/example/template/${id}`,
    method: 'GET',
  });
}

/**
 * 创建 示例模版
 */
export function createExampleTemplate(data: ExampleTemplateForm) {
  return request({
    url: '/example/template',
    method: 'POST',
    data,
  });
}

/**
 * 更新 示例模版
 */
export function updateExampleTemplate({id, data}: { id: number; data: ExampleTemplateForm }) {
  return request({
    url: `/example/template/${id}`,
    method: 'POST',
    data,
  });
}

/**
 * 删除 示例模版
 */
export function deleteExampleTemplate(id: number) {
  return request({
    url: `/example/template/${id}/delete`,
    method: 'POST',
  });
}
```

### query-hooks

```ts title="src/api/query-hooks/example/template.ts"
import {
  createExampleTemplate,
  deleteExampleTemplate,
  getExampleTemplate,
  getExampleTemplates,
  updateExampleTemplate,
} from '@/api/query-fn/example/template.ts';
import {keepPreviousData, useMutation, useQuery, useQueryClient} from '@tanstack/react-query';
import type {ExampleTemplateQueryParams} from '@/types/example/template.ts';

/**
 * 获取 示例模版
 */
export function useExampleTemplates(params?: ExampleTemplateQueryParams) {
  return useQuery({
    queryKey: ['/example/template', params],
    queryFn: () => getExampleTemplates(params),
    select: (data) => data.data,
    placeholderData: keepPreviousData,
  });
}

/**
 * 获取 单个示例模版
 */
export function useExampleTemplate(id: number | null) {
  return useQuery({
    queryKey: [`/example/template/${id}`],
    queryFn: () => getExampleTemplate(id!),
    enabled: !!id,
    select: (data) => data.data,
  });
}

/**
 * 创建 示例模版
 */
export function useCreateExampleTemplate() {
  const queryClient = useQueryClient();
  return useMutation({
    mutationFn: createExampleTemplate,
    onSuccess: () => {
      queryClient.invalidateQueries({queryKey: ['/example/template']});
    },
  });
}

/**
 * 更新 示例模版
 */
export function useUpdateExampleTemplate() {
  const queryClient = useQueryClient();
  return useMutation({
    mutationFn: updateExampleTemplate,
    onSuccess: () => {
      queryClient.invalidateQueries({queryKey: ['/example/template']});
    },
  });
}

/**
 * 删除 示例模版
 */
export function useDeleteExampleTemplate() {
  const queryClient = useQueryClient();
  return useMutation({
    mutationFn: deleteExampleTemplate,
    onSuccess: (_, id) => {
      queryClient.invalidateQueries({queryKey: ['/example/template']});
      queryClient.removeQueries({queryKey: [`/example/template/${id}`]});
    },
  });
}
```

### table

```tsx title="src/views/example/index.tsx"
import type {ProTableColumnsType} from '@/components/pro/ProTable/types.ts';
import ProTable, {type QueryDataParams} from '@/components/pro/ProTable';
import {type ReactNode, useRef} from 'react';
import Creator, {type CreatorRef} from '@/views/example/template/components/Creator.tsx';
import Editor, {type EditorRef} from '@/views/example/template/components/Editor.tsx';
import {Button, Dropdown, Form, Input, Space} from 'antd';
import {EllipsisOutlined, PlusOutlined} from '@ant-design/icons';
import {PlusOutlined} from '@ant-design/icons';
import type {ExampleTemplate, ExampleTemplateQueryParams} from '@/types/example/template.ts';
import {
  useDeleteExampleTemplate,
  useExampleTemplates,
} from '@/api/query-hooks/example/template.ts';
import useCountdownConfirm from '@/components/business/confirmation/useCountdownConfirm.tsx';

function ExampleTemplateView() {
  const creatorRef = useRef<CreatorRef>(null);
  const editorRef = useRef<EditorRef>(null);
  const deleteExampleTemplate = useDeleteExampleTemplate();
  const confirmModal = useCountdownConfirm();

  function useQueryHook({pagination, filter}: QueryDataParams<ExampleTemplateQueryParams>) {
    return useExampleTemplates({
      page: pagination?.current,
      per_page: pagination?.pageSize,
      ...filter,
    });
  }

  function handleDelete(record: ExampleTemplate) {
    confirmModal.confirm(() => {
      deleteExampleTemplate.mutate(record.id);
    });
  }

  const columns: ProTableColumnsType<ExampleTemplate> = [
    {
      title: 'ID',
      dataIndex: 'id',
      key: 'id',
      width: 80,
      hidden: true,
    },
    {
      title: '标题',
      dataIndex: 'title',
      key: 'title',
      width: 300,
    },
    {
      title: '描述',
      dataIndex: 'description',
      key: 'description',
      width: 300,
    },
    {
      title: '创建时间',
      dataIndex: 'created_at',
      key: 'created_at',
      width: 180,
    },
    {
      title: '更新时间',
      dataIndex: 'updated_at',
      key: 'updated_at',
      minWidth: 180,
    },
    {
      title: '操作',
      key: 'action',
      width: 80,
      fixed: 'right',
      render: (_, record) => (
        <Space.Compact>
          <Button
            size="small"
            onClick={() => editorRef.current?.open(record.id)}
          >
            编辑
          </Button>
          <Dropdown
            trigger={['click']}
            menu={{
              style: { minWidth: '100px' },
              items: [
                {
                  key: 'delete',
                  label: '删除',
                  onClick: () => handleDelete(record),
                },
              ],
            }}
          >
            <Button
              size="small"
              icon={<EllipsisOutlined />}
            />
          </Dropdown>
        </Space.Compact>
      ),
    },
  ];

  // 筛选栏
  const filterBarItems: ReactNode[] = [
    <Form.Item<ExampleTemplateQueryParams>
      name="title"
      label="标题"
    >
      <Input
        placeholder="请输入"
        allowClear
      />
    </Form.Item>,
  ];

  // 表格工具栏
  const tableToolBarItems: ReactNode = (
    <>
      <Button
        type="primary"
        icon={<PlusOutlined/>}
        onClick={() => creatorRef.current?.open()}
      >
        添加
      </Button>
    </>
  );

  return (
    <>
      <ProTable<ExampleTemplate, ExampleTemplateQueryParams>
        tableTitle="示例模版"
        columns={columns}
        tableToolBarItems={tableToolBarItems}
        filterBarItems={filterBarItems}
        useQueryHook={useQueryHook}
      />

      <Creator ref={creatorRef}></Creator>
      <Editor ref={editorRef}></Editor>
    </>
  );
}

export default ExampleTemplateView;
```

### creator

```tsx title="src/views/example/template/components/Creator.tsx"
import {Button, Flex, Form, Input, Modal} from 'antd';
import {type Ref, useImperativeHandle, useState} from 'react';
import {useCreateExampleTemplate} from '@/api/query-hooks/example/template.ts';
import type {ExampleTemplateForm} from '@/types/example/template.ts';

type CreatorRef = {
  /** 打开 Modal 弹窗 */
  open: () => void;
};

type CreatorProps = {
  ref: Ref<CreatorRef>;
};

function Creator({ref}: CreatorProps) {
  const [form] = Form.useForm();

  const [isModalOpen, setIsModalOpen] = useState<boolean>(false);
  const createExampleTemplate = useCreateExampleTemplate();

  useImperativeHandle(ref, () => {
    return {
      open() {
        setIsModalOpen(true);
      },
    };
  }, []);

  function onFinish(e: ExampleTemplateForm) {
    createExampleTemplate.mutate(e, {
      onSuccess: () => {
        form.resetFields();
        setIsModalOpen(false);
      },
    });
  }

  return (
    <Modal
      title="添加示例模版"
      width={500}
      open={isModalOpen}
      maskClosable={false}
      confirmLoading={createExampleTemplate.isPending}
      onOk={() => form.submit()}
      onCancel={() => setIsModalOpen(false)}
      footer={(_, {OkBtn, CancelBtn}) => (
        <Flex gap={8}>
          <Button
            style={{marginRight: 'auto'}}
            onClick={() => form.resetFields()}
          >
            重置
          </Button>
          <CancelBtn/>
          <OkBtn/>
        </Flex>
      )}
    >
      <Form
        name="exampleTemplateCreator"
        form={form}
        layout="vertical"
        onFinish={onFinish}
        autoComplete="off"
      >
        <Form.Item<ExampleTemplateForm>
          label="标题"
          name="title"
          rules={[{required: true, message: '请填写标题'}]}
        >
          <Input/>
        </Form.Item>
        <Form.Item<ExampleTemplateForm>
          label="描述"
          name="description"
        >
          <Input/>
        </Form.Item>
      </Form>
    </Modal>
  );
}

export default Creator;

export type {CreatorRef};
```

### editor

```tsx title="src/views/example/template/components/Editor.tsx"
import {type Ref, useEffect, useImperativeHandle, useState} from 'react';
import {Form, Input, Modal} from 'antd';
import {
  useExampleTemplate,
  useUpdateExampleTemplate,
} from '@/api/query-hooks/example/template.ts';
import type {ExampleTemplateForm} from '@/types/example/template.ts';

type EditorRef = {
  /** 打开 Modal 弹窗 */
  open: (id: number) => void;
};

type EditorProps = {
  ref: Ref<EditorRef>;
};

function Editor({ref}: EditorProps) {
  const [form] = Form.useForm();

  const [isModalOpen, setIsModalOpen] = useState<boolean>(false);
  const [exampleTemplateId, setExampleTemplateId] = useState<number | null>(null);

  const {data, isFetching} = useExampleTemplate(exampleTemplateId);
  const updateUpdateExampleTemplate = useUpdateExampleTemplate();

  useImperativeHandle(ref, () => {
    return {
      open(id: number) {
        setExampleTemplateId(id);
        setIsModalOpen(true);
      },
    };
  }, []);

  useEffect(() => {
    if (data) {
      form.setFieldsValue({
        title: data.title,
        description: data.description,
      });
    }
  }, [data, form]);

  function onFinish(e: ExampleTemplateForm) {
    if (exampleTemplateId !== null) {
      updateUpdateExampleTemplate.mutate(
        {id: exampleTemplateId, data: e},
        {
          onSuccess: () => {
            form.resetFields();
            setIsModalOpen(false);
          },
        }
      );
    }
  }

  function handleReset() {
    form.resetFields();
    setExampleTemplateId(null);
  }

  return (
    <Modal
      title="编辑示例模版"
      width={500}
      open={isModalOpen}
      loading={isFetching}
      maskClosable={false}
      confirmLoading={updateUpdateExampleTemplate.isPending}
      onOk={() => form.submit()}
      onCancel={() => setIsModalOpen(false)}
      afterClose={handleReset}
    >
      <Form
        name="exampleTemplateEditor"
        form={form}
        layout="vertical"
        onFinish={onFinish}
        autoComplete="off"
      >
        <Form.Item<ExampleTemplateForm>
          label="标题"
          name="title"
          rules={[{required: true, message: '请填写标题'}]}
        >
          <Input/>
        </Form.Item>
        <Form.Item<ExampleTemplateForm>
          label="描述"
          name="description"
        >
          <Input/>
        </Form.Item>
      </Form>
    </Modal>
  );
}

export default Editor;

export type {EditorRef};
```

### route

```tsx title="src/router/routes/example.tsx"
import type { AppRoute } from '@/router/types.ts';
import BasicLayout from '@/layouts/basic-layout';
import { lazy } from 'react';
import { AppstoreOutlined } from '@ant-design/icons';

const ExampleTemplateView = lazy(() => import('@/views/example/template'));

const example: AppRoute[] = [
  {
    path: 'example',
    element: <BasicLayout />,
    handle: { title: '配送', icon: <AppstoreOutlined /> },
    children: [
      {
        path: 'template',
        element: <ExampleTemplateView />,
        handle: { title: '运费模版' },
      },
    ],
  },
];

export default example;
```
