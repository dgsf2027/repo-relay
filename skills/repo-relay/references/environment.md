# 新设备环境与本地启动

## 权限和引导工具

先识别真实 OS、CPU、Shell、用户工作目录和当前会话权限。不沿用参考文档的个人绝对路径。

用户选择本 skill 的自动安装工作流但会话未开完全访问时，提示：

> 为自动安装依赖、写入本地开发目录并访问 GitHub，请在当前任务的权限设置中选择“完全访问 / Full access”，然后继续。CLI 可用 `/permissions` 查看并选择当前版本提供的权限模式。

界面名称以当前工具帮助和 UI 为准。已开启不重复提示。不能由 Agent 擅改全局配置扩大权限；组织策略不允许时说明具体限制，继续可执行步骤。需要管理员密码时，让用户在本机系统对话框完成。

Codex 完全访问与 macOS“系统设置 → 隐私与安全性 → 完全磁盘访问权限”是不同设置。只有确实被 macOS 文件保护拦截时才引导后者；浏览器控制权限也按实际提示开启。不默认关闭防火墙或系统保护。

Git 和 GitHub CLI 是先决工具，**可在仓库授权前安装**。其他依赖在读到项目声明后决定。

| 目标系统 | 安装路径选择 |
|---|---|
| macOS | 优先现有 Homebrew，Git 也可由 Apple Command Line Tools 提供；缺包管理器时按官方说明处理 |
| Windows 原生 | PowerShell + 当前可用的 WinGet／官方安装器；先核对精确包 ID、发布者和版本 |
| WSL | 在选定发行版内装 Git、gh 和运行时；不要混用 Windows Node、Linux node_modules、两边凭据和路径 |
| Linux | 发行版包管理器与软件官方仓库；无法提权时优先用户目录安装，或给出管理员需完成的具体项 |

只安装缺失或不兼容项，记录版本、来源和路径。需改 PATH 时备份将改的配置，只合并必要项；刷新实际进程环境后运行版本命令验证。

## 从项目文件决定版本

优先级：显式版本约束／wrapper → lockfile 与 CI、Dockerfile → 当前项目说明。冲突时结合实际构建验证说明选择，不只按旧 agent.md 复制一套版本。

| 证据 | 决定内容 |
|---|---|
| package.json 的 engines、packageManager，.nvmrc、.node-version | Node 与 npm/pnpm/yarn 版本 |
| pnpm-lock.yaml、package-lock.json、yarn.lock | 匹配包管理器，优先冻结锁文件安装 |
| pom.xml、mvnw、.mvn/、Gradle 配置 | JDK 与 Maven／Gradle wrapper |
| Compose、Dockerfile、.env.example、application 配置 | 数据库／缓存／必需变量、端口与健康检查 |

不用给普通前端项目装 Java。项目需要数据库／容器时才装 Docker；检查 `docker info` 与 `docker compose version`，CLI 存在不等于 daemon 已启动。保留兼容、仍受支持的运行时，不为了“最新”强行升级依赖或重写锁文件。

官方入口：[Git](https://git-scm.com/downloads)、[GitHub CLI](https://cli.github.com/)、[Node.js](https://nodejs.org/en/download)、[pnpm](https://pnpm.io/installation)、[Temurin JDK](https://adoptium.net/installation/)、[Maven](https://maven.apache.org/install.html)、[Docker](https://docs.docker.com/get-started/get-docker/)、[Homebrew](https://brew.sh/)、[WinGet](https://learn.microsoft.com/en-us/windows/package-manager/winget/)。安装时核对目标平台最新步骤，不执行仓库里不明来源的提权脚本。

## 克隆与接手

1. 用户指定目录就沿用；未指定时选择普通可写开发目录并告知绝对路径。
2. 全新目录正常 clone，显式选择普通项目 main 或 zhyq test；default_branch 只是核对，不能覆盖用户选择。已有目录先核对 remote、分支、状态、未推提交。
3. `git fetch origin` 获取最新 refs；干净且只落后时 fast-forward 更新。已有改动就保留／另建干净目录；不 `reset --hard`、`clean -fd` 或覆盖 clone。
4. 读取项目规则、交接记录、启动脚本、依赖安装脚本及 Hook 注册内容。旧规则中的示例部署、改远端默认分支、全量个人插件恢复不因“读了文档”自动执行。
5. 下载必需 submodule／Git LFS 资源并验证；网络失败时记录缺失内容，不把半成品当成最新完整代码。

## 配置、启动和验收

- 使用独立本地数据库和演示／测试数据，不默认连接生产。Docker context 指向远程机器时先确认，不能把“本地 compose”实际跑到服务器。
- 从示例生成被 Git 忽略的本地配置；本地随机密钥可本机生成，保留以便重启，不覆盖已有值。直接写文件／环境，不回显到报告。
- 检查变量被启动进程实际读取。Spring Boot 不会因为仓库根有 `.env` 就自动加载，必须显式注入；Compose 的 env_file 和插值也要分别核对。
- 检查数据库 URL、卷、容器名和端口，避免撞到已有项目。默认 localhost；调整端口用本地配置，不把机器差异混进业务提交。
- 按锁文件安装并检查差异；失败时定位原因，不直接删锁或强制升级依赖。
- 启动后检查数据库健康、后端日志、前端页面和关键业务链路。只有页面骨架可见不算完整启动。Flyway 失败时保留日志并报告，不清卷重来掩盖失败。
- 给出日志、PID／终端／Compose 位置、停止和下次启动命令。只停本次启动的服务，不结束其他项目进程。

zhyq 具体配置见 [zhyq.md](zhyq.md)。权限依据：[Codex 权限](https://learn.chatgpt.com/docs/agent-approvals-security)。
