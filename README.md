# CSA Learning Journal

这个仓库记录我对 CSA Official 项目的学习过程。目标不是抄写现成文档，而是逐步做到：

- 能从源码定位技术栈、配置和模块；
- 能追踪一次请求从前端到数据库的完整链路；
- 能解释认证、授权、统一返回、异常和缓存；
- 能独立排错、修改代码并验证结果；
- 最终能完整讲解项目，并继续开发 RAG / Agent 能力。

## 学习对象

CSA Official 是一个计算机协会官网和内部管理平台：

- 前端：Next.js、React、TypeScript、Axios；
- 后端：Spring Boot、Spring Security、MyBatis-Plus；
- 数据：MySQL，缓存支持 Redis 和本地内存实现；
- 业务：用户与部门、资源、竞赛、简历、投票、贡献和公开内容；
- 安全：JWT、HttpOnly Cookie、CSRF 和接口权限控制。

## 学习进度

| 阶段 | 主题 | 状态 |
| --- | --- | --- |
| Day 1 | 项目全景 | 已完成 |
| Day 2 | 资源列表请求链路 | 已完成 |
| Day 3 | 统一返回和异常 | 待开始 |
| Day 4-6 | 登录、安全和权限 | 待开始 |
| Day 7-11 | 业务模块 | 待开始 |
| Day 12 | 测试 | 待开始 |

## 当前记录

- [Day 1：项目全景](learning-log/day-01-project-overview.md)
- [Day 2：资源列表请求链路](learning-log/day-02-request-chain.md)
- [CSA 项目全景图](notes/csa-project-map.md)

## 记录原则

1. 结论尽量写明源码位置。
2. 不会的内容直接记录，不用 AI 答案冒充自己的理解。
3. 区分“独立掌握”“经提示掌握”和“尚未学习”。
4. 每天至少留下学习目标、源码证据、理解偏差和下一步动作。
