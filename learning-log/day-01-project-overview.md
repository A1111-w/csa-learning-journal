# Day 1：项目全景

## 学习目标

建立 CSA Official 的项目地图，知道前端、后端、主要模块、依赖和外部设施分别在哪里。Day 1 不要求提前吃透 JWT、CSRF、Mapper 或业务状态流转，这些内容会在后续请求链路和安全学习中深入。

## 阅读过的源码

```text
README.md
csa-official-backend/pom.xml
csa-official-backend/src/main/resources/application.yml
csa-official-backend/src/main/java/com/csa/official/CsaOfficialApplication.java
csa-official-frontend/package.json
csa-official-frontend/src/app
```

为了确认模块职责，我还阅读了：

```text
modules/sys/controller/AuthController.java
modules/sys/controller/CarouselController.java
modules/sys/controller/PublicController.java
modules/sys/entity/User.java
modules/sys/service/DeptService.java
modules/biz/controller/CompetitionController.java
modules/resume/controller/ResumeController.java
```

## 源码中确认的版本

| 技术 | 版本 | 查找位置 |
| --- | --- | --- |
| Spring Boot | 3.5.8 | `pom.xml` 的 `parent.version` |
| Java | 17 | `pom.xml` 的 `properties.java.version` |
| Next.js | 16.2.10 | `package.json` 的 `dependencies.next` |
| React | 19.2.4 | `package.json` 的 `dependencies.react` |
| React DOM | 19.2.4 | `package.json` 的 `dependencies.react-dom` |
| TypeScript | ^5 | `package.json` 的 `devDependencies.typescript` |

## 模块理解

### common、config、modules

- `common`：跨业务复用的返回、异常、安全、缓存、工具和注解。
- `config`：Spring Security、缓存、跨域、MyBatis、Swagger 等全局配置。
- `modules`：具体业务代码。

### sys、biz、resume

- `sys`：目前是较大的综合模块，不只包含用户，还包含部门、资源、投票、贡献、轮播图、公开配置和导出等功能。
- `biz`：竞赛、竞赛编辑授权以及 Git 相关能力。
- `resume`：简历保存、提交、审核和状态流转。

竞赛和简历被单独拆成业务域，但 `sys` 的范围仍然很宽。Day 1 先描述当前源码结构，不假设它是唯一正确的模块划分。

## 前端页面

公开页面包括：

```text
about
competitions
contributors
resources
```

工作台页面包括：

```text
dashboard/competitions
dashboard/departments
dashboard/profile
dashboard/resources
dashboard/resume
dashboard/settings
dashboard/vote
```

当前没有单独的 `dashboard/files` 页面，文件能力由资源等业务使用。

## 依赖与实际用途

| 依赖 | 当前理解 | 源码证据 |
| --- | --- | --- |
| EasyExcel | 导出 Excel | `UserExportService.java` |
| JJWT | 生成、签名、解析和校验 JWT | `JwtUtils.java` |
| MySQL Connector | Java 连接 MySQL 的 JDBC 驱动 | `pom.xml` 中为 runtime 依赖 |
| Validation | 对 DTO 字段执行声明式参数校验 | `RegisterDto.java`、`AuthController.java` |

需要牢记：JJWT 不负责加密用户密码。密码由 `PasswordEncoder` 做单向编码；JWT 用签名保护 Token 不被合法篡改。

Validation 的关系是：

```text
DTO 上的 @NotBlank / @Size / @Email / @Pattern 定义规则
-> Controller 参数上的 @Valid 启动校验
-> 校验失败时方法正文不继续执行
-> GlobalExceptionHandler 返回统一的 400 响应
```

## 后端启动入口

`CsaOfficialApplication.java` 中：

- `@SpringBootApplication`：标记启动配置类，开启自动配置，并扫描当前包下的 Spring Bean。
- `@MapperScan("com.csa.official.modules.*.mapper")`：扫描 `sys`、`biz`、`resume` 下的 Mapper 接口。
- `@EnableScheduling`：开启 `@Scheduled` 定时任务。

当前定时任务包括 `ContributionTask.settleIntroContribution()`：每天凌晨 4 点检查协会介绍是否稳定超过 7 天，并避免重复发放贡献分。

## 阅读源码时纠正的偏差

1. `sys` 不只是用户信息模块，它还包含多个系统和组织治理功能。
2. `SecurityUtils.getUserId()` 是获取当前认证用户 ID，不是读取 DTO 字段。
3. 竞赛 `/grant` 是授予编辑权限，不是“同意竞赛”。
4. `SaveCompetitionDto` 与 `GrantDto` 是两个不同的请求对象。
5. Java `stream()` 是集合处理，不是流式上传。
6. `ContributorVo` 是返回给前端的视图对象。
7. `User` 是映射 `sys_user` 表的实体，不只是注册时创建用户的临时对象。
8. `DeptService.batchPromoteToMember` 使用一条批量更新 SQL，不是逐条更新。

## 当前掌握状态

### 独立掌握

- 能从 `pom.xml` 和 `package.json` 查版本。
- 能找到公开页面和工作台页面。
- 能进入 `sys`、`biz`、`resume` 并找到真实 Controller、Entity 和 Service。
- 能顺着 Controller 的注解和方法大致判断接口用途。

### 经提示后掌握

- `@Valid` 与字段约束注解的配合。
- Controller 与 Service 的职责边界。
- `@MapperScan` 和 `@EnableScheduling` 的基本作用。
- JWT 签名与密码编码的区别。

### 后续继续学习

- Controller、Service、Mapper 的完整请求链路。
- Axios 为什么能自动拆出后端返回的数据。
- Spring Security、JWT、Cookie 和 CSRF 的完整认证链路。
- Service 中的事务、锁、缓存和状态流转。

## Day 1 结论

Day 1 目标已完成：已经能够从源码建立项目全景，并能区分哪些内容是自己找到的、哪些内容仍需后续学习。当前还不能独立解释整个系统，但已经具备进入 Day 2 追踪请求链路的基础。

## Day 2 第一项动作

从资源列表页面开始追踪：

```text
/dashboard/resources
-> ResourceLibrary
-> resourceService.list
-> Axios
-> GET /api/sys/resource/list
-> ResourceController
-> Mapper
-> MySQL
-> R<Page<Resource>>
-> 前端渲染
```
