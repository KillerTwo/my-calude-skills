# 后端实现阶段

## 实现依据

你只能依据以下内容编码：

- `docs/<module>/后端实施计划.md`
- `docs/<module>/后端开发任务清单.md`
- `docs/<module>/前端Mock接口Curl示例.md`
- `docs/<module>/前端mock对齐表.md`
- `docs/<module>/数据库表与状态流转设计.md`
- 前端页面代码
- mock 数据
- 现有后端代码

## 固定流程

1. 读取当前模块规格与任务清单
2. 只选择当前未完成的第一个任务
3. 先写失败测试
4. 运行测试并确认失败，且失败原因正确
5. 写最小实现
6. 运行测试并确认通过
7. 站在实现者视角做自审
8. 用独立 reviewer 视角做第二轮审查
9. 按 curl 文档执行真实接口 smoke test
10. 确认没有阻塞后再提交

## 编码规则

- 优先使用 `BaseCrudService` 和 `Dao`
- 复杂查询再使用 xml mapper
- 所有 controller、service 接口、service impl、重要类都写标准 javadoc
- service 接口方法必须写 javadoc
- 保持国际化风格，参考 material 模块
- 只使用 DTO，禁止 `xxxVO`
- 包结构、分层、校验、返回包装参考 material 模块
- 数据库更新脚本生成到 `ht-starter.js`，版本号必须递增
- controller 的接口方法上禁止使用 `@OpenApi`
- 分页接口返回以后端标准 `PageResult` 为准，不照抄错误 mock
- 后端接口路径禁止默认添加 `/api`

## 自审检查清单

- 是否严格只实现了当前任务
- 是否先写失败测试
- 是否验证过失败
- 是否验证过通过
- 是否存在猜测性实现
- 是否与前端 mock 完全对齐
- 是否遗漏国际化消息
- 是否遗漏 javadoc
- 是否新增前端未定义的必填字段
- 是否遗漏前端已定义字段
- 是否符合 material 模块风格

## 独立审查重点

按严重级别给出问题：

- Critical
- Important
- Minor

重点关注：

- 行为回归
- 接口路径或 method 偏差
- 请求参数结构偏差
- 返回字段缺失或多余
- 字段类型不一致
- 分页结构不一致
- 状态值不一致
- 国际化遗漏
- 测试覆盖缺口
- 猜测性实现

只要存在 `Critical` 或 `Important`，就不能进入提交阶段。

## smoke test 要求

- 服务地址固定为 `http://localhost:18005`
- 不允许使用 `127.0.0.1`
- 启动服务时使用 `${JAVA_HOME_21}` 和 `${MAVEN_HOME}`
- 启动 Spring Boot 时添加：
  `-Dspring-boot.run.jvmArguments='--add-opens java.base/java.lang=ALL-UNNAMED --add-opens java.base/java.lang.reflect=ALL-UNNAMED'`
- 先调用 `POST /authorize/login` 获取测试 token
- 测试登录账号使用 `superadmin/admin`
- 测试验证码使用 `1234`
- 调用其他受保护接口时加 `Authorization: $TOKEN`
- 请求参数、header、body 必须与文档一致
- 不只看 HTTP 状态码，还要看返回结构和字段

## 后端任务完成定义

每个开发任务只有同时满足以下条件才算完成：

- 已定位为当前唯一任务
- 已写失败测试
- 已运行测试并确认失败原因正确
- 已完成最小实现
- 已运行测试并确认通过
- 已完成自审
- 已完成独立审查
- 已执行 curl smoke test
- 实现内不存在猜测性代码
- 已记录测试与审查结果

