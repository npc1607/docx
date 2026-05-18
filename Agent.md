# Project Agent Guide

本仓库是基于 Kratos v2 的 Go 1.24 项目，服务名为 `<service-name>`。默认复用现有分层、命名和注入方式，不要发明与仓库结构冲突的新目录或抽象。

## 项目结构

- `api/<module>/v1/*.proto`: API 协议定义的单一事实来源
- `cmd/<service-name>/main.go`: 程序入口
- `cmd/<service-name>/wire.go`: Wire 注入声明
- `cmd/<service-name>/wire_gen.go`: Wire 生成文件
- `internal/service`: 协议适配、参数校验、错误转换
- `internal/biz`: 业务编排、usecase、repo interface
- `internal/data`: DB / Redis / MQ / 外部依赖实现
- `internal/server`: HTTP / gRPC / MQ / Scheduler 组装
- `internal/conf/*.proto`: 配置结构定义

## 硬约束

- 接口变更先改 `api/<module>/v1/*.proto`，不要直接改生成产物。
- 新模块默认按 `api`、`internal/service`、`internal/biz`、`internal/data` 分层落位。
- 不要跨层写业务逻辑；`service` 只做适配，`biz` 负责业务，`data` 负责外部依赖。
- 新增依赖保持显式注入，不要引入全局状态。
- 处理 git 冲突时，不要手工解决生成文件冲突；生成文件出现冲突时，执行对应命令重新生成后再提交。
- 跨 RPC、DB、Redis、MQ、HTTP 等边界的方法优先显式传递 `context.Context`，并保持 `ctx` 为第一个参数。
- 错误包装使用 `%w`；service 层按现有模块风格转换为 Kratos 错误。
- 函数参数或返回值超过 3 个时，优先改为结构体。
- 初始化阶段的确定性错误可使用 Must 风格；外部依赖导致的失败必须显式处理。

## 禁止手改的生成文件

- `api/**/v1/*.pb.go`
- `api/**/v1/*_grpc.pb.go`
- `api/**/v1/*_http.pb.go`
- `api/**/v1/*_errors.pb.go`
- `openapi.yaml`
- `cmd/<service-name>/wire_gen.go`

## 修改后必须执行

- 修改 API proto 后执行项目约定的 API 生成命令，例如 `make api`
- 修改配置 proto 后执行项目约定的配置生成命令，例如 `make config`
- 新增或调整 Provider、构造函数、ProviderSet、依赖绑定后执行 `cd cmd/<service-name> && wire`
- 修改 Go 文件后执行 `gofmt`，如可用再执行 `goimports`

## 工具与技能约束

- 阅读和分析 Go 代码优先使用 `gopls`，不要只依赖普通文本搜索。
- 涉及第三方库、框架、SDK、API、CLI 或云服务文档时，优先使用 `context7` 获取最新文档。
- 新增、修改或评审 `.go` 文件前使用 `golang-style` 和 `Effective Go` skill。
- 新增、修改或评审代码方案前使用 `implementation-guardrails` skill。
- 涉及现有模块、函数、服务、handler、repository 或集成的替换、重写、迁移、现代化改造或高风险重构时，使用 `module-migration-replacement-strategy` skill；先确定最小可替换边界，保留新旧实现并行，通过比对验证后再逐步切换调用方，禁止直接一次性破坏性替换。
- 涉及 GORM、sqlx、PostgreSQL、repository pattern、事务、分页或 migrations 时使用 `golang-gin-database` skill。
- 进行代码审核或 PR review 时使用 `code-review-skill`。
- 任务涉及建分支或提交时，先应用 `git-commit` skill；`git commit` 信息必须符合 Conventional Commits。

## 参考文档

- `README.md`
- `Makefile`
- 目标模块现有的 `api/<module>/v1/*.proto` 与对应 `internal/service`、`internal/biz`、`internal/data` 实现
