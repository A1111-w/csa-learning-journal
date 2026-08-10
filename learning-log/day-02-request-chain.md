# Day 2：资源列表请求链路

## 学习目标

以资源列表接口为样本，从 Next.js 页面开始，追到 Spring Boot、MyBatis-Plus 和 MySQL，再沿响应方向追到 Axios 拆包和 React 状态更新。

## 当前源码基线

本次追踪必须按当前重构后的源码进行。资源模块已经从 Controller 直接持有 Mapper，改成了 Controller → Service → Mapper；响应也从 `Resource` Entity 改成 `ResourceVO`。

## 完整链路

```text
/dashboard/resources
-> csa-official-frontend/src/app/dashboard/resources/page.tsx
-> DashboardResourcesPage
-> ResourceLibrary
-> useEffect 触发 loadResources(page, activeCategory)
-> resourceService.list({ page, size, category })
-> Axios request interceptor
-> GET /api/sys/resource/list?page=x&size=x&category=...
-> SecurityConfig 过滤器链
-> JwtAuthenticationFilter.doFilterInternal
-> Cookie/Bearer 取 Token
-> JwtUtils 校验 Token
-> UserDetailsServiceImpl 加载当前用户
-> UsernamePasswordAuthenticationToken
-> SecurityContextHolder.setAuthentication
-> @PreAuthorize("hasRole('LEVEL_1')")
-> ResourceController.list
-> resourceService.listResources(page, size, category)
-> PageUtils.of(page, size)
-> LambdaQueryWrapper<Resource>
-> category 等值筛选（如果有分类）
-> createTime 降序
-> resourceMapper.selectPage(pageParam, query)
-> MySQL
-> Page<Resource>
-> ResourceVO.from
-> Page<ResourceVO>
-> R.ok
-> Axios response interceptor
-> response.data：取出后端 R<T>
-> payload.data：取出 PageResult<ResourceItem>
-> setItems / setPages / setTotal
-> ResourceLibrary 重新渲染
```

## 前端 Hook 的作用

`ResourceLibrary` 中的第一个 `useEffect` 在组件提交到页面后执行资源加载；它依赖 `page`、`activeCategory`、`canViewResources` 等状态，所以翻页或切换分类时会再次请求。分类请求由另一个 `useEffect` 调用 `loadCategories` 完成。`useMemo` 只负责根据已有状态计算展示用的分类数组，不负责发 HTTP 请求。

## 后端认证与授权

`JwtAuthenticationFilter` 不是把用户永久保存到数据库，而是在当前请求线程的 `SecurityContext` 中写入 `Authentication`。后续 `@PreAuthorize` 从这个上下文读取当前用户的 authorities，判断是否拥有 `LEVEL_1`。

`JwtAccessDeniedHandler` 只负责权限不足后的 403 响应，不负责解析 JWT 或主动检查权限。

## 分页与查询

```text
PageUtils.of
- page 为 null 或小于 1：归一为 1
- size 为 null：默认 10
- size 范围收敛到 1..100
```

如果 `category` 有文字，Service 使用等值条件筛选，并始终按 `createTime DESC` 排序。`PageUtils.of` 不知道真实总页数；如果前端请求页码超过实际页数，`ResourceLibrary` 收到响应后会重新请求最后一页。

## Entity、VO 和统一返回

`ResourceMapper.selectPage` 返回 `Page<Resource>`。Service 用 `ResourceVO.from` 只复制允许对外暴露的字段，得到 `Page<ResourceVO>`，Controller 再用 `R.ok` 包装。

Axios 的响应拦截器运行时完成两次取值：

```text
AxiosResponse.data -> 后端 R<T>
R<T>.data          -> PageResult<ResourceItem>
```

TypeScript 泛型只描述类型，不执行拆包。

## React 状态更新

响应拦截器返回 `PageResult<ResourceItem>` 后，`loadResources` 执行：

```ts
setItems(response.records)
setPages(Math.max(response.pages || 1, 1))
setTotal(response.total || 0)
```

这些调用会更新组件 State 并安排 React 重新渲染。下一次渲染时，`items` 用于生成资源卡片，`pages` 用于分页控件，`total` 保存结果总数；`loading` 则控制加载状态。

## Day 2 验收结论

已能从前端页面追到后端数据库，再追到 VO、统一返回和前端状态更新。需要继续巩固的点是 React Hook 的触发时机、SecurityContext 的请求级生命周期，以及 Mapper 继承 `BaseMapper` 后的实际调用位置。
