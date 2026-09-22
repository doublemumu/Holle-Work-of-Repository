# Multica 开源方案调研补充：同类开源项目横向对比与选型建议

> 归属专项：DFX-工程效能设计-MultiAgent 协作专项
> 补充对象：《Multica 开源方案调研文档》
> 资料基准时间：2026-09（所有结论基于该时点的公开资料与官方仓库/文档）

---

## 0. 本篇定位与阅读说明

### 0.1 这份文档补充了什么

原《Multica 开源方案调研》回答的是"Multica 是什么、能不能用"。本篇补充三件事：

1. **同类开源项目是谁**——把 Multica 放回它所在的赛道，明确哪些项目是竞品、哪些是互补件、哪些只是名字相近但根本不在一层。
2. **同一需求的不同实现路径**——针对 DFX 工程效能场景下的典型需求，逐个描述"如果用 X 方案来做，具体怎么落、要写多少东西、会卡在哪"。
3. **优劣势明细与决策依据**——把"看起来都能做"拆成可判定的维度，给出推荐主线、备选路线和必须自建的能力缺口。

### 0.2 一个必须说明的前提

本文档撰写环境**无法访问两个飞书链接**（`wiki/WvEOwyS7XiHvark3KRac6vUznKg` 与 `wiki/ZRzjwHLVNipl5VkJAELckDjZn8e` 均需企业登录态，匿名访问被重定向到登录页）。因此：

- 第 2 章的评估维度是**基于 DFX 工程效能 MultiAgent 协作的通用需求推导**得出的，而非直接读取需求表 `tbldJkKtocOnuMZF` 的字段。
- 落库到飞书多维表格时，请按该表实际的字段名做一次映射；如果表中有本文未覆盖的维度（例如特定的内部系统对接、特定合规条款），需要按同样的方法补测。
- 原调研文档中如已存在对 Multica 的结论，本文第 3.1 节可能与之重复或存在出入，以原文档为准，本文仅作事实校核与补充（尤其是 **License 条款**与**沙箱边界**两处，这两点在多数二手资料中是错的）。

### 0.3 事实可靠性分级

| 标记 | 含义 |
| --- | --- |
| 【一手】 | 来自项目官方仓库 README / LICENSE / 官方文档，已逐字核对 |
| 【二手】 | 来自第三方评测、技术博客、社区对比文章，方向可信但数字可能过时 |
| 【推断】 | 基于架构事实做的工程判断，需要 POC 验证 |

---

## 1. 结论速览

### 1.1 一句话结论

**Multica 是目前"人机混编任务协同层"里完成度最高、也最贴合国内工程团队习惯的开源方案，建议作为主线；但它不是一个多智能体编排引擎，也不提供执行沙箱，这两块能力必须从别处补。**

### 1.2 分层认知：这些项目根本不在同一层

调研中最容易出错的地方，是把"都叫多智能体"的项目摆在一张表里直接比分。它们解决的是完全不同的问题：

| 层 | 层的职责 | 代表项目 | 与 Multica 的关系 |
| --- | --- | --- | --- |
| **L4 人机协同工作管理层** | 谁来做、做到哪一步、谁验收、花了多少钱 | **Multica**、Vibe Kanban、OpenHands Agent Canvas、GitHub Agent HQ（闭源） | **同层直接竞品** |
| **L3 企业智能体应用平台** | 面向业务人员的智能体搭建与多渠道发布 | 飞书 aily（智能伙伴创建平台）、Coze Studio、Dify | 场景错位，非竞品；可作为非研发侧智能体的承载 |
| **L2 Agent 运行时 / 个人智能体框架** | 一个 Agent 自己的循环：工具调用、记忆、技能、子代理 | **Hermes Agent**、OpenClaw、Claude Code、Codex、Cursor Agent | **互补件**——Multica 调度它们，Hermes 是 Multica 已支持的 26 个 CLI 之一 |
| **L1 多智能体编排框架** | 用代码把多个 Agent 编成图/流水线，管状态与恢复 | LangGraph、CrewAI、Microsoft Agent Framework、AG2 | 互补件；Multica 的 Squad 只做"路由"不做"编排"，复杂流程要在这层补 |

> **关键结论**：用户提到的"飞书智能体"和"Hermes Agent 框架"**都不是 Multica 的替代方案**。飞书 aily 在 L3（业务侧智能体，不碰代码仓库），Hermes 在 L2（是被 Multica 驱动的执行器）。真正需要和 Multica 二选一的，是 Vibe Kanban、OpenHands Agent Canvas 和 GitHub Agent HQ。

### 1.3 推荐组合（详见第 7 章）

```
触达层    飞书群 / IM            —— 触发任务、同步进展（Multica 原生支持 Lark 渠道）
协同层    Multica（自部署）      —— Issue 看板、Agent 身份、Squad 路由、评审门禁、执行日志
执行层    Claude Code / Codex / Hermes / 内部 CLI  —— 真正写代码的那一环
隔离层    专用 Unix 用户 + 容器/VM（自建）          —— Multica 不提供，必须自己加
治理层    成本归集 / 效能指标 / 审计（自建或对接内部平台）—— Multica 只给原料，不给报表
```

### 1.4 三个必须在立项前拍板的风险点

| # | 风险 | 事实依据 | 影响 |
| --- | --- | --- | --- |
| R1 | **License 不是标准 Apache 2.0** | 【一手】`LICENSE` 为"Multica License"= Apache 2.0 正文 + Part I 附加条款。附加条款禁止未获商业授权时对**组织外第三方**提供托管服务；禁止在未获书面豁免时移除 UI 上的 Multica Logo/产品名/版权信息 | 内部单组织使用**明确允许**，但"把它白标成公司内部 DFX 平台的一个模块"需要单独申请 branding waiver |
| R2 | **官方明示不提供文件系统沙箱** | 【一手】官方 Security model 文档原文：`Multica makes no filesystem-sandbox guarantee`；默认以 `danger-full-access`(Codex) / `bypassPermissions`(Claude Code) 运行，Run 继承 daemon 用户的完整权限与 `HOME` | 一次跑飞的 Run 可以读到该用户的 SSH key、云凭据并外发。隔离边界必须由我们自己建 |
| R3 | **Squad 是"路由"不是"编排"** | 【一手】官方 Squads 文档：Leader 通过**发一条 @ 提及的评论**来派活，成员回帖后 Leader 被重新唤醒；没有 DAG、没有状态机、没有断点续跑 | 需要确定性流程（如"必须先出测试用例再改代码"）的场景，Multica 本体做不到，要么靠 Prompt 约束（不可靠），要么在 L1 层自建 |

---

## 2. 评估基线：DFX 场景下的 12 个维度

> 这一套维度是本文所有横向比较的判分依据。落飞书表时按此映射。

| 编号 | 维度 | 在 DFX 工程效能场景下的具体含义 | 判定要点 |
| --- | --- | --- | --- |
| D1 | **任务承载与人机协同** | 需求/缺陷能否作为一等公民被人和 Agent 共同持有；状态流转、评论、评审门禁是否完整 | 有无 Issue 模型、有无 review 状态、人能否中途接管 |
| D2 | **多 Agent 编排与角色分工** | 能否表达"PM 拆解 → RD 实现 → QA 补用例 → CR 复核"这类多角色协作 | 是 LLM 自由路由还是确定性编排；有无并行、有无 barrier |
| D3 | **执行运行时与隔离** | Agent 在哪台机器上、以什么权限跑；一次跑飞的爆炸半径多大 | 容器沙箱 / git worktree / 裸进程；凭据可达范围 |
| D4 | **工具与模型中立性** | 能否同时接 Claude Code、Codex、国产 CLI 和内部自研 CLI；换模型的成本 | 支持的 CLI 数量；是否绑定单一厂商 |
| D5 | **上下文与知识沉淀** | 解决过一次的问题能否复用；跨会话记忆；团队级知识资产 | Skills / Memory / RAG 的机制与共享范围 |
| D6 | **研发工具链集成** | Git 托管（含自建 GitLab）、PR 流程、CI、MCP、IDE | 是否支持自建 Git；PR 是否原生产物 |
| D7 | **可观测与成本治理** | 谁跑的、跑了什么命令、改了什么、花了多少 token | 执行日志粒度；token 归集维度；能否导出 |
| D8 | **权限、安全与合规** | RBAC、SSO、审计、代码是否出域 | 角色模型粒度；有无 SSO/SAML；数据驻留边界 |
| D9 | **私有化部署与运维成本** | 能否完全内网部署；部署与升级的复杂度 | Docker Compose / Helm；依赖组件数量 |
| D10 | **开源许可与可持续性** | 能不能改、能不能白标、社区会不会断供 | License 附加条款；社区活跃度；治理模式 |
| D11 | **国内环境适配** | 飞书/企微/钉钉触达、国产模型与 CLI、网络可达性 | IM 渠道是否官方维护；国产 CLI 支持情况 |
| D12 | **扩展与二次开发** | 加一个内部能力要动多少地方 | API/CLI/SDK 完备度；技术栈与团队匹配度 |

---

## 3. 候选项目明细

### 3.1 Multica（基线，含事实校核）

**定位**：开源的人机混编工作区。把 AI 编码 Agent 变成看板上可被指派的"同事"。【一手】

**架构**【一手，来自官方 README 架构图】：

| 层 | 技术栈 |
| --- | --- |
| Web | Next.js 16（App Router） |
| Desktop | Electron，复用 Web UI 包 |
| Mobile | Expo / React Native（iOS，需自行从源码构建，未上架 App Store） |
| Backend | Go（Chi router、sqlc、gorilla/websocket） |
| Database | PostgreSQL 17（`pgcrypto` + `pg_trgm`） |
| Agent 运行时 | 本地 daemon，拉起 26 个 Agent CLI 中的任意一个 |

控制面与执行面分离：Multica Server 只存数据与调度（不执行任何 Agent 任务），daemon 跑在用户自己的机器上，**每 3 秒拉一次任务、每 15 秒发一次心跳**，拉到任务后拉起本地 AI 编码工具。【一手】

**关键能力明细**：

| 能力 | 事实 |
| --- | --- |
| Agent CLI 覆盖 | 26 个：Claude Code、Codex、Cursor Agent、Copilot CLI、OpenCode、OpenClaw、**Hermes**、Pi、Antigravity、CodeBuddy、DevEco Code、Grok、Kimi、Kiro CLI、Qoder CLI / Qoder CN、Qwen Code、QwenPaw、Reasonix、Trae CLI、DeepSeek Harness、Oh-My-Pi、MiniMax Code、Dim、**华为云 CodeArts**【一手】 |
| 触发方式 | 指派 Issue / 评论 @ / 直接 Chat / Autopilot 定时；**Agent 永不自启动**，每个 Run 都由显式动作触发【一手】 |
| Squad | 一个 Leader Agent + 任意成员（可含人）。指派给 Squad 只唤醒 Leader，Leader 发一条 @ 评论派活，成员回帖后 Leader 被重新唤醒。有防自触发与去重规则【一手】 |
| Skills | 把解决过的问题沉淀成 playbook，全工作区 Agent 复用【一手】 |
| 运行时概念 | runtime = daemon × 一款 AI 编码工具 × 工作区。同机装 2 个工具并加入 2 个工作区 = 注册 4 个独立 runtime【一手】 |
| 可观测 | 执行日志可回放每次工具调用/命令/报错（带时间戳）；token 用量按 Agent 和 Issue 归集【一手】 |
| 权限 | 工作区隔离；`owner`/`admin`/`member` 三级角色；可精确到"谁能跑哪些 Agent"（Access）【一手】 |
| Git 集成 | GitHub、GitLab、Gitea、Forgejo，**含自建实例**【一手】 |
| IM 渠道 | Slack、**Lark（飞书）** 为官方维护；DingTalk、WeCom、Telegram 为**社区维护**【一手】 |
| 部署 | Docker Compose 或 Helm，镜像走 GHCR【一手】 |
| 隔离能力 | 每 Run 独立工作目录（`~/multica_workspaces/`）、每 Run 独立 Agent 状态（如 run-scoped `CODEX_HOME`）、Run 级 API Token（绑定该 Agent 该 Run）。官方明确定性为"降低爆炸半径的便利功能，**不是安全边界**"【一手】 |

**社区热度**：GitHub 约 4.8 万 stars、6 千+ forks、1300+ open issues，仓库创建于 2026-01-13，官方称"大部分工作日都发版，main 移动很快"。【一手 + 二手】

**优势**

1. **心智模型对研发团队零成本**——Issue、指派、评论、Review 列，和 Linear/Jira 一模一样。这是它和 L1 编排框架最大的区别：不需要培训工程师学新概念。
2. **厂商中立是真的中立**——26 个 CLI 里有 Claude Code / Codex，也有 Kimi / Qwen / Trae / DeepSeek / 华为云 CodeArts。换供应商是下拉框，不是迁移。对国内合规与降本切换极其友好。
3. **控制面/执行面分离的架构选得对**——代码不出本机，Server 只存元数据。这个边界让"自部署 Server + 开发机跑 daemon"成为最自然的形态，也是内网落地成本最低的形态。
4. **人机同一套治理**——角色、Access、审计时间线对人和 Agent 是同一套，不需要为 Agent 单独建一套权限体系。
5. **国内适配度在同层里最好**——飞书渠道官方维护、国产 CLI 覆盖全、有中文文档和中文 README。同层的 Vibe Kanban / OpenHands 都没有。
6. **Review 门禁是产品级约束**——活干完落在 in_review，`done` 明确留给人类或 PR 合并事件。不是靠 Prompt 劝说 Agent 别直接合。

**劣势与风险**

1. **无执行沙箱（R2）**——官方文档写得很坦白：边界就是 daemon 用户本身，Run 能读该用户的 SSH key、能用它的云 CLI 凭据、能无限制访问网络。官方推荐的隔离方案（专用 Unix 用户 / 容器 / VM）**全部需要我们自己搭**。对比 OpenHands 自带容器沙箱、Vibe Kanban 自带 git worktree 隔离，这是 Multica 在 D3 维度上的明显短板。
2. **Squad 不是编排引擎（R3）**——Leader 的决策是 LLM 生成的一条评论，路由结果不可预测、不可回放、不可断点续跑。要求"必须先跑测试再提交"这类硬约束，只能写进 Squad Instructions 里劝，做不到强制。对比飞书侧 OpenClaw 多智能体实践中"LLM 只负责生成内容，调度/状态/发布顺序全部由确定性控制面代码保障"的做法，Multica 在 D2 上是弱的。
3. **License 附加条款（R1）**——单组织内部使用没问题，但白标进公司平台需要 branding waiver；且 Part I 条款 2.a 写明"**producer 可按需要把本许可调整得更严格或更宽松**"，贡献者提交代码即视为接受。这是一个真实的长期供应链治理风险，不是形式条款。注意：多份第三方资料（包括一些工具目录站）把它写成 MIT 或 Apache 2.0，**是错的**。
4. **成本治理只到"原料"级**——有 token 用量按 Agent/Issue 归集，但没有预算硬约束、没有超额熔断、没有部门级成本报表。对比 OpenHands Enterprise 明确提供 budget enforcement，这块要自建。
5. **无 SSO/SAML**——三级角色够用，但企业身份体系对接需要二开。
6. **项目非常年轻**——2026 年 1 月才建仓，`main` 快速移动，1300+ open issues。自部署要锁版本、要有回滚预案，不能跟着 main 跑。
7. **daemon 轮询模型**——3 秒一次拉取，任务下发存在秒级延迟；Run 排队依赖 runtime 在线，runtime 离线则任务堆积等待。大规模并发下的调度公平性与队列行为需要压测验证。【推断】

---

### 3.2 Vibe Kanban（同层竞品，轻量路线）

**定位**：本地优先的看板式 AI 编码 Agent 编排壳。开发者的角色从"写代码"变成"描述需求 + 评审 diff"。

**事实**【二手】：Apache 2.0；Rust 后端 + React 前端；`npx vibe-kanban` 一条命令起；每个任务分配独立 **git worktree**；内置 diff 评审与预览浏览器；支持 Claude Code、Cursor CLI、Copilot、Codex、Gemini CLI、Amp、OpenCode、Factory Droid 等；约 2.5 万 stars。

**优势**
- 启动成本极低，一条 npx 命令，没有 Server、没有数据库、没有部署。
- **git worktree 天然隔离**——并行任务互不踩踏，这是比 Multica 更干净的并发模型。
- 评审体验做得扎实（内置 diff + 可编辑 + 合并检查单）。

**劣势**
- **单机单人**，没有多用户、没有角色、没有工作区隔离。团队协同需求直接出局。
- 没有 Squad / 多 Agent 角色分工概念。
- 没有 IM 渠道、没有定时任务、没有成本归集。
- 本地优先意味着没有团队级的沉淀资产（Skills 共享）。

**DFX 适配结论**：**适合个人/小组试点期**，不适合作为团队级 DFX 平台底座。可作为 Multica 落地前的最小验证工具，或作为"worktree 隔离"这一设计的参考实现。

---

### 3.3 OpenHands / Agent Canvas（同层竞品，重基建路线）

**定位**：自托管的编码 Agent 控制中心 + 软件工程 Agent 平台。提供 GUI、CLI、SDK 和企业控制面。

**事实**【一手官网 + 二手】：MIT 许可（开源版）；核心分为 `Agent Canvas`（控制中心）、`software-agent-sdk`（Python SDK / Agent Server / 工具 / 工作区）、`typescript-client`、`automation`（调度、webhook、运行历史）四个仓库。Agent Server 是一台机器上跑多 Agent 的 REST API，Canvas 可连接多个 Agent Server 并切换。支持运行 OpenHands、Claude Code、Codex、Gemini 或任意 **ACP 兼容**的 Agent，后端可以是本地、Docker 容器、VM 或公司基础设施。企业版走 Helm 部署到自有 K8s，组件包括 Server、Runtime API、Runtimes(沙箱)、Keycloak、LiteLLM Proxy、PostgreSQL、Redis，提供 RBAC、SAML/SSO、审计日志、预算强制（budget enforcement）。

**优势**
- **沙箱是一等公民**——每个 Agent 跑在隔离容器里，这正是 Multica 明确不做的那一块。
- **企业治理最完整**——SSO/SAML、RBAC、审计、预算强制、VPC/气隙部署模式，是唯一一个把这些做全的开源系。
- **SDK 化程度最高**——能嵌进自研工具链，不只是一个 UI。
- LiteLLM Proxy 内置，模型路由与配额天然可控。

**劣势**
- **企业能力在商业版闸门后**——K8s 安装需要商业 license，且"目前仅对部分客户按需开放"。开源版的定位是"单用户、本地"（官方部署对比表里 Open Source 一列 Users = 1，无 SSO、无多用户 RBAC）。这意味着**想要那些企业能力，就不是纯开源方案了**。
- **协同心智弱于 Multica**——Canvas 是会话/画布模型，不是 Issue 看板模型。人机混编的"指派-评论-验收"链路没有 Multica 顺。
- **部署重**——完整形态要 Keycloak + Redis + PostgreSQL + Runtime API + 沙箱镜像预加载，运维成本远高于 Multica 的 Compose。
- 国内适配弱：没有飞书/企微渠道，没有国产 CLI 的一等支持。

**DFX 适配结论**：**如果隔离与合规是一票否决项，且能接受商业授权，OpenHands 是最强选项**；如果坚持纯开源 + 团队协同体验，Multica 更合适。二者也可共存：Multica 管协同，OpenHands Agent Server 当受控执行后端。【推断，需 POC】

---

### 3.4 Hermes Agent（互补件，不是竞品）

**定位**：Nous Research 开源的**自进化个人 Agent 运行时**。与 Claude Code、Codex、OpenClaw 同类——通过 tool calling 与系统交互的自主编码/任务执行 Agent。

**注意**：Hermes 是 **Multica 已内置支持的 26 个 CLI 之一**（命令 `hermes`）。把它和 Multica 对比，等价于把"发动机"和"调度中心"对比。

**事实**【一手官方文档 + 二手代码走读】：
- Python 实现，核心是 `run_agent.py` 的 `AIAgent` 类：系统提示词组装、工具 schema、provider 选择（`chat_completions` / `codex_responses` / `anthropic_messages` 三种 API 模式）、可中断的模型调用、串行或线程池并发的工具执行、压缩/重试/降级换模型、父子 Agent 的迭代预算追踪、上下文丢失前的持久记忆刷写。
- **学习闭环是它的差异化卖点**：从经验中创建技能、在使用中改进技能、主动持久化知识、搜索过往会话、跨会话逐步建立对用户的理解。有 `/curator` 做后台技能维护。
- **多平台网关**：同一个 Agent 核心可跑在 Telegram、Discord、Slack、WhatsApp、iMessage、Signal、Matrix、Teams、Email 等十几个平台。
- **多界面**：CLI、Ink TUI、Electron 桌面应用、Web 仪表盘，以及面向 VS Code / Zed / JetBrains 的 **ACP server**。
- **多执行后端**：local、Docker、SSH、Modal、Daytona、Singularity。
- 有子代理（隔离上下文 + 受限工具集）、cron 调度、MCP、`/kanban` 多 profile 协作板。
- `tools/code_execution_tool.py` 提供一个有意思的设计：让模型写一小段 Python 脚本，通过本地 RPC stub 访问受限工具面，而不是每步都回主循环——显著降低机械任务的轮次消耗与上下文膨胀。
- Provider 无关，支持 OpenRouter、Anthropic、OpenAI、Google、DeepSeek、xAI 及本地模型等 20+ 家。

**优势**
- **记忆与技能的深度是同层最强**——Multica 的 Skills 是静态 playbook，Hermes 是会自我改写的技能库。D5 维度上 Hermes 明显更强。
- **执行后端可插拔（含 Docker/SSH/远程沙箱）**——正好能补 Multica 的 D3 缺口。
- **ACP 支持**意味着它能同时被 IDE 和 OpenHands Canvas 这类平台驱动。
- 轻量，能在 $5 VPS 上跑。

**劣势**
- **单 Agent 视角**——它的多 Agent 是"子代理"，服务于一次任务的分解，不是团队级的多人多 Agent 协同。没有 Issue、没有指派、没有验收门禁。
- 面很宽（CLI + 网关 + 记忆 + 技能 + RL/eval 轨道全在一个仓库），**边界不清晰，二开时要判断哪些部分是稳定面**。
- 企业治理能力基本为零：无 RBAC、无审计、无成本归集。
- 消息平台网关对企业内网环境意义有限（主要是 Telegram/Discord 这类）。

**DFX 适配结论**：**作为 Multica 的执行器候选之一引入**，特别是在需要跨会话记忆/技能自进化的场景（例如长期维护同一个老系统的 Agent）。不作为独立方案评估。

---

### 3.5 OpenClaw + 飞书多智能体实践（互补件 + 编排范式参考）

**定位**：OpenClaw 是开源多智能体框架，同样是 Multica 支持的 CLI 之一（`openclaw`）；"飞书 OpenClaw"是飞书官方推出的一键部署方案，把 OpenClaw 与飞书深度整合。

**事实**【一手 OpenClaw 官方文档 + 二手实践文章】：
- **多智能体路由模型**：一个 Gateway 进程内跑多个互相隔离的 Agent，每个有自己的 workspace、状态目录 `agentDir`、SQLite 会话历史，以及多个渠道账号。`binding` 按 `(channel, accountId, peer)` 把入站消息**确定性路由**到 `agentId`（第一个匹配生效，最具体规则优先）。
- **A2A 通信**：`tools.agentToAgent` 控制 Agent 间消息，可用 `allow` 白名单限定可通信的 Agent 对，可 `enabled: false` 整体关闭；`session.agentToAgent.maxPingPongTurns` 限制来回轮次防死循环。
- **会话工具**：`sessions_spawn` 起子代理并可选等待、`sessions_send` 发消息并等回复、`sessions_yield`、`sessions_subagents` 等完整 spawn 生命周期；`tools.sessions.visibility` 有 `all`/`tree`/`agent`/`self` 四档可见性。
- **官方预设**：coordinator（chief of staff）+ researcher + writer + reviewer，各有独立 workspace 与身份；协调者对接人类，分派有界的工作、检查产物、汇总结果；专家只回交产物与证据，**不再向下转派**。行为规范写在 `AGENTS.md` 里，spawn 出的会话同样生效。
- **飞书侧生产实践**（`OpenClaw-Feishu-Multi-Agent` 等）：一套配置管多个飞书群，每群一个"主管 + 多专家"的虚拟团队。核心设计是——**LLM 只负责生成内容，任务调度、状态管理、消息顺序发布全部由确定性的控制面代码保障**；专家不直接往群里发消息，而是把结构化 JSON（草稿、摘要、风险点）回调给控制面，控制面按预设 `publishOrder` 顺序发布。配置用声明式蓝图（`teams` + `roleCatalog`）自动生成 OpenClaw 的 `channels.feishu` / `bindings` / `agents` 配置，支持增量部署、灰度和回滚。

**优势**
- **确定性路由与确定性发布顺序**——这是 Multica Squad 最欠缺的东西。`bindings` 是规则匹配不是 LLM 判断，`publishOrder` 是代码保证不是 Prompt 期望。
- **A2A 有显式的安全闸门**（白名单 + 轮次上限 + 会话可见性四档），比"全靠 @ 提及"可控得多。
- **飞书群原生就是工作界面**，对非研发角色参与的场景友好。
- 声明式配置 + 生成脚本的做法，让"加一个团队 = 数组里加一项"，运维模型清晰。

**劣势**
- **不是研发工作流工具**——没有 Issue、没有 PR 门禁、没有代码 diff 评审、没有 Git 托管集成。它的产物是群消息和文档，不是合入主干的代码。
- 会话可见性默认 `all`（未沙箱会话可跨 Agent 列举/读取/搜索，**可能包含其他用户的对话记录**），企业环境必须显式收紧，否则是数据泄露面。
- 飞书多智能体那套控制面是社区/项目级实践，不是飞书官方产品承诺，稳定性与长期维护要自己兜。
- 配置复杂度高（`agents.list` + `agentToAgent.allow` + `bindings` 三处联动），容易配错。

**DFX 适配结论**：**不作为主方案，但它的编排范式值得直接借鉴**——如果我们要补 Multica 的 D2 缺口，"确定性控制面 + LLM 只生成内容 + 结构化 JSON 回调 + 顺序发布"是一套已被生产验证的设计，可以照搬到自建编排层上。

---

### 3.6 飞书 aily（智能伙伴创建平台）

**定位**：飞书官方的**企业级智能体开发平台**，围绕 LLM 提供 AI 技能编排、知识数据处理、效果调优与持续运营，一键发布到飞书、Web 等渠道。

**事实**【二手，来自飞书官网内容页】：
- 闭源 SaaS，非开源方案。
- 能力：丰富数据源接入（文档、多维表格、数据库）、智能预处理（打标、智能切片）、全链路评测反馈与术语平台、多种召回排序策略、Agent 智能规划与执行（可调度上百个企业服务连接器与飞书协同套件）、可视化工作流引擎（白盒化编排）、单节点效果优化（引导式编写、多模型评测、辅助调优）。
- 2026 年升级"自定义智能体"：由管理员统一搭建管控，关联知识空间自动更新，可操作飞书文档/多维表格/任务，支持定时任务主动工作。有"标准模式"（仅开发者在后台改，发布生效）与"无感发布模式"（在飞书/aily 专业版里通过对话改人设、教技能）两种。
- 官方自述定位差异：相比专注个人效率的 aily 智能伙伴、轻量的妙搭，自定义智能体"更适合需要体系化设计、对数据安全和智能体行为有管控要求的团队"。

**优势**
- 与飞书生态深度打通（文档、多维表格、任务、审批、知识空间），这些恰好是 DFX 专项的需求与文档所在地。
- 有管理员统一管控、权限与数据治理，符合企业合规习惯。
- 非技术角色可参与搭建，覆盖研发以外的效能场景（报表汇总、流程提醒、问答）。
- 效果调优链路（评测、术语、召回策略）是研发向工具普遍不具备的。

**劣势（针对本专项）**
- **闭源、不可私有化**——无法审计、无法二开、无法内网部署。
- **不碰代码仓库**——没有 Git 集成、不生成 PR、不执行构建与测试。它调度的是"企业服务连接器"，不是编码 Agent CLI。
- 没有代码执行运行时与沙箱概念。
- 与 D1/D3/D6 三个核心维度基本不相交。

**DFX 适配结论**：**与 Multica 不是替代关系，是分工关系**。建议的分工是：aily 承载"文档/表格/流程类"的效能智能体（例如自动汇总专项进展、维护需求表），Multica 承载"代码类"的工程 Agent；两者在飞书群里汇合。

---

### 3.7 Coze Studio / Dify（应用平台层，非同类）

**事实**【二手】：
- **Coze Studio**：字节 Coze 的开源版，Apache 2.0，Go 后端 + React/TS 前端，微服务 + DDD。全可视化拖拽 Agent 开发，知识库、插件、Workflow 一站式，Docker 快速部署。强调多 Agent 协同与长期记忆。已知短板：高度封装导致扩展性受限；开源版**不支持多租户/多空间**，每个用户单独注册，无法团队协作开发；模型配置需改 yaml 重启。
- **Dify**：开源 LLM 应用生产平台，把 RAG、Workflow、Agent、模型管理、发布、观测放进同一套工程系统；Workflow 可发布成 API / Web App / MCP Server；支持广泛向量库与检索策略，短期/长期记忆分层清晰，调试能力完善，Docker Compose 部署。定位偏"在自己系统里落一个可控的 AI 应用后端"。

**优势**：知识库/RAG 与工作流编排能力远超 Multica；Dify 的自托管与工程化程度高；都是 Apache 2.0，二开无附加条款负担。

**劣势（针对本专项）**：它们编排的是"LLM 调用 + 工具节点"，不是"编码 Agent 在真实仓库上干活"。没有 daemon、没有 runtime、没有 Git worktree、没有 PR 产物。用它们做 DFX 编码任务，等于从零自己实现执行层。

**DFX 适配结论**：**作为知识/RAG 能力的补位候选**。如果专项需要"内部规范、历史缺陷、架构文档"的检索增强，Dify 可以作为知识服务挂在 Multica 的 Agent 后面（通过 MCP 或 HTTP 工具）；不作为协同层候选。

---

### 3.8 LangGraph / CrewAI / Microsoft Agent Framework（编排框架层）

**事实**【二手，2026 年公开对比资料】：
- **LangGraph**：有向图 + 类型化状态 + checkpointer（开发用 MemorySaver，生产用 SQLite/Postgres）；原生 `interrupt()` 支持 human-in-the-loop 并可 resume；支持时间旅行调试与断点恢复；Python + JS；被 Klarna、LinkedIn、Uber、Replit 等在规模场景使用。**生产就绪度公认最高**，代价是周边（持久化后端、重试、追踪、护栏、运行环境）全要自己装配。
- **CrewAI**：角色制（Agent 即员工）Crew 模型，上手最快，100+ 内置工具、Pydantic 结构化输出；但持久执行与 HITL 是短板，关键链路常需迁移到 LangGraph。
- **AutoGen**：**已进入维护模式，新项目不建议采用**。微软的继任者是 **Microsoft Agent Framework (MAF)**，2026-04-02 GA，合并了 AutoGen 与 Semantic Kernel，一等公民地提供 sequential / concurrent / handoff / group chat / Magentic 五种编排原语，外加带 checkpointing、streaming、HITL、time-travel 的图式工作流；Python + .NET；但护栏、PII 保护、prompt 注入防御、托管 Agent 管理、可读 trace 这些"让 Agent 在生产环境安全"的部分要走 Azure AI Foundry 才点亮。AG2 是社区维护的 AutoGen drop-in fork。

**在本专项中的意义**：这一层**不与 Multica 竞争，而是填补 Multica 的 D2 空白**。如果确认需要"确定性多阶段流程 + 断点续跑 + 人工审批节点"，LangGraph 是最合适的补位（provider 无关、可内网、无云绑定）。MAF 若非 .NET/Azure 技术栈，引入收益不明显。

---

### 3.9 GitHub Agent HQ（闭源商业基线）

**事实**【二手】：GitHub 在 2025 年底发布、2026 年扩展的多 Agent 编排层。Mission Control 作为统一控制台，可跨仓库指派任务给 Copilot / Claude / Codex，看实时会话日志，运行中干预（暂停、修正、重启），直接跳到产出的 PR。核心工作流就是 GitHub 原生的 Issue → 指派 → 草稿 PR → 评审 → 合并。用仓库内 `AGENTS.md` 定义 Agent 人设与上下文；用 MCP 做工具契约。支持"同一 issue 派给多个 Agent 并行，对比 PR 择优合并"的竞争式模式。随 Copilot 套餐提供（Claude/Codex 接入需 Pro+/Enterprise 档）。

**优势**：与 GitHub 原生工作流零摩擦；企业级访问控制、审计日志、指标齐备；"多 Agent 竞争同一 issue 再择优"这个模式很有价值。

**劣势（针对本专项）**：闭源、SaaS、绑定 GitHub 与 Copilot 商业套餐；无法内网部署；国内可用性与合规存疑；不支持国产 CLI 与模型。

**在本专项中的意义**：作为**功能完备度的参照基线**——用它来检查我们的自建/开源方案还缺什么（例如"并行竞争 + 择优合并"这个模式，Multica 目前没有原生支持，值得在需求清单里单列）。

---

## 4. 横向能力矩阵

评分：● 强 / ◐ 部分 / ○ 弱或无 / — 不适用

| 维度 | Multica | Vibe Kanban | OpenHands | Hermes | OpenClaw | 飞书 aily | Dify/Coze | LangGraph | Agent HQ |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| D1 任务承载与人机协同 | ● | ◐ | ◐ | ○ | ○ | ◐ | ○ | ○ | ● |
| D2 多 Agent 编排 | ◐ (仅路由) | ○ | ◐ | ◐ (子代理) | ● (确定性路由) | ◐ | ● (工作流) | ● | ◐ |
| D3 执行运行时与隔离 | ○ (**无沙箱**) | ◐ (worktree) | ● (容器沙箱) | ● (多后端可插) | ◐ | — | — | — | ● |
| D4 工具/模型中立性 | ● (26 CLI) | ● | ● (ACP) | ● (20+ provider) | ● | ○ | ● | ● | ◐ (三家) |
| D5 上下文与知识沉淀 | ◐ (Skills) | ○ | ◐ | ● (自进化技能+记忆) | ◐ | ● (知识空间+RAG) | ● (RAG) | ◐ | ◐ |
| D6 研发工具链集成 | ● (含自建 Git) | ◐ | ● | ◐ | ○ | ○ | ○ | ○ | ● (仅 GitHub) |
| D7 可观测与成本治理 | ◐ (日志+token) | ○ | ● (企业版预算强制) | ○ | ○ | ◐ | ● | ◐ (LangSmith) | ● |
| D8 权限/安全/合规 | ◐ (RBAC，**无 SSO**) | ○ | ● (企业版 SAML/审计) | ○ | ◐ | ● | ◐ | ○ | ● |
| D9 私有化部署 | ● (Compose/Helm) | ● (本地) | ◐ (K8s 需商业授权) | ● | ● | ○ (**不可私有化**) | ● | ● | ○ |
| D10 许可与可持续性 | ◐ (**附加条款**) | ● (Apache 2.0) | ◐ (开源 MIT/企业闭源) | ● | ● | ○ | ● (Apache 2.0) | ● (MIT) | ○ |
| D11 国内环境适配 | ● (飞书官方+国产 CLI) | ○ | ○ | ○ | ● (飞书深度) | ● | ● | ◐ | ○ |
| D12 扩展与二次开发 | ● (CLI/API 全覆盖) | ◐ | ● (SDK) | ◐ (边界不清) | ◐ | ○ | ● | ● | ○ |

**读表要点**：

- Multica 的 ● 集中在 D1/D4/D6/D9/D11/D12——**协同体验、生态中立、国内落地**这三件事它做得最好。
- Multica 的 ○ 在 D3——**这是唯一的硬伤**，而且官方是明示不做，不是还没做。
- 没有任何单一方案在所有维度拿满，**组合是必然结论**。

---

## 5. 同一需求的不同实现路径对比

> 本章是"基于同样需求实现"的核心。每个场景给出各方案的具体落法、需要自建的部分、以及优劣。

### 场景 A：需求单 → Agent 实现 → PR → 人工评审合入

*最主流的 DFX 场景，占预期用量的大头。*

| 方案 | 具体落法 | 需要自建 | 优 | 劣 |
| --- | --- | --- | --- | --- |
| **Multica** | 建 Issue → 指派给 Agent → daemon 拉取 → 本地 CLI 执行 → 边做边评论 → 落 in_review → 人评审 → PR 合入触发 done | Git 仓库与凭据的隔离账号 | 原生链路，零开发；意图/执行/决策/diff 挂在同一个 Issue 下，上下文不丢 | 执行无沙箱；PR 由 CLI 自己开，Multica 不介入 PR 模板/门禁 |
| **Vibe Kanban** | 建卡 → 选 Agent → 自动开 worktree → 内置 diff 评审 → 合并 | 团队协同的一切 | worktree 隔离干净，评审体验最好 | 单机单人，无法团队化 |
| **OpenHands** | Canvas 起会话或 automation 派单 → 容器沙箱内执行 → 产出 PR | Issue 层的协同语义 | 沙箱隔离；企业版有审计与预算 | 无 Issue 看板心智，多人协作要自己贴 |
| **Hermes 单飞** | `hermes` CLI 或网关触发 → 本地/Docker/SSH 后端执行 | Issue、指派、评审门禁全部自建 | 记忆与技能最强 | 等于自研一个 Multica |
| **飞书 aily** | 不可行 | — | — | 不碰代码仓库，场景不成立 |
| **自研** | — | 全部 | 完全贴合内部流程 | 成本最高，且要长期跟进 26 个 CLI 的适配 |

**结论**：Multica 原生覆盖，其他方案要么缺协同、要么缺执行。**选 Multica，隔离问题单独解。**

---

### 场景 B：多角色流水线（PM 拆解 → RD 实现 → QA 用例 → CR 复核）

*这是"MultiAgent 协作"专项真正的靶心，也是分歧最大的场景。*

| 方案 | 具体落法 | 确定性 | 优 | 劣 |
| --- | --- | --- | --- | --- |
| **Multica Squad** | 建 Squad（Leader + 前端/后端/测试 Agent）→ Issue 指派给 Squad → Leader 读 Roster 与 Squad Instructions → **发一条 @ 评论派活** → 成员回帖 → Leader 被重新唤醒 → 决定下一步或推 in_review | **低**：路由是 LLM 生成的评论 | 零代码；人和 Agent 可以在同一个 Squad 里；有防自触发与去重 | 无法强制阶段顺序（"必须先出用例"只能靠 Instructions 劝）；无 barrier、无断点续跑；Leader 每轮都消耗一次模型调用 |
| **OpenClaw / 飞书多智能体范式** | 声明式蓝图定义 `teams` + `roleCatalog` → 生成 `agents`/`bindings` 配置 → 控制面代码拆解任务、并行分派 → 专家回交结构化 JSON → 控制面按 `publishOrder` 顺序发布 | **高**：调度与顺序由代码保障，LLM 只生成内容 | 顺序、状态、发布全可控；可灰度可回滚；配置声明式 | 产物是消息/文档不是 PR；无 Git 集成；配置三处联动易错 |
| **LangGraph** | 把四个角色写成图节点，类型化状态流转，`interrupt()` 挂人工审批，Postgres checkpointer 持久化 | **最高**：显式图 + 断点恢复 + 时间旅行 | 可回放、可恢复、审批是一等公民 | 纯代码，没有 UI，没有看板；要自己接执行器与 Git |
| **Multica Squad + LangGraph** | Multica 承载 Issue 与人机界面，把"多阶段流程"这一段委托给一个 LangGraph 服务（作为一个特殊 Agent 或 MCP 工具暴露） | 高 | 两边的长处都拿到 | 需要自建集成层，是本方案里工作量最大的一块 |

**结论**：**Multica Squad 只能覆盖"路由"，覆盖不了"流程"。** 如果 DFX 需求里存在硬性的阶段顺序或质量门禁（大概率存在），必须在 L1 层补一个确定性编排件，推荐 LangGraph，实现范式参考 OpenClaw 飞书实践的"控制面 + 结构化回调 + 顺序发布"。

---

### 场景 C：定时巡检 / 质量报告 / 存量代码批量整改

| 方案 | 具体落法 | 优 | 劣 |
| --- | --- | --- | --- |
| **Multica Autopilots** | cron 配置定时任务，跑站会汇总、代码审计、报告生成 | 原生，无需开发；产物落在 Issue 里可追溯 | 大批量仓库时受 runtime 数量限制（runtime = daemon × 工具 × 工作区），并发能力需压测 |
| **OpenHands automation** | 独立 automation 仓负责调度、webhook、运行历史 | 调度是独立组件，能力更强；容器后端便于横向扩 | 报告落地到哪里要自己定 |
| **Hermes cron** | `/cron` 管理定时任务，复用同一 Agent 核心 | 轻量 | 单机视角，无团队级汇总 |
| **飞书 aily 定时任务** | 定时主动工作，直接操作飞书文档/多维表格 | **报告直接落到飞书表格**，这是它不可替代的地方 | 不能读代码仓库 |

**结论**：**混合最优**——代码侧巡检用 Multica Autopilots，结果通过飞书渠道推送；报表/表格维护用 aily 定时任务。

---

### 场景 D：跨仓库大规模重构（同一改动 × N 个仓库）

| 方案 | 可行性 | 说明 |
| --- | --- | --- |
| **Multica** | ◐ | 可以批量建 Issue 批量指派，但没有"一个母任务 fan-out 到 N 个仓库再汇总"的原语；并发受 runtime 数量约束；失败重试是 Run 级不是批次级 |
| **Vibe Kanban** | ○ | 单机 worktree，规模上不去 |
| **OpenHands** | ● | Agent Server + 容器后端天然可横向扩，automation 可编排批次 |
| **LangGraph 自建** | ● | fan-out/fan-in 是图的基本能力，失败重试与断点恢复原生 |
| **Agent HQ** | ● | Mission Control 支持跨仓库批量指派 |

**结论**：这是 **Multica 的第二个能力缺口**。如果专项包含大规模整改类需求，需要在 Multica 之上做一个批次调度器（读 Multica CLI/API 批量建单 + 汇总），或直接把这类任务路由到 OpenHands/自建 LangGraph 流水线。

---

### 场景 E：飞书群内触发与进度同步

| 方案 | 具体落法 | 优 | 劣 |
| --- | --- | --- | --- |
| **Multica Lark 渠道** | 官方维护的 Lark channel，在群里触发和跟进 Agent 工作 | 官方维护（不是社区维护，区别于钉钉/企微）；与 Issue 时间线打通 | 交互深度未知，需 POC 验证（例如能否在群里直接看 diff、能否审批） |
| **飞书 OpenClaw** | 每群一个"主管 + 专家"团队，A2A 协作，控制面顺序发布 | 群内体验最好，非研发角色可直接参与 | 与代码仓库脱节 |
| **飞书 aily 自定义智能体** | 一键加入飞书群，可操作文档/表格/任务 | 与飞书协同套件深度打通 | 闭源、不碰代码 |
| **自建 Webhook** | 监听 Multica 事件推飞书 | 完全可控 | 要自己做 |

**结论**：**先验证 Multica 的 Lark 官方渠道够不够用**。如果交互深度不足（例如无法在群里审批），用自建 Webhook 补；aily/OpenClaw 用于非研发侧场景。

---

## 6. 优劣势总表

| 方案 | 核心优势（针对本专项） | 核心劣势 | 主要风险 | 建议 |
| --- | --- | --- | --- | --- |
| **Multica** | 人机混编协同心智最顺；26 CLI 厂商中立含国产；自建 Git 全支持；飞书官方渠道；Compose/Helm 自部署；CLI/API 全可脚本化 | 官方明示无文件系统沙箱；Squad 只路由不编排；无 SSO；无预算熔断；跨仓库批量原语缺失 | License 附加条款（白标需豁免、producer 可单方收紧）；项目仅 8 个月，main 快速移动 | **主线采用**，锁版本，隔离层与编排层自建 |
| **Vibe Kanban** | 零部署；worktree 隔离干净；diff 评审体验最好 | 单机单人；无团队/角色/渠道/成本 | 无 | 作为个人试点工具，不入平台 |
| **OpenHands** | 容器沙箱一等公民；企业治理最完整（SAML/审计/预算）；SDK 化程度最高 | 企业能力在商业授权后；协同心智弱；部署重；国内适配弱 | 开源版定位单用户，能力与商业版落差大 | **备选路线**，或作为受控执行后端与 Multica 共存 |
| **Hermes Agent** | 自进化技能与跨会话记忆最强；执行后端可插拔（Docker/SSH/Modal/Daytona）；ACP 支持 | 单 Agent 视角；无协同/治理；仓库边界不清 | 二开时稳定面难判断 | **作为 Multica 的执行器之一接入**，用于长期维护类任务 |
| **OpenClaw / 飞书多智能体** | 确定性路由与顺序发布；A2A 有白名单与轮次上限；飞书群原生 | 不产出 PR；无 Git 集成；默认会话可见性过宽 | 社区实践非官方承诺 | **借鉴其编排范式**；非研发场景可直接用 |
| **飞书 aily** | 飞书生态深度打通；管理员统一管控；非技术角色可用；效果调优链路完整 | 闭源不可私有化；不碰代码仓库 | 无法审计、无法二开 | **分工使用**：承载文档/表格/流程类效能智能体 |
| **Dify / Coze Studio** | RAG 与工作流编排能力强；Apache 2.0 无附加条款 | 不面向"真实仓库上干活"的编码 Agent | Coze 开源版无多租户 | **作为知识/RAG 能力补位**，通过 MCP 挂给 Multica 的 Agent |
| **LangGraph** | 确定性图编排 + checkpoint + 原生 HITL + 时间旅行；provider 无关可内网 | 纯代码无 UI；周边全要自装配 | 无 | **补 D2 缺口的首选** |
| **GitHub Agent HQ** | 与 GitHub 原生零摩擦；治理完整；支持"多 Agent 竞争择优" | 闭源 SaaS；绑定 Copilot 套餐；无法内网；无国产 CLI | 国内可用性与合规 | **作为功能完备度参照基线**，不采用 |

---

## 7. 选型建议

### 7.1 推荐主线：Multica 为协同底座的组合方案

**为什么是 Multica 而不是 OpenHands**：本专项的第一性需求是"让人和 Agent 在同一套工程流程里协作"，这是 D1 的事。Multica 在 D1 上的完成度、以及 D4/D6/D9/D11 上的国内适配度，是 OpenHands 开源版给不了的。OpenHands 强在 D3/D7/D8，但那些能力在**商业授权闸门后**——一旦付费，"开源方案"的前提就不成立了，评估基准要换。

**为什么不自研**：Multica 已经解决的部分（26 个 CLI 的适配与持续跟进、daemon/runtime 模型、Issue-Run-Comment 的数据模型、桌面端/移动端、四种 Git 托管集成）自研成本极高且需要长期投入，而这些恰恰不是我们的差异化。我们应该把工程投入放在它明确不做的两件事上：**隔离**和**确定性编排**。

### 7.2 必须自建的能力清单

| # | 能力 | 为什么 Multica 给不了 | 建议做法 |
| --- | --- | --- | --- |
| 1 | **执行隔离** | 官方明示不提供文件系统沙箱，且这是刻意设计（避免破坏需要正常 HOME 的工具链） | 按官方推荐从轻到重三选一：专用 `multica` Unix 用户 → 容器（只挂需要的目录与 secret）→ VM。凭据用 per-purpose deploy key，不用个人 SSH key，daemon 用户的 home 里不放任何生产凭据 |
| 2 | **确定性阶段编排** | Squad 的路由是 LLM 生成的评论，不可预测不可恢复 | 用 LangGraph 建一个编排服务，通过 MCP 或 HTTP 工具暴露给 Multica 的 Agent。范式参考 OpenClaw 飞书实践：LLM 只生成内容，调度/状态/顺序由代码保障，专家回交结构化 JSON |
| 3 | **成本治理与预算熔断** | 只有 token 用量归集，没有预算硬约束 | 消费 Multica 的 Run/token 数据，做部门级归集与阈值告警；必要时在 LLM 网关层（LiteLLM 或内部网关）做硬熔断 |
| 4 | **效能指标** | 没有 DFX 需要的度量口径 | 从 Run 日志与 Git 事件构建指标：Agent 任务采纳率、返工率、人工介入次数、单需求交付时长、单位需求 token 成本 |
| 5 | **跨仓库批次调度** | 无 fan-out/fan-in 原语 | 基于 Multica CLI/API 写批次调度器；或把这类任务路由到独立流水线 |
| 6 | **企业身份对接** | 无 SSO/SAML | 二开或前置网关做身份代理 |
| 7 | **版本与供应链管控** | main 快速移动 + License 可被单方收紧 | 锁定发版版本、镜像入内部仓库、保留 fork、定期 review LICENSE 变更 |

### 7.3 备选路线（触发条件）

| 触发条件 | 切换到 |
| --- | --- |
| 安全评审判定"无沙箱"为一票否决，且可接受商业授权 | OpenHands Enterprise |
| 主要需求是流程编排而非人机协同，且团队愿意接受纯代码方案 | LangGraph 自建 + 轻量 Web 界面 |
| 主要需求在非研发侧（文档、表格、流程） | 飞书 aily + 飞书 OpenClaw |
| Multica License 后续收紧到不可接受 | fork 当前版本自维护，或迁移到 OpenHands/Vibe Kanban（数据模型差异需评估迁移成本） |

### 7.4 分阶段落地（按能力递进，非时间排期）

**阶段一 · 可用性验证**
自部署 Multica（Compose，锁版本）+ 专用 Unix 用户隔离 + 接入 2 个 Agent CLI（一个国际、一个国产）+ 打通一个自建 GitLab 仓库。验收：场景 A 端到端跑通，人能在 Issue 里完整看到意图、执行日志、diff。

**阶段二 · 边界收敛**
把 daemon 迁入容器，凭据收敛到 per-purpose deploy key，接通飞书 Lark 渠道，接入 Autopilot 做第一个定时巡检。验收：一次刻意跑飞的 Run 无法读到该容器外的任何凭据。

**阶段三 · 编排补位**
引入 LangGraph 编排服务，实现场景 B 的一条硬门禁流水线（例如"改动必须先有测试用例"）。验收：流程顺序在代码层被强制，不依赖 Prompt。

**阶段四 · 治理闭环**
成本归集与预算告警、效能指标看板、批次调度器。验收：能回答"上月 DFX Agent 花了多少钱、采纳率多少、返工率多少"。

---

## 8. POC 验证清单

以下每一条都是当前资料无法确定、必须实测的：

| # | 待验证 | 方法 | 判定标准 |
| --- | --- | --- | --- |
| P1 | Multica Lark 官方渠道的交互深度 | 群内触发一个 Issue，观察能做什么 | 能否在群内看进展、看 diff、做审批 |
| P2 | daemon 3 秒轮询在高并发下的调度行为 | 同时压入 50+ Run，观察队列与公平性 | 无饥饿、无重复领取、延迟可接受 |
| P3 | runtime 离线/恢复的任务行为 | 执行中 kill daemon | 任务状态正确、可恢复或明确失败 |
| P4 | Squad Leader 的路由稳定性 | 同一类 Issue 重复 20 次指派给同一 Squad | 路由结果一致性比例（预期偏低，用于量化 R3） |
| P5 | 容器化 daemon 下各 CLI 是否正常 | 在容器里跑 Claude Code / 国产 CLI | 登录态、工具链、网络均正常（官方警告过分部沙箱会导致"未登录"类隐性故障） |
| P6 | 自建 GitLab 集成完整度 | 接自建实例 | PR/MR 创建、评论回写、合并事件回传均正常 |
| P7 | token 用量数据的可导出性 | 查 API/CLI | 能按 Agent、Issue、时间窗导出 |
| P8 | 升级路径 | 跨若干版本升级一次 | 数据迁移无损，有回滚手段 |
| P9 | License 商业条款确认 | 与 producer 书面确认 | 内部白标是否需要 branding waiver、如何申请 |
| P10 | 国产 CLI 的实际可用度 | 实测 Kimi / Qwen / Trae / CodeArts | 与 Claude Code 的任务成功率差距 |

---

## 9. 风险登记

| ID | 风险 | 等级 | 缓解措施 |
| --- | --- | --- | --- |
| R1 | License 附加条款：白标需豁免；producer 可单方收紧 | 高 | 书面确认 waiver；保留 fork；定期 review LICENSE |
| R2 | 无执行沙箱，凭据可被 Run 读取并外发 | 高 | 容器/VM 隔离 + 凭据最小化 + 出网管控 + Run 命令审计 |
| R3 | Squad 非确定性，无法表达硬门禁 | 中高 | L1 层补 LangGraph 编排；关键门禁放在 CI 而非 Agent |
| R4 | 项目年轻，main 快速移动，1300+ open issues | 中 | 锁版本 + 内部镜像仓 + 升级前回归 |
| R5 | 无 SSO、无预算熔断 | 中 | 网关层代理身份；LLM 网关层做成本硬约束 |
| R6 | 跨仓库批量能力缺失 | 中 | 自建批次调度器 |
| R7 | 国产 CLI 的实际能力差距 | 中 | P10 实测后按任务类型分流 Agent |
| R8 | 社区维护渠道（钉钉/企微）质量不可控 | 低 | 仅用官方维护的 Lark 渠道；其余走自建 Webhook |

---

## 10. 参考资料

**一手**
- Multica 仓库与 README（英/中）：https://github.com/multica-ai/multica
- Multica LICENSE（Multica License = Apache 2.0 + Part I 附加条款）：https://github.com/multica-ai/multica/blob/main/LICENSE
- Multica 官方文档：https://multica.ai/docs — 其中 `how-multica-works`、`agents`、`squads`、`security-model`、`self-host-quickstart`
- OpenClaw 多智能体路由与会话工具文档：https://docs.openclaw.ai/zh-CN/concepts/multi-agent 、https://docs.openclaw.ai/concepts/session-tool
- Hermes Agent 官方文档与仓库：https://hermes-agent.nousresearch.com/docs 、https://github.com/NousResearch/hermes-agent
- OpenHands 官网/文档/仓库：https://www.openhands.dev 、https://docs.openhands.dev/enterprise 、https://github.com/OpenHands/openhands

**二手**
- 飞书 aily 智能体定制能力升级（飞书官网内容页）
- 飞书 OpenClaw AgentTeam 多智能体协作（飞书官网内容页）
- OpenClaw-Feishu-Multi-Agent 生产级部署实践（CSDN）
- Vibe Kanban 项目评述与官网
- Multi-Agent Orchestration Frameworks Compared（Reinvently）
- LangGraph / CrewAI / AutoGen / MAF 2026 生产就绪度对比（ODSEA、Pickaxe、FutureAGI）
- GitHub Agent HQ 与 Mission Control（GitHub Blog 及第三方指南）
- Dify / Coze Studio 深度对比（多来源）
