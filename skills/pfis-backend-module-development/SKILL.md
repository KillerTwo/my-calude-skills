---
name: pfis-backend-module-development
description: 用于 pfis 项目按前端事实源驱动后端模块开发，先生成并审查模块规格，再按任务清单逐个做 TDD 实现、接口对齐、双重审查与 smoke test；适用于把前端 mock 切换为真实后端接口且不允许猜测推进的场景。
---

# PFIS Backend Module Development

## 适用场景

在 `pfis-*` 多模块项目中，用户要求你为某个明确指定的模块做后端业务开发时使用本 skill。

典型输入：

- `给 src/modules/material/device 生成后端规格并开始开发`
- `继续实现 docs/material/后端开发任务清单.md 里的下一个任务`
- `把某个前端页面从 mock 切到真实后端接口`

本 skill 只覆盖业务开发规则、规格流程、实现流程、测试与审查要求。

## 先决条件

开始前必须确认：

- 用户明确指定了模块；不能由你自己扫描并决定做哪个模块。
- 一次只处理一个模块。
- 关键事实源可读取：前端页面代码、前端交互效果、api 定义、mock 数据或接口、接口数据类型和格式。

如果模块未明确指定，或关键事实源缺失，立即停止并向用户确认。

## 事实源边界

你只能依据以下内容得出关键结论：

- 前端页面代码
- 前端交互效果
- api 定义
- mock 数据或接口
- 接口数据类型和格式
- 已提交的模块规格文档
- 现有后端代码中可直接复用且可证明相关的模式
- 用户明确确认的信息

不能仅凭经验补齐字段、状态、校验、数据库关系或流程规则。

详细边界与暂停条件见 [references/fact-sources-and-stop-rules.md](./references/fact-sources-and-stop-rules.md)。

## Worktree 规则

开始规格阶段或后端实现阶段前，必须先处理工作目录隔离：

- 默认使用 git worktree 进行开发；如果当前已经处于 linked worktree 中，直接继续，不要再套一层 worktree。
- 如果当前不在 worktree 中，则在当前仓库根目录下创建项目内 worktree，固定放在 `.worktrees/` 目录下；不要使用全局目录或仓库外目录。
- `.worktrees/` 必须被 git 忽略；如果 `.gitignore` 尚未忽略 `.worktrees/`，先补上忽略规则，再创建 worktree。
- 如果环境限制导致无法创建 worktree，必须明确向用户说明原因，并在得到用户确认前不要静默改为原地开发。

## 总流程

### 1. 判断当前阶段

先判断当前模块是否已经具备已确认的规格文档：

- 如果还没有完整规格，先做规格阶段。
- 如果规格已存在且用户要求继续开发，再进入后端实现阶段。

### 2. 规格阶段必须先于开发阶段

规格阶段必须先完成并通过审查，之后才能开始后端实现。

规格阶段必须生成 `docs/<module>/` 下这 5 份文档：

- `后端实施计划.md`
- `后端开发任务清单.md`
- `前端Mock接口Curl示例.md`
- `前端mock对齐表.md`
- `数据库表与状态流转设计.md`

规格阶段要求见 [references/spec-phase.md](./references/spec-phase.md)。

### 3. 后端实现阶段按任务逐个推进

只允许处理当前一个最小任务，不允许连做多个任务，也不允许顺手补下一个任务。

实现流程固定为：

1. 先写失败测试
2. 运行测试并确认失败，且失败原因正确
3. 写最小实现
4. 运行测试并确认通过
5. 自审
6. 独立审查
7. 执行 curl smoke test
8. 确认完成后再提交

后端实现规则见 [references/backend-phase.md](./references/backend-phase.md)。

## 接口与实现硬规则

- 严格按当前模块已确认的规格文档开发。
- 接口定义必须完全对齐前端页面、api 定义与 mock。
- 不允许新增前端未定义的必填字段。
- 不允许省略前端已定义字段。
- 不允许写占位实现、TODO、猜测性代码。
- controller 接口方法禁止使用 `@OpenApi`。
- 后端接口真实路径禁止默认添加 `/api` 前缀，必须以前端代码中的真实调用路径为准。
- 分页接口返回结构以服务端标准 `PageResult` 为准；如果 mock 分页结构不同，要标记为 mock 偏差，不能直接照抄。
- 优先使用 `BaseCrudService` 和 `Dao`；复杂查询再使用 xml mapper。
- 只使用 DTO，禁止新增 `xxxVO`。
- 保持国际化风格，参考已有 material 模块。
- `controller`、`service` 接口、`service impl`、重要类都写标准 javadoc；`service` 接口方法必须写 javadoc。
- 数据库变更写入 `pfis-station/src/main/resources/ht-starter.js`，版本号必须在当前基础上递增。

接口对齐检查项与完成清单见 [references/checklists.md](./references/checklists.md)。

## 测试与验证

- 必须走 TDD，不能跳过“先失败再实现”。
- 测试不能只看 HTTP 状态码，还要校验返回结构和字段。
- 如需启动本地服务，使用 `${JAVA_HOME_21}` 与 `${MAVEN_HOME}`。
- 启动 Spring Boot 时附加：
  `-Dspring-boot.run.jvmArguments='--add-opens java.base/java.lang=ALL-UNNAMED --add-opens java.base/java.lang.reflect=ALL-UNNAMED'`
- smoke test 基础地址固定为 `http://localhost:18005`，不要用 `127.0.0.1`。
- 登录取 token 使用 `POST /authorize/login`，测试账号 `superadmin/admin`，验证码 `1234`。
- 其余受保护接口请求头加 `Authorization: $TOKEN`。

## 何时必须停止并问用户

遇到以下任一情况必须停止，不得自行猜测推进：

- 无法证明某项结论的事实来源
- 前端页面、交互效果、api 定义、mock 之间存在冲突
- 无法判断字段含义、是否必填、状态值语义、数据库关系、状态流转、编号规则
- 生物识别、上传结构、打印模板、文件模板、外部依赖或设备交互规则不明确
- 测试失败但无法确认是代码问题还是环境问题
- 服务无法启动且无法自动恢复
- 本地端口不可访问且无法自动恢复

提问时直接说明：

- 当前模块
- 当前阶段
- 当前任务
- 已确认事实
- 缺失或冲突的关键点
- 需要用户明确回答的问题

## 推荐输出节奏

在一次完整开发中，优先使用下面的输出结构向用户汇报：

1. 当前模块与当前阶段
2. 正在处理的唯一任务
3. 已读取的事实源
4. 当前完成的测试、实现、审查、smoke test
5. 是否阻塞，以及下一步

## 使用建议

- 如果任务是“先做规格”，先读 [references/spec-phase.md](./references/spec-phase.md)。
- 如果任务是“继续实现某个后端功能”，先读 [references/backend-phase.md](./references/backend-phase.md)。
- 如果要快速复核规则，读 [references/checklists.md](./references/checklists.md)。

## 推荐调用话术

把这个 skill 拷到 Claude 可发现的位置后，可以直接这样下达任务：

- `使用 pfis-backend-module-development skill，为 <模块目录> 先生成后端规格，再开始后端开发。`
- `使用 pfis-backend-module-development skill，继续实现 <module> 的下一个后端任务，只做一个任务并严格走 TDD。`
- `使用 pfis-backend-module-development skill，审查 <module> 当前任务是否满足接口对齐、双重审查和 smoke test 要求。`
