# 依赖

## 如何升级依赖

:::warning

请不要使用 `pnpm update` 一次性更新全部依赖！

:::

你应该明确知道要升级的依赖及其版本，一个一个的单独进行升级。使用 `pnpm` 更新依赖时，不要忘了版本号和版本控制符，否则
package.json 可能不会更新。

```
// 好的例子
pnpm update antd@5.8.5

// 反例一：一次性更新全部依赖
pnpm update

// 反例二：没指明版本
pnpm update antd
```

## Dependencies

项目运行所需的依赖项：

| 名称                                | 版本      | 说明                              |
|-----------------------------------|---------|---------------------------------|
| @amap/amap-jsapi-loader           | 1.0.1   | 高德地图 jsapi 启动器                  |
| @ant-design/icons                 | 6.1.1   | Antd 的图标库                       |
| @ant-design/v5-patch-for-react-19 | 1.0.3   | Antd 的 React 19 兼容包             |
| @antv/g2                          | 5.4.8   | AntV G2 图表组件库                   |
| @dagrejs/dagre                    | 1.1.5   | 流程图布局（与 xyflow 配合使用）            |
| @dnd-kit/core                     | 6.3.1   | dnk-kit 拖放                      |
| @dnd-kit/sortable                 | 10.0.0  | dnk-kit 拖放排序                    |
| @dnd-kit/utilities                | 3.2.2   | dnk-kit 拖放工具包                   |
| @tanstack/react-query             | 5.96.2  | 异步状态管理（获取、缓存、同步和更新服务器状态）        |
| @tiptap/extension-highlight       | 3.22.2  |                                 |
| @tiptap/extension-text-align      | 3.22.2  |                                 |
| @tiptap/pm                        | 3.22.2  | Tiptap 富文本编辑器                   |
| @tiptap/react                     | 3.22.2  | Tiptap 富文本编辑器                   |
| @tiptap/starter-kit               | 3.22.2  | Tiptap 富文本编辑器                   |
| @xyflow/react                     | 12.10.2 | flow 流程图库                       |
| antd                              | 6.3.5   | Antd UI 组件库                     |
| axios                             | 1.14.0  | http 网络请求库                      |
| dayjs                             | 1.11.20 | 日期格式化等处理（⚠️注意版本要和Antd依赖的版本保持一致） |
| i18next                           | 26.0.3  | 国际化                             |
| js-cookie                         | 3.0.5   | 管理 Cookie                       |
| qs                                | 6.15.0  | http 查询参数序列化和解析                 |
| react                             | 19.2.4  | 前端框架 React 的核心库                 |
| react-dom                         | 19.2.4  | React Web 库，与 React 包配合使用       |
| react-i18next                     | 17.0.2  | 国际化 React 适配包，与 i18next 包搭配使用   |
| react-router                      | 7.14.0  | react 路由                        |
| throttle-debounce                 | 5.0.2   | 防抖与节流                           |
| zustand                           | 5.0.12  | 状态管理                            |

## DevDependencies

仅用于开发的依赖项：

| 名称                          | 版本      | 说明                      |
|-----------------------------|---------|-------------------------|
| @eslint/js                  | 10.0.1  | 与 eslint 包搭配使用          |
| @types/js-cookie            | 3.0.6   | js-cookie 的类型定义         |
| @types/node                 | 25.5.2  | node 的类型定义              |
| @types/qs                   | 6.15.0  | qs 的类型定义                |
| @types/react                | 19.2.14 | react 的类型定义             |
| @types/react-dom            | 19.2.3  | react-dom 的类型定义         |
| @types/throttle-debounce    | 5.0.2   | throttle-debounce 的类型定义 |
| @vitejs/plugin-react        | 6.0.1   | vite：react 插件           |
| eslint                      | 10.2.0  | js 规则校验                 |
| eslint-plugin-react-hooks   | 7.0.1   | eslint：react-hooks 插件   |
| eslint-plugin-react-refresh | 0.5.2   | eslint：react-refresh 插件 |
| globals                     | 17.4.0  | 与 eslint 搭配使用           |
| prettier                    | 3.8.1   | 代码格式化                   |
| typescript                  | 6.0.2   |                         |
| typescript-eslint           | 8.58.0  |                         |
| vite                        | 8.0.5   | 前端构建工具                  |
