# Claude Code 2.1.88 恢复工程说明

本仓库是基于已发布包中的 `cli.js` 与 `cli.js.map` 反推恢复出来的 Claude Code 源码树。当前目标不是“完全还原官方开发仓库历史”，而是提供一个可以继续编译、启动、调试、补模块和二次开发的可维护工程。

当前验证状态：

- 可以执行 `npm_config_cache=.npm-cache npm run build`
- 可以执行 `npm run cli:status`
- 可以执行 `npm run cli:run -- -p "Reply with exactly: OK"`
- 可以执行 `npm run cli:run` 进入交互界面

相关手册：

- 编译手册：[docs/BUILD_MANUAL.md](/Users/test/Downloads/claude-code/package/claude-code-2.1.88/docs/BUILD_MANUAL.md)
- 二次开发手册：[docs/SECONDARY_DEVELOPMENT_MANUAL.md](/Users/test/Downloads/claude-code/package/claude-code-2.1.88/docs/SECONDARY_DEVELOPMENT_MANUAL.md)

友链：

- https://linux.do/

## 快速开始

```bash
npm install
npm_config_cache=.npm-cache npm run build
npm run cli:status
npm run cli:run
```

如果你希望运行时继承全局 `~/.claude/settings.json` 中的代理和认证环境，直接使用上面的 `npm run cli:run` 即可。  
如果你希望用项目内隔离配置启动，可以使用：

```bash
CLAUDE_RECOVERY_SKIP_GLOBAL_ENV=1 npm run cli:run
```

## 项目结构

### 根目录

| 路径 | 功能 |
| --- | --- |
| `package.json` | 包元数据、依赖和常用脚本入口。 |
| `package-lock.json` | 依赖锁文件。 |
| `tsconfig.json` | TypeScript 检查配置，本项目主要依赖 esbuild 打包，不直接用 `tsc` 产物。 |
| `scripts/` | 构建、缺失模块审计、恢复启动器和若干 shim。 |
| `src/` | 恢复后的主源码目录。 |
| `dist/` | esbuild 产物目录，包含 `cli.js` 与 `cli.js.map`。 |
| `types/` | 顶层类型补丁与声明文件。 |
| `vendor/` | 第三方或外部搬运代码。 |
| `.claude-recovery/` | 项目级运行配置与会话数据目录。 |
| `.npm-cache/` | 当前工程本地 npm 缓存目录。 |
| `image-processor.node` | 恢复阶段保留的原生模块占位文件。 |

### scripts 模块

| 路径 | 功能 |
| --- | --- |
| `scripts/build.mjs` | esbuild 打包脚本，负责宏替换、缺失模块回退、文本资源 loader、原生模块 TS 替代映射。 |
| `scripts/audit-missing.mjs` | 缺失源码/文本/type-only 模块审计工具。 |
| `scripts/run-recovered-cli.mjs` | 恢复版 CLI 启动器，负责项目级配置目录和全局环境继承/隔离。 |
| `scripts/shims/empty-module.js` | 空模块 shim，用于无内容依赖的兜底。 |
| `scripts/shims/missing-module.cjs` | 缺失运行时代码时的通用降级模块。 |
| `scripts/shims/missing-text.cjs` | 缺失文本资源时的通用降级模块。 |
| `scripts/shims/bun-bundle.js` | `bun:bundle` 的兼容 shim。 |
| `scripts/shims/bun-ffi.js` | `bun:ffi` 的兼容 shim。 |

## 架构分层

### 启动与入口层

| 路径 | 功能 |
| --- | --- |
| `src/entrypoints` | CLI/SDK 入口，负责启动分流和轻量 fast-path。 |
| `src/entrypoints/sdk` | SDK 入口 schema 与 SDK 模式支持。 |
| `src/bootstrap` | 运行时全局状态、会话上下文和初始化阶段共享状态。 |
| `src/main.tsx` | 主 CLI 入口，实现参数解析、初始化、插件/技能/工具装配与 REPL 拉起。 |
| `src/commands.ts` | 内建命令注册表，聚合所有 slash/CLI 命令。 |

### 交互与呈现层

| 路径 | 功能 |
| --- | --- |
| `src/components` | Ink/React 交互组件总目录。 |
| `src/ink` | Ink 渲染适配、自定义布局、事件和终端能力封装。 |
| `src/screens` | 完整屏幕级交互视图。 |
| `src/outputStyles` | 输出风格与展示模式。 |
| `src/vim` | Vim 风格输入和编辑支持。 |
| `src/voice` | 语音交互相关模块。 |

### 能力与业务层

| 路径 | 功能 |
| --- | --- |
| `src/services` | API、OAuth、MCP、LSP、分析、策略、记忆等业务服务。 |
| `src/tools` | 暴露给模型或运行时的工具实现。 |
| `src/tasks` | 本地/远端/工作流/子代理任务执行单元。 |
| `src/plugins` | 插件系统和内置插件装配。 |
| `src/skills` | 技能系统与打包技能资源。 |
| `src/remote` | 远程会话、同步和传输管理。 |
| `src/bridge` | 远程控制/桥接模式支持。 |
| `src/server` | 本地服务端能力和直连会话创建。 |

### 基础设施层

| 路径 | 功能 |
| --- | --- |
| `src/utils` | 工程内最庞大的基础设施层，包含配置、认证、模型、文件、权限、插件、shell、sandbox 等通用能力。 |
| `src/constants` | 常量、产品文案、模型配置、OAuth 配置等静态定义。 |
| `src/context` | 会话上下文、系统上下文、用户上下文聚合。 |
| `src/state` | 状态仓库、应用状态与变更处理。 |
| `src/types` | 共享类型定义。 |
| `src/schemas` | 校验 schema。 |
| `src/native-ts` | 用 TypeScript 重写的原生模块替代实现。 |
| `src/migrations` | 兼容旧配置、旧模型和旧行为的迁移脚本。 |

## 模块清单

说明：

- 本清单按“目录模块”统计，覆盖 `src/` 下一级和二级目录。
- 二级模块以职责为主描述，便于后续恢复与维护。
- 名称明显为实验、内部或特性开关模块的目录，会按名称和当前调用链给出职责说明。

### src 一级模块

| 路径 | 功能 |
| --- | --- |
| `src/assistant` | assistant / brief / kairos 模式相关逻辑。 |
| `src/bootstrap` | 启动期全局状态和进程级上下文。 |
| `src/bridge` | 远程控制桥接、会话接入与桥接 UI。 |
| `src/buddy` | buddy 协作/代理式辅助能力。 |
| `src/cli` | CLI 输出、transport 和 headless 打印模式实现。 |
| `src/commands` | 内建命令目录。 |
| `src/components` | React/Ink 组件目录。 |
| `src/constants` | 常量定义。 |
| `src/context` | 运行上下文模型。 |
| `src/coordinator` | 多代理/协调者模式。 |
| `src/entrypoints` | 程序入口。 |
| `src/hooks` | React hooks 与通知、权限 hooks。 |
| `src/ink` | Ink 基础设施。 |
| `src/jobs` | 后台任务和分类/异步作业。 |
| `src/keybindings` | 快捷键体系。 |
| `src/memdir` | memory directory 与记忆遥测。 |
| `src/migrations` | 配置与行为迁移。 |
| `src/moreright` | 质量/安全相关扩展逻辑。 |
| `src/native-ts` | 原生模块 TS 替代实现。 |
| `src/outputStyles` | 输出样式实现。 |
| `src/plugins` | 插件系统。 |
| `src/proactive` | proactive 预判/主动建议能力。 |
| `src/query` | 查询分析与检索辅助逻辑。 |
| `src/remote` | 远程会话与远程管理。 |
| `src/schemas` | 数据结构 schema。 |
| `src/screens` | 屏幕级 UI。 |
| `src/server` | 本地服务端与直连会话。 |
| `src/services` | 服务层目录。 |
| `src/skills` | 技能系统与内置技能。 |
| `src/state` | 状态容器。 |
| `src/tasks` | 任务执行单元。 |
| `src/tools` | 模型工具集。 |
| `src/types` | 类型系统。 |
| `src/upstreamproxy` | 上游代理中转与 CONNECT/WS 处理。 |
| `src/utils` | 通用基础设施。 |
| `src/vim` | Vim 模式能力。 |
| `src/voice` | 语音能力。 |

### src/commands 模块

| 路径 | 功能 |
| --- | --- |
| `src/commands/add-dir` | 添加额外目录到工具访问白名单。 |
| `src/commands/agents` | 管理和列出 agent 定义。 |
| `src/commands/agents-platform` | 平台级 agent 管理，偏内部用途。 |
| `src/commands/ant-trace` | 内部跟踪/追踪调试命令。 |
| `src/commands/assistant` | assistant 模式命令。 |
| `src/commands/autofix-pr` | 自动修复 PR 问题的命令。 |
| `src/commands/backfill-sessions` | 会话数据回填。 |
| `src/commands/branch` | 分支/工作树相关操作。 |
| `src/commands/break-cache` | 主动破坏缓存、清理缓存用于调试。 |
| `src/commands/bridge` | 远程控制桥接命令。 |
| `src/commands/btw` | 内部/实验命令。 |
| `src/commands/buddy` | buddy 模式命令。 |
| `src/commands/bughunter` | 缺陷排查、内建 bughunter 工作流。 |
| `src/commands/chrome` | Claude in Chrome 相关命令。 |
| `src/commands/clear` | 清空上下文、清理状态。 |
| `src/commands/color` | 颜色/语法高亮相关命令。 |
| `src/commands/compact` | 压缩上下文、触发 compact 流程。 |
| `src/commands/config` | 配置查看、写入和检查。 |
| `src/commands/context` | 上下文展示、调试和非交互导出。 |
| `src/commands/copy` | 复制输出、会话内容或结果。 |
| `src/commands/cost` | 成本分析与费用查看。 |
| `src/commands/ctx_viz` | 上下文可视化。 |
| `src/commands/debug-tool-call` | 调试工具调用。 |
| `src/commands/desktop` | 桌面端相关功能。 |
| `src/commands/diff` | diff 查看与差异分析。 |
| `src/commands/doctor` | 健康检查和环境诊断。 |
| `src/commands/effort` | effort 模式设置。 |
| `src/commands/env` | 环境变量与环境诊断。 |
| `src/commands/exit` | 退出命令。 |
| `src/commands/export` | 导出会话、内容或报告。 |
| `src/commands/extra-usage` | 附加用量/额外额度相关命令。 |
| `src/commands/fast` | fast mode 相关命令。 |
| `src/commands/feedback` | 反馈提交。 |
| `src/commands/files` | 文件资源管理和下载。 |
| `src/commands/fork` | 派生/分叉代理或会话。 |
| `src/commands/good-claude` | 内部/实验命令。 |
| `src/commands/heapdump` | 堆转储诊断。 |
| `src/commands/help` | 帮助系统。 |
| `src/commands/hooks` | hooks 管理与调试。 |
| `src/commands/ide` | IDE 接入与连接。 |
| `src/commands/install-github-app` | GitHub App 安装流程。 |
| `src/commands/install-slack-app` | Slack App 安装流程。 |
| `src/commands/issue` | issue 相关工作流。 |
| `src/commands/keybindings` | 快捷键查看与配置。 |
| `src/commands/login` | 登录命令。 |
| `src/commands/logout` | 登出命令。 |
| `src/commands/mcp` | MCP 服务管理。 |
| `src/commands/memory` | 记忆系统管理。 |
| `src/commands/mobile` | 移动端相关能力。 |
| `src/commands/mock-limits` | mock 限额，偏测试用途。 |
| `src/commands/model` | 模型选择和模型配置。 |
| `src/commands/oauth-refresh` | OAuth 刷新调试命令。 |
| `src/commands/onboarding` | 首次启动引导。 |
| `src/commands/output-style` | 输出风格切换。 |
| `src/commands/passes` | Passes 资格与权益查看。 |
| `src/commands/peers` | peer/收件箱/协作对象命令。 |
| `src/commands/perf-issue` | 性能问题上报与诊断。 |
| `src/commands/permissions` | 权限模式管理。 |
| `src/commands/plan` | plan mode 命令。 |
| `src/commands/plugin` | 插件管理。 |
| `src/commands/pr_comments` | PR 评论处理。 |
| `src/commands/privacy-settings` | 隐私设置。 |
| `src/commands/rate-limit-options` | 限流/配额相关设置。 |
| `src/commands/release-notes` | 版本说明查看。 |
| `src/commands/reload-plugins` | 重载插件。 |
| `src/commands/remote-env` | 远端环境查看。 |
| `src/commands/remote-setup` | 远程初始化和部署。 |
| `src/commands/remoteControlServer` | 远程控制服务端命令。 |
| `src/commands/rename` | 会话或实体重命名。 |
| `src/commands/reset-limits` | 重置限额。 |
| `src/commands/resume` | 恢复历史会话。 |
| `src/commands/review` | 代码评审、ultrareview 等评审流。 |
| `src/commands/rewind` | 回退会话状态。 |
| `src/commands/sandbox-toggle` | 切换 sandbox 相关状态。 |
| `src/commands/session` | 会话级管理。 |
| `src/commands/share` | 分享会话与结果。 |
| `src/commands/skills` | 技能列举、调用和管理。 |
| `src/commands/stats` | 统计信息命令。 |
| `src/commands/status` | 当前状态查看。 |
| `src/commands/stickers` | sticker/徽章类展示能力。 |
| `src/commands/summary` | 生成摘要。 |
| `src/commands/tag` | 打标签。 |
| `src/commands/tasks` | 任务系统命令。 |
| `src/commands/teleport` | Teleport 远端工作流。 |
| `src/commands/terminalSetup` | 终端初始化指引。 |
| `src/commands/theme` | 主题切换。 |
| `src/commands/thinkback` | 历史回放、回溯分析。 |
| `src/commands/thinkback-play` | thinkback 播放/演示。 |
| `src/commands/upgrade` | 升级与更新。 |
| `src/commands/usage` | 用量查看。 |
| `src/commands/vim` | Vim 模式设置。 |
| `src/commands/voice` | 语音模式命令。 |
| `src/commands/workflows` | workflow 脚本与流程编排。 |

### src/components 模块

| 路径 | 功能 |
| --- | --- |
| `src/components/ClaudeCodeHint` | Claude Code 提示信息组件。 |
| `src/components/CustomSelect` | 自定义选择器组件。 |
| `src/components/DesktopUpsell` | 桌面端能力升级引导组件。 |
| `src/components/FeedbackSurvey` | 反馈问卷 UI。 |
| `src/components/HelpV2` | 新版帮助页组件。 |
| `src/components/HighlightedCode` | 代码高亮展示。 |
| `src/components/LogoV2` | Logo 与品牌展示组件。 |
| `src/components/LspRecommendation` | LSP 推荐提示。 |
| `src/components/ManagedSettingsSecurityDialog` | 托管设置安全提示对话框。 |
| `src/components/Passes` | Passes 权益相关组件。 |
| `src/components/PromptInput` | Prompt 输入区域。 |
| `src/components/Settings` | 设置界面。 |
| `src/components/Spinner` | 统一加载动画组件。 |
| `src/components/StructuredDiff` | 结构化 diff 组件。 |
| `src/components/TrustDialog` | 工作区信任对话框。 |
| `src/components/agents` | agent 编辑、选择和管理 UI。 |
| `src/components/design-system` | 设计系统与主题 token。 |
| `src/components/diff` | diff 相关组件。 |
| `src/components/grove` | grove 风格/品牌相关 UI。 |
| `src/components/hooks` | hook 展示和 hook 结果 UI。 |
| `src/components/mcp` | MCP 连接、授权和服务管理 UI。 |
| `src/components/memory` | 记忆查看、选择和编辑 UI。 |
| `src/components/messages` | 会话消息组件。 |
| `src/components/permissions` | 权限解释、授权列表与决策 UI。 |
| `src/components/sandbox` | sandbox 相关提示和面板。 |
| `src/components/shell` | shell 输出和终端消息组件。 |
| `src/components/skills` | 技能列表和技能帮助 UI。 |
| `src/components/tasks` | 任务视图。 |
| `src/components/teams` | 团队/协作对象 UI。 |
| `src/components/ui` | 通用 UI 组件。 |
| `src/components/wizard` | 向导式交互组件。 |

### src/services 模块

| 路径 | 功能 |
| --- | --- |
| `src/services/AgentSummary` | agent 输出摘要与归纳。 |
| `src/services/MagicDocs` | 文档抽取、解析或知识化服务。 |
| `src/services/PromptSuggestion` | prompt 建议与补全。 |
| `src/services/SessionMemory` | 会话级记忆能力。 |
| `src/services/analytics` | 事件、埋点、GrowthBook、遥测。 |
| `src/services/api` | Anthropic API、bootstrap、files、会话请求和网络层。 |
| `src/services/autoDream` | 自动 dream/自动思考相关流程。 |
| `src/services/compact` | 压缩上下文与 reactive compact。 |
| `src/services/contextCollapse` | 上下文折叠与缩减。 |
| `src/services/extractMemories` | 从对话中抽取记忆。 |
| `src/services/lsp` | LSP 连接、诊断与资源管理。 |
| `src/services/mcp` | MCP 客户端、配置、OAuth、XAA 与注册中心。 |
| `src/services/oauth` | OAuth 登录、刷新、profile 和 token 生命周期。 |
| `src/services/plugins` | 插件安装、查询和 CLI 命令扩展。 |
| `src/services/policyLimits` | 组织策略和功能限额控制。 |
| `src/services/remoteManagedSettings` | 远端托管设置同步。 |
| `src/services/sessionTranscript` | 会话转录与记录。 |
| `src/services/settingsSync` | 设置同步。 |
| `src/services/skillSearch` | 技能索引、预取、本地/远端技能搜索。 |
| `src/services/teamMemorySync` | 团队记忆同步。 |
| `src/services/tips` | 提示词、tips、上下文提示注册表。 |
| `src/services/toolUseSummary` | 工具使用摘要。 |
| `src/services/tools` | 工具运行时的服务端辅助层。 |

### src/tools 模块

| 路径 | 功能 |
| --- | --- |
| `src/tools/AgentTool` | 创建/调度 agent。 |
| `src/tools/AskUserQuestionTool` | 向用户提问。 |
| `src/tools/BashTool` | 执行 Bash 命令。 |
| `src/tools/BriefTool` | 生成 brief/摘要型交付物。 |
| `src/tools/ConfigTool` | 读取与修改配置。 |
| `src/tools/DiscoverSkillsTool` | 发现可用技能。 |
| `src/tools/EnterPlanModeTool` | 进入 plan mode。 |
| `src/tools/EnterWorktreeTool` | 进入 worktree 模式。 |
| `src/tools/ExitPlanModeTool` | 退出 plan mode。 |
| `src/tools/ExitWorktreeTool` | 退出 worktree 模式。 |
| `src/tools/FileEditTool` | 局部编辑文件。 |
| `src/tools/FileReadTool` | 读取文件。 |
| `src/tools/FileWriteTool` | 写文件。 |
| `src/tools/GlobTool` | 文件通配检索。 |
| `src/tools/GrepTool` | 文本搜索。 |
| `src/tools/LSPTool` | 调用 LSP 能力。 |
| `src/tools/ListMcpResourcesTool` | 列出 MCP 资源。 |
| `src/tools/MCPTool` | 通用 MCP 工具桥。 |
| `src/tools/McpAuthTool` | MCP OAuth/授权流程。 |
| `src/tools/MonitorTool` | 监控型工具。 |
| `src/tools/NotebookEditTool` | notebook 编辑。 |
| `src/tools/OverflowTestTool` | 溢出/边界测试工具。 |
| `src/tools/PowerShellTool` | PowerShell 命令执行。 |
| `src/tools/REPLTool` | REPL 控制。 |
| `src/tools/ReadMcpResourceTool` | 读取单个 MCP 资源。 |
| `src/tools/RemoteTriggerTool` | 触发远程动作。 |
| `src/tools/ReviewArtifactTool` | 评审产物。 |
| `src/tools/ScheduleCronTool` | 定时任务安排。 |
| `src/tools/SendMessageTool` | 发送消息给 agent/peer。 |
| `src/tools/SendUserFileTool` | 向用户传递文件。 |
| `src/tools/SkillTool` | 调用技能。 |
| `src/tools/SleepTool` | 等待/休眠。 |
| `src/tools/SnipTool` | 截断/裁剪上下文。 |
| `src/tools/SyntheticOutputTool` | 合成输出或模拟输出工具。 |
| `src/tools/TaskCreateTool` | 创建任务。 |
| `src/tools/TaskGetTool` | 查询任务。 |
| `src/tools/TaskListTool` | 列出任务。 |
| `src/tools/TaskOutputTool` | 输出任务结果。 |
| `src/tools/TaskStopTool` | 停止任务。 |
| `src/tools/TaskUpdateTool` | 更新任务。 |
| `src/tools/TeamCreateTool` | 创建团队/协作单元。 |
| `src/tools/TeamDeleteTool` | 删除团队/协作单元。 |
| `src/tools/TerminalCaptureTool` | 采集终端输出。 |
| `src/tools/TodoWriteTool` | 写入 todo 列表。 |
| `src/tools/ToolSearchTool` | 搜索工具。 |
| `src/tools/TungstenTool` | tungsten 相关占位/内部工具。 |
| `src/tools/VerifyPlanExecutionTool` | 校验 plan 执行状态。 |
| `src/tools/WebBrowserTool` | 浏览器交互。 |
| `src/tools/WebFetchTool` | 抓取网页。 |
| `src/tools/WebSearchTool` | 网页搜索。 |
| `src/tools/WorkflowTool` | 工作流编排工具。 |
| `src/tools/shared` | 工具共享逻辑。 |
| `src/tools/testing` | 工具测试支撑。 |

### src/tasks 模块

| 路径 | 功能 |
| --- | --- |
| `src/tasks/DreamTask` | dream 型后台任务。 |
| `src/tasks/InProcessTeammateTask` | 进程内 teammate 任务。 |
| `src/tasks/LocalAgentTask` | 本地 agent 任务。 |
| `src/tasks/LocalShellTask` | 本地 shell 任务。 |
| `src/tasks/LocalWorkflowTask` | 本地 workflow 任务。 |
| `src/tasks/MonitorMcpTask` | MCP 监控任务。 |
| `src/tasks/RemoteAgentTask` | 远程 agent 任务。 |

### src/utils 子模块

| 路径 | 功能 |
| --- | --- |
| `src/utils/background` | 后台任务与后台 housekeeping。 |
| `src/utils/bash` | Bash 适配与命令前缀、解析。 |
| `src/utils/claudeInChrome` | Claude in Chrome 启动、MCP、本地宿主。 |
| `src/utils/computerUse` | computer use 模式支持。 |
| `src/utils/deepLink` | 深链路入口与横幅。 |
| `src/utils/dxt` | dxt 相关解析和支持逻辑。 |
| `src/utils/filePersistence` | 文件持久化抽象。 |
| `src/utils/git` | Git 相关辅助逻辑。 |
| `src/utils/github` | GitHub 状态、路径映射等能力。 |
| `src/utils/hooks` | hook 执行和 hook 事件工具。 |
| `src/utils/mcp` | MCP 运行时工具。 |
| `src/utils/memory` | 记忆系统工具。 |
| `src/utils/messages` | 消息处理辅助。 |
| `src/utils/model` | 模型配置、能力、能力判断、字符串映射。 |
| `src/utils/nativeInstaller` | 原生安装包下载和安装逻辑。 |
| `src/utils/permissions` | 权限模式、自动模式、权限解释。 |
| `src/utils/plugins` | 插件缓存、装载、目录与清理。 |
| `src/utils/powershell` | PowerShell 适配。 |
| `src/utils/processUserInput` | 用户输入预处理。 |
| `src/utils/sandbox` | sandbox 抽象层。 |
| `src/utils/secureStorage` | keychain/安全存储。 |
| `src/utils/settings` | 设置读取、合并、校验、MDM。 |
| `src/utils/shell` | shell 命令和只读校验。 |
| `src/utils/skills` | 技能检测与技能资源辅助。 |
| `src/utils/suggestions` | 建议系统辅助模块。 |
| `src/utils/swarm` | swarm/teammate/多代理协作。 |
| `src/utils/task` | 任务系统通用工具。 |
| `src/utils/telemetry` | 遥测导出与辅助工具。 |
| `src/utils/teleport` | teleport 网络与会话操作。 |
| `src/utils/todo` | todo 模型与工具。 |
| `src/utils/ultraplan` | ultraplan prompt 和辅助逻辑。 |

### 其他二级模块

| 路径 | 功能 |
| --- | --- |
| `src/cli/handlers` | 各 CLI 子命令的 headless 处理器。 |
| `src/cli/transports` | SSE、WebSocket、Hybrid 等传输实现。 |
| `src/hooks/notifs` | 通知类 hooks。 |
| `src/hooks/toolPermission` | 工具权限相关 hooks。 |
| `src/ink/components` | Ink 基础组件。 |
| `src/ink/events` | 终端事件封装。 |
| `src/ink/hooks` | Ink hooks。 |
| `src/ink/layout` | 终端布局。 |
| `src/ink/termio` | DEC/TTY 级终端 IO 细节。 |
| `src/native-ts/color-diff` | `color-diff-napi` 的 TypeScript 替代实现。 |
| `src/native-ts/file-index` | 文件索引的 TS 替代实现。 |
| `src/native-ts/yoga-layout` | Yoga 布局层的 TS 替代或兼容实现。 |
| `src/plugins/bundled` | 内置插件集合。 |
| `src/skills/bundled` | 内置技能集合。 |
| `src/types/generated` | 生成的类型定义。 |

## 恢复工程的实现特点

### 1. 构建不是官方原始构建链，而是恢复构建链

当前工程使用 [scripts/build.mjs](/Users/test/Downloads/claude-code/package/claude-code-2.1.88/scripts/build.mjs) 通过 esbuild 重新打包，核心目标是：

- 让恢复源码可以重新产出 `dist/cli.js`
- 对缺失文本资源和缺失模块提供统一降级
- 对部分 Bun/原生模块提供 shim 或 TS 替代实现

### 2. 运行器支持“继承全局环境”和“完全隔离”

[scripts/run-recovered-cli.mjs](/Users/test/Downloads/claude-code/package/claude-code-2.1.88/scripts/run-recovered-cli.mjs) 默认会：

- 把会话数据写到项目内 `.claude-recovery/`
- 继承 `~/.claude/settings.json` 中的 `env` 配置

当设置 `CLAUDE_RECOVERY_SKIP_GLOBAL_ENV=1` 时，运行器会跳过全局 `env` 注入，方便测试完全隔离环境。

### 3. 原生模块已有部分 TS 回退实现

当前恢复工程已经接入的代表性回退有：

- `color-diff-napi` -> `src/native-ts/color-diff`
- 缺失文本资源 -> `scripts/shims/missing-text.cjs`
- 缺失代码模块 -> `scripts/shims/missing-module.cjs`

## 常用命令

```bash
npm_config_cache=.npm-cache npm run build
npm run audit:missing
npm run cli:status
npm run cli:run
npm run cli:run -- -p "Reply with exactly: OK"
CLAUDE_RECOVERY_SKIP_GLOBAL_ENV=1 npm run cli:run
```

## 文档索引

- [docs/BUILD_MANUAL.md](/Users/test/Downloads/claude-code/package/claude-code-2.1.88/docs/BUILD_MANUAL.md)
- [docs/SECONDARY_DEVELOPMENT_MANUAL.md](/Users/test/Downloads/claude-code/package/claude-code-2.1.88/docs/SECONDARY_DEVELOPMENT_MANUAL.md)
