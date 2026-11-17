# MonitorBar：显示器音量、输入源与画面控制

**主开发目录**：`/home/niermhy/workspace/project/project_myself/github_project/MonitorBar`

**项目文档根**：`/home/niermhy/workspace/Obsidian/Obsidian_Note_Server/工作知识记录/个人开发项目/MonitorBar/`

**独立 GitHub 仓库**：`https://github.com/NieRMHY/MonitorBar.git`

**接力开发及 GitHub 默认分支**：`dev/minipc`

<!-- Add by MHY, 2026-11-17 配置 miniPC 主开发工作区与多端同步规范 -->

## 项目概述

MonitorBar 使用 Go + mygo 提供 macOS 菜单栏与 Windows 托盘界面，通过澎湃 OS 局域网接口、DDC/CI 和 Gamma 调整显示器。当前本地开发分支以 Windows 已安装的 v1.11.1 标签为基线；没有把上游 main 的后续变化混入基线。

## 工作方式与规则适用

- 个人仓库只配置 `origin` 指向 `NieRMHY/MonitorBar`，不配置上游 remote 或自动合并；原项目来源见 README，保留许可证和版权信息。
- miniPC（Linux）是主开发机；此仓库保留完整 Git 历史和远程关联。Windows/macOS 原生功能在相应平台构建与验收。
- 开始任务先读本文件及适用的 `AGENTS.md`，按任务查阅模块文档；不要求每次通读所有协议材料。
- 源码沿用上游命名、目录和语言习惯；公共模板的通用命名规则不用于批量改名。修改标注使用所属语言合法的注释形式。
- 保留上游 README、许可证和技术文档；本机介绍、方案及工作记录放上述 vault 文档根。本次用户授权建立这些介绍和规则。
- 已授权范围内直接完成编辑、验证及所需 Git 操作；提交、推送和发布以当次用户授权为准，已有授权无需重复确认。2026-11-17 已授权将三个项目独立维护并同步这些规则。
- 多端同步以 Git 提交为单位：先检查分支和工作树，再 `git fetch origin`；有对应追踪分支且工作树干净时使用 `git pull --ff-only`。发现分叉或未提交修改时保留现场，禁止覆盖或强推。
- Windows 源码副本：`E:\Workspace\codex\MonitorBar`（本次新建）；原有安装包位于 `D:\Tools\MonitorBar_v1.11.1_Portable`。通过 Git 在两端交换后续代码；不要反复复制整个工作区或覆盖 `.git`。
- 网络沿用环境文档的会话代理。2026-11-17 初始化时既有代理不可用，临时经 Windows SSH 隧道取回 GitHub 数据；临时端口没有写入 Git 配置。后续先核对网络，不写入失效的临时地址。

## 来源与同步边界

- `origin` 指向独立个人仓库 `NieRMHY/MonitorBar`；原作者信息仅作为 README 的来源说明，不保留上游 remote。
- `dev/minipc` 起点为 `v1.11.1`，保留完整原始提交历史。原始 `main` 分支和标签属于历史基线，不自动合入当前开发分支。
- 日常在当前分支检查工作树后 fetch / fast-forward；不得把历史 `main` 当作个人开发分支自动覆盖当前功能。

## 环境与常用命令

- `go.mod` 为 Go 1.27.2、mygo v0.5.5；依赖版本以当前 `go.mod/go.sum` 为准。
- 当前项目的原生平台层是 Windows/macOS，CI 在 macOS 执行；没有已验证的 Linux GUI 后端。miniPC 可编辑代码，工具链可用后再验证平台无关逻辑或交叉构建。
- 本次仅迁入源码与规则；没有安装 Go、执行构建、运行显示器探测或更改真实显示器设置。

以下为上游已有命令，原生开发及完整测试在受支持平台执行：

```sh
go tool mygo dev
go test ./...
go vet ./...
go tool mygo generate
go tool mygo build -platform darwin/universal,windows/amd64
```

`generate` 用于变更 `locales/` 后重建语言资源；构建输出为 `build/`。Linux 安装符合 `go.mod` 的工具链后，可先尝试不依赖 GUI 的 `go test ./internal/schedule ./internal/usage ./internal/mitv`，本次未执行；不能据此宣称完整应用可在 Linux 运行。

## 架构与核心规则

- `main.go/app.go`：入口与托盘；`store.go`：状态和动作；`panel.go/tabs.go/widgets.go`：界面。
- `internal/mitv/`：澎湃 OS 局域网接口与发现；`internal/display/`：Windows/macOS 显示器、Gamma、DDC/CI、分辨率与音频。
- `internal/usage/`：使用时长；`internal/schedule/`：自动场景；`locales/`：翻译；`mygo.json`：应用及更新配置。
- 应用状态只在主线程读写；网络操作在 goroutine 中，结果经既有 `post` 返回 UI 线程。
- 输入源和音量优先 LAN，再用 DDC/CI；亮度有硬件能力时调背光，否则软件调光。不可把软件 Gamma 说成硬件背光。
- 新显示器先收集该设备的 `--probe` 结果；型号、接口和输入源编号按实际响应适配，不复制另一型号的假设。
- 保留退出时恢复 Gamma、多显示器身份和各平台差异。Linux 编辑不会证明 Windows 托盘或 macOS 菜单栏正常。
- `mygo.json` 仍指向上游更新源和公钥；个人版本发布前应明确独立更新身份、签名和版本策略，避免个人版本被上游更新覆盖。
- 真实输入源切换、Gamma、音量及分辨率修改属于设备操作；按具体任务授权范围验证，不因编译成功自动运行 `--probe` 或控制显示器。
- `%APPDATA%\MonitorBar` 为 Windows 运行数据目录；历史显示器地址只作记录，不硬编码到公开源码。

## 初始化验证与交付状态

- 源码基线：`4aaa7691b14ba79b8573340fa46160e38ac01c42`（2026-11-17）。具体文件比对与环境限制见 vault `工作记录/2026-11-17_迁入miniPC与源码核验.md`。
- 本次将 README 来源说明和开发规则提交到独立个人仓库；未执行原生程序构建及硬件测试，不能把历史测试结果当作此次验证。

## 公共规则

以下按现有项目约定复制自 vault 根目录 `_project_CLAUDE_template.md`；已确定的文档根与平台边界以上方为准。

## 文档规则
- 项目过程中按需生成记录、说明文档，统一存放在 Obsidian vault
- **Vault 根路径**: `/home/niermhy/workspace/Obsidian/Obsidian_Note_Server/`
- 开始时先检索 vault 目录结构，找到与项目领域匹配的分类目录；无合适分类则在 vault 根目录新建
- 在该分类下创建本项目子目录作为文档根
- 确定文档路径后，将完整路径写入本文档顶部，后续直接使用无需重复检索
- 生成/更新/读取文档 → 均从 vault 对应路径操作
- 写入与生成文档时遵循 vault 根目录 `CLAUDE.md` 的整理规则
- 使用 `[[内部链接]]` 在文档之间交叉引用

### 工作记录
- 中间过程记录、阶段总结、plan 方案均写入本项目文档目录下的 `工作记录/` 子目录
- 记录文档命名：`YYYY-MM-DD_简短描述.md`
- Plan 文档命名：`_Plan_描述.md`
- 已完成的 plan 在正文开头标注完成日期

## 工作区环境汇总
- 阅读 `/home/niermhy/workspace/environment/README.md`（含服务器连接、SDK 共享、烧写等索引）
- environment 文档变更后运行 `env-sync` 同步 vault 镜像（`信息记录/environment镜像/`）

## 语言
- 沟通使用简体中文，代码标识符使用英文
- 日志信息：优先使用中文，或英文

## 网络环境
- 当前环境：国内网络，GitHub 直连不稳定
- 需要访问外部网络时先 `proxy-on`（`~/scripts/ConfigProxy.sh`，结束后可 `proxy-off`）

## 命名规范
- 函数/类名：PascalCase（大驼峰）
- 变量名：camelCase（小驼峰）
- 文件名：camelCase（小驼峰），少用连字符或下划线

## 代码风格
- 不写多余注释，只写"为什么"而非"是什么"
- 修改代码时添加标注：`# Add by MHY, <说明>` 或 `# Modify by MHY, <说明>`
- 不引入不必要的抽象，三行相似代码好过一次过早的封装
- 只处理真实发生的场景，不为"可能"做防御性编程
- 不写 docstring、不写 README（项目记录/Plan 写入 vault 工作记录目录，不在此列）

## 工具使用
- 优先使用 Read/Edit/Write 工具而非 cat/sed/awk
- 独立操作并行执行（多个 Bash 调用或 Agent 调用同时发出）
- git 操作需确认，不做 force push / reset --hard 等破坏性操作
- 提交时作者使用当前 git config，不添加 Co-Authored-By（除非明确要求）
