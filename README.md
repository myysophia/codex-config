# Codex CLI 配置仓库

## 项目概述

本仓库收集了 Codex CLI 的个性化配置与提示词模板，目标是将资深全栈技术专家的工作方式固化为可复用的自动化流程。通过统一的模型参数、受信任项目列表以及丰富的工具集，本仓库帮助团队快速搭建一致、可审计、可扩展的 AI 协作环境。

## 核心特性

- **模型与推理策略统一**：`config.toml` 固定使用 `gpt-5-codex`，并启用高推理强度，保证复杂任务的可靠性。
- **项目信任分级**：针对常用项目配置 `trust_level`，在敏感仓库与普通仓库之间实现差异化的安全策略。
- **多工具协同**：预置 Context7、Sequential Thinking、时间查询、Playwright 等 MCP 服务，覆盖检索、规划、自动化操作等多种场景。
- **粒度化提示词库**：`prompts/` 目录按照任务阶段提供规划、实现、质检、文档等多种模版，支持组合调用。
- **中文工作流**：所有系统提示与输出约束均为中文，保证团队内部交流一致性。

## 目录结构

```
├── AGENTS.md                  # 顶层系统提示词与交互准则
├── config.toml                # Codex CLI 主配置，包含模型、项目、MCP 服务声明
├── internal_storage.json      # Codex CLI 内部持久化状态
├── prompts/                   # 任务提示词模板集合（Markdown + YAML Front Matter）
└── skills/                    # 可复用 Codex 技能集合
    └── codex-team/SKILL.md    # 按需协作、需求澄清与委派流程
```

> 建议在提交新文件前运行 `tree` 或 `ls` 确认目录层级保持清晰；若新增子目录，请同步更新本节说明。

## 快速上手

1. **准备环境**
   - 安装最新的 Codex CLI（参考官方文档）。
   - 确保本地具备 Node.js ≥ 18 与 Python ≥ 3.11，以便运行 `npx` 与 `uvx` 命令。
2. **应用配置**
   - 将本仓库克隆到本地，例如 `~/projects/codex-config`。
   - 在 Codex CLI 的配置目录（默认 `~/.config/codex`）创建或更新符号链接，使其指向本仓库的 `config.toml` 与 `prompts/`。
   - 如需要自定义 `projects` 节点，请复制 `config.toml` 并在本地分支修改，避免直接覆盖主干。
3. **验证安装**
   - 运行 `codex doctor` 或执行一次简单对话，确认模型、项目信任与 MCP 服务均已正确加载。
   - 在 Codex CLI 中执行 `list prompts`（若 CLI 支持）确保自定义提示词可被发现。

## 配置详解

### 模型与推理

- `model = "gpt-5-codex"`：指定 Codex CLI 的默认模型。
- `model_reasoning_effort = "high"`：要求更高的思考深度，适合复杂架构或评审任务。

### 项目信任级

- `[projects."…"]`：为常用工程设定 `trust_level = "trusted"`，允许 CLI 在这些仓库执行写操作与高权限命令。
- 新增项目时建议：
  1. 先以 `read-only` 模式试运行，确认自动化流程安全；
  2. 再将 `trust_level` 调整为 `trusted`，并在 README 的“维护建议”中记录原因与风险缓解措施。

### MCP 服务

下表列出默认启用的 MCP 服务及其作用：

| 服务标识                          | 启动命令                                  | 主要用途                       |
| --------------------------------- | ----------------------------------------- | ------------------------------ |
| `context7`                        | `npx -y @upstash/context7-mcp@latest`     | 查询官方文档 / SDK 参数        |
| `sequential-thinking`             | `npx -y @modelcontextprotocol/server-sequential-thinking` | 复杂任务的分步规划            |
| `mcp-server-time`                 | `uvx mcp-server-time --local-timezone=Asia/Shanghai` | 时区转换与时间获取           |
| `mcp-shrimp-task-manager`         | `npx -y mcp-shrimp-task-manager`          | 任务拆解与可视化（中文模版）  |
| `playwright`                      | `npx @playwright/mcp@latest`              | 浏览器自动化与可视化验证       |

> **最佳实践**：所有 MCP 服务均以“最小必要”原则调用，使用前评估网络、隐私与合规风险；若调用失败，优先回退至离线操作，并在交互中说明降级措施。

### 内部存储

- `internal_storage.json` 记录 CLI 的轻量状态，例如模型提示词是否已展示。除非调试，通常无需修改；若需要重置，可在停止 CLI 后删除该文件。

## 提示词模板说明

### 模板结构

- 每个 Markdown 文件以 YAML Front Matter 开头（如 `description` 字段），用于在 CLI 中生成摘要信息。
- 正文包含执行步骤、输入要求与安全边界，所有文本采用中文约束，确保输出一致性。
- 模板遵循“只描述，不执行”的理念：实际的文件编辑或命令执行由 Codex CLI 调度完成。

### 分类总览

| 分类           | 文件示例                                 | 功能说明                                               |
| -------------- | ---------------------------------------- | ------------------------------------------------------ |
| 规划与分析     | `analyze.md`、`tasks.md`、`instruction-reflector.md` | 生成任务分解、校验规范、反思指令一致性                 |
| 设计与文档     | `specify.md`、`insight-documenter.md`、`api-docs.md` | 产出规格说明、知识沉淀、API 文档                       |
| 实施与测试     | `implement.md`、`gen-tests.md`、`optimize.md` | 指导编码实现、测试生成与性能优化                       |
| 质检与评审     | `review-code.md`、`github-pr-reviewer.md`、`commit-msg.md` | 代码评审、PR 审核、提交信息撰写                       |
| 调试与支持     | `debug.md`、`clarify.md`、`check-env.md` | 问题定位、需求澄清、环境检查                           |
| 专用场景       | `kiro-*.md` 系列                         | 针对 Kiro 产品线的特定流程（需求评审、特性设计等）     |
| UI/前端        | `ui-engineer.md`                         | 聚焦界面实现与交互策略                                 |

> **提示**：添加新模板时保持命名一致性（动词-名词），并在 Front Matter 的 `description` 里使用一句话中文概括，方便快速检索。

### 完整模板清单

| 模板文件 | 典型用途 |
| -------- | -------- |
| `analyze.md` | 交叉校验规格、计划与任务的一致性 |
| `api-docs.md` | 汇总或生成接口文档与参数说明 |
| `check-env.md` | 排查运行环境与依赖配置 |
| `clarify.md` | 引导澄清不完整或歧义的需求 |
| `commit-msg.md` | 帮助撰写符合规范的提交信息 |
| `constitution.md` | 定义或回顾项目的宪章级原则 |
| `db-schema.md` | 设计领域数据模型与数据库结构 |
| `debug.md` | 分析定位故障与异常原因 |
| `deep-reflector.md` | 深度反思任务执行策略与改进点 |
| `explain.md` | 解释代码、正则表达式等技术细节 |
| `gen-tests.md` | 生成或完善测试用例 |
| `github-issue-fixer.md` | 分析 GitHub Issue 并提供解决方案 |
| `github-pr-reviewer.md` | 审查 Pull Request 并输出评审意见 |
| `implement.md` | 按 tasks.md 执行实现计划 |
| `insight-documenter.md` | 记录关键洞察与技术总结 |
| `instruction-reflector.md` | 对既有指令进行复盘与优化 |
| `kiro-assistant.md` | 支持 Kiro 产品线的一般性工作流 |
| `kiro-feature-designer.md` | 规划 Kiro 新特性的需求与设计 |
| `kiro-spec-creator.md` | 为 Kiro 场景编写规格文档 |
| `kiro-task-executor.md` | 执行 Kiro 专属任务清单 |
| `kiro-task-planner.md` | 拆解 Kiro 需求并生成任务计划 |
| `optimize.md` | 优化查询、算法或系统性能 |
| `prompt-creator.md` | 辅助设计新的提示词模板 |
| `refactor.md` | 指导代码重构与结构调整 |
| `review-code.md` | 进行代码评审与缺陷识别 |
| `specify.md` | 编写或完善规格说明 |
| `tasks.md` | 生成分阶段、可执行的任务列表 |
| `ui-engineer.md` | 聚焦界面实现、交互与样式策略 |

### 使用建议

- 在 Codex CLI 中调用命令时，优先选择语义最贴近的模板；若存在流程依赖（如先 `/tasks` 后 `/implement`），请严格按模板所述顺序执行。
- 模板强调“安全边界”：若出现读写限制、需要人工审批等提醒，请在实际操作前与团队确认。
- 若某些模板需要特定项目结构（如 `.specify/` 目录），请在调用前确认目标仓库已满足前置条件。

### 命令调用示例

- `/explain "这个正则表达式：^(?=.*[A-Z])(?=.*[a-z])(?=.*\d).{8,}$"`
- `/explain "为什么这段 Go 代码在并发下产生数据竞态？"`
- `/debug "Docker 容器启动后立即退出，exit code 137"`
- `/debug "K8s Pod 健康检查失败，Readiness probe 超时"`
- `/optimize "SELECT * FROM orders WHERE user_id = ?"`
- `/optimize "GraphQL 列表查询延迟高，如何分页与缓存？"`
- `/db-schema "博客平台：文章、评论、标签、分类、用户关注"`
- `/db-schema "电商：用户、商品、库存、订单、支付、退款"`
- `/analyze "对 feature-X 的 spec.md/plan.md/tasks.md 做一致性分析"`
- `/api-docs "订单服务 v1：列出/创建/取消订单接口，返回 JSON Schema"`
- `/check-env "Node 18 + pnpm + Docker Desktop + Postgres 需要检查哪些项？"`
- `/clarify "报表导出的‘增量模式’应该如何定义时间窗口？"`
- `/commit-msg "feat: 支持多租户鉴权与组织切换"`
- `/constitution "新增隐私数据处理原则：最小化采集、可追溯、可删除"`
- `/deep-reflector "回顾上周生产事故的根因、缓解与长期改进项"`
- `/gen-tests "为用户注册接口补充单测：邮箱校验、重复注册、速率限制"`
- `/github-issue-fixer "#1234 CI 构建偶发失败：超时 60s"`
- `/github-pr-reviewer "#5678 增加缓存层与指标打点，请给出风险与建议"`
- `/implement "按 tasks.md 的 Core 阶段开始实现并输出变更清单"`
- `/insight-documenter "总结搜索性能优化洞察：索引、分页、缓存、N+1"`
- `/instruction-reflector "审查 /optimize 模板是否与 AGENTS.md 原则一致"`
- `/kiro-assistant "整理 Kiro 看板的优先级、依赖与风险"`
- `/kiro-feature-designer "为‘智能模板’设计 MVP 范围与验收标准"`
- `/kiro-spec-creator "为‘知识库同步’编写规格：触发、冲突合并、安全"`
- `/kiro-task-executor "执行 Sprint-12 的集成与验收任务"`
- `/kiro-task-planner "将‘评论系统’拆解为 5 个可并行任务"`
- `/prompt-creator "生成用于代码评审的团队风格化提示词"`
- `/refactor "重构 monolith：拆分 auth、billing、report 三个模块"`
- `/review-code "评审 PR #4321：关注并发安全、可测试性、日志"`
- `/specify "编写‘导出报表’功能的规格：功能、非功能、边界"`
- `/tasks "基于 spec.md 与 plan.md 生成任务清单"`
- `/ui-engineer "设计订单列表的空态、加载态与错误态"`

## 维护与扩展指南

- **版本管理**：新增或更新模板后，请在提交信息中说明变更动机、适用场景与风险缓解措施。
- **一致性检查**：对 `AGENTS.md` 的改动会影响所有任务，请保持审慎；建议在合并前进行团队评审。
- **测试流程**：在真实项目中试运行新模板，记录反馈并回写到模板末尾的注意事项或到本 README 的维护章节。
- **多语言需求**：若需支持英文等其他语言，建议建立独立分支或目录，避免混用导致输出风格混乱。

## 常见问题

- **无法启动 MCP 服务？** 检查本地是否安装 `npx`、`uvx` 等命令；若网络受限，请寻求离线替代方案，并在 CLI 中提示用户。
- **提示词输出为英文？** 确认调用时没有覆盖系统提示；必要时重启 CLI，使 `AGENTS.md` 中的中文约束重新生效。
- **项目未列在 `projects` 中？** CLI 将以默认信任级运行，可能限制写操作。根据需要手动将仓库添加到 `config.toml`。

## 后续规划

- 跟随 Codex CLI 版本迭代，持续验证指令兼容性。
- 根据团队反馈扩展 UI、数据工程、运维等领域的模板，并在本 README 更新分类表。
- 探索通过 MCP 管理更多基础设施（如日志查询、工单系统），并确保遵守安全合规原则。

---

欢迎在实际使用中记录经验与改进建议，并通过 Issue 或 Pull Request 反馈，共同完善这套中文友好的 Codex 工作流。
