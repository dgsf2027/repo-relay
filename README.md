# Repo Relay · 项目接力

**换台电脑，发一个仓库地址，继续开发。改完说“上传”，Agent 就按团队约定处理。**

给经常换设备、直接推代码、不走 PR 的小团队使用。包含可安装的 `repo-relay` skill 和这份从零开始的中文指南，适用于 Codex；Claude Code 也可安装同一技能目录。

- 普通项目：同步最新 main，处理冲突、检查，通过后推 main。
- **xjxl0104/zhyq**：先推 test，Agent 检查并合并 main，服务器轮询部署。
- 直接数据库修改：提醒向 **xjxl** 索要对应服务器密钥／授权，完成必要核验后发布。

这是给 Agent 执行的工作流。安装本 skill 不会自动克隆、安装项目环境或发布代码；需要按下面的口令告诉它要做什么。

## 0. 新电脑先准备什么

1. 安装并登录你要使用的 Codex 或 Claude Code。以产品当前支持的平台和官方安装说明为准。
2. 准备你自己的 GitHub 账号，以及**需要修改并上传的仓库地址**。仓库管理员应给该账号 Write（写入）或更高权限；邀请要先接受。
3. 在 Agent **当前任务**的权限设置中选择“完全访问 / Full access”，让它能安装依赖、写入开发目录和访问 GitHub。CLI 可通过 `/permissions` 查看当前支持的模式。界面不同就让 Agent 根据当前版本指路。

系统弹出管理员密码、浏览器授权或设备登录时，由你本人完成。其他步骤交给 Agent。完全访问不会赋予 GitHub 仓库权限，也不会自动取得服务器密钥。

macOS 的“完全磁盘访问权限”是另一回事；只有实际遇到系统文件访问拦截时才按提示开启，不用一开始就修改所有系统权限。

参考：[Codex／桌面端快速开始](https://learn.chatgpt.com/docs/quickstart)、[Codex 权限](https://learn.chatgpt.com/docs/agent-approvals-security)、[Claude Code 安装](https://code.claude.com/docs/en/setup)。

## 1. 安装技能：复制这段给 Codex

```text
请使用 $skill-installer 安装这个 GitHub 技能：
https://github.com/dgsf2027/repo-relay/tree/main/skills/repo-relay
检查当前 Codex 支持的用户级 skill 目录，只安装一份。
安装后确认可以发现 repo-relay，并告诉我实际安装路径。
```

安装后在下一轮调用 `$repo-relay`；若列表没有刷新，重新打开任务或重启 Codex。安装器可能使用 `~/.codex/skills`；当前公开文档的用户目录为 `~/.agents/skills`。以目标设备的安装器与实际发现结果为准，不向两处重复安装。[Codex Skills](https://learn.chatgpt.com/docs/build-skills)

### 没有 skill-installer，或者用 Claude Code

把下面这段交给当前 Agent，也能安装：

```text
请从 https://github.com/dgsf2027/repo-relay 获取最新代码，
把 skills/repo-relay 整个目录安装到本机当前工具支持的用户级 skill 目录。
Codex 按当前版本实际支持的目录；Claude Code 使用 ~/.claude/skills/repo-relay。
没有 Git 时先按系统官方方式安装 Git，或者下载 GitHub ZIP 解压。
只复制技能目录，不复制 .git，不安装任何业务项目。
若已存在同名技能，先备份旧目录，再完整更新并核对引用文件。
最后确认技能能被发现，告诉我调用方式。
```

手动安装同样可以：从 GitHub 的 **Code → Download ZIP** 下载并解压，将 `skills/repo-relay` 文件夹整体复制到上述用户级目录。最终结构应为 `.../skills/repo-relay/SKILL.md`，不要多套一层 `repo-relay/skills/repo-relay`。

Windows 的 `~` 对应当前用户主目录；在 PowerShell 中可用 `$env:USERPROFILE` 定位。不要复制他人机器上的绝对路径。Claude Code 调用 `/repo-relay`，Codex 调用 `$repo-relay`。[Claude Skills](https://code.claude.com/docs/en/skills)

## 2. 第一次接手：发仓库地址

替换成你实际要改的仓库即可：

```text
使用 $repo-relay。
这是我要修改并上传的仓库：https://github.com/xjxl0104/zhyq
请在这台新电脑从零接手：检查权限，缺 Git/gh 就安装；
新开浏览器让我登录 GitHub 并输入这次生成的 8 位设备验证码，
然后核对登录账号对这个仓库是否有写权限。
拉取最新开发代码，按项目声明安装缺少的环境，配置独立本地环境并启动。
没开启完全访问时请指示我开。完成后给我本地网页地址和启动/停止方式。
这一步先启动，等我说“上传”再推代码。
```

还可以加一句：“放到 D:\Projects\zhyq”或“放到我的 Documents/Projects 下”。未指定目录时，Agent 会选一个普通开发目录并告知。

### GitHub 授权时你要做的事

1. Agent 启动 GitHub CLI 登录，打开新的授权页面。
2. 用**你自己的账号**登录，在官方页面输入本次显示的 `XXXX-XXXX` 设备代码：共 8 个字母或数字，中间有连字符。
3. 核对授权对象是本次 GitHub CLI 登录，然后确认。密码、2FA 或 passkey 在 GitHub 页面处理。
4. Agent 等终端明确成功，再调用 GitHub API 核对账号和仓库写权限。

如果这台设备已有正确且有效的授权，可直接复用。登录错误账号时重新选择自己的账号；不要用修改 Git 提交姓名来代替登录。

**能打开仓库不等于能上传。** 检查结果至少应显示：当前登录名、完整仓库名、写权限 true/false。没有权限时，让仓库管理员邀请检查结果里的那个账号，接受邀请后再次核验。[GitHub 授权说明](https://cli.github.com/manual/gh_auth_login)

### Agent 会自动补齐什么

按项目需要安装 Git、gh、Node、npm/pnpm、Java、Maven、Docker 等；已有兼容版本会保留。只做前端项目，不会为它额外装 Java。

zhyq 通常需要 Java 17、Maven、兼容的 Node/pnpm，以及本地 MySQL 8；可选择热更新开发或全 Docker。Agent 会读取当前配置，准备本地数据库密码、JWT 和需要的字段加密密钥，验证它们被进程实际读取。不是只装工具就宣告部署完成。

本机新环境使用本地测试数据。GitHub clone 不会带回生产数据库、附件或另一台电脑未提交的文件。

## 3. 日常修改与换机

正常告诉 Agent 要改什么，例如：

```text
使用 $repo-relay，先同步同事最新代码，再修改首页筛选功能。
完成后启动本地预览，我看完再上传。
```

离开旧设备前，先上传已完成的工作；未完成的改动若需单独同步，应明确交给 Agent 保存到合适的工作分支，不混进生产。让它更新交接记录，写明进度、验证命令和待办。

在另一台已配置的设备上说：

```text
使用 $repo-relay，继续接手 https://github.com/xjxl0104/zhyq。
先核对本地未提交/未推内容，再拉取最新代码和交接记录，启动开发环境。
```

本地目录已有改动时，Agent 应先保留，不覆盖、不强制重置；多人同时上传时先合并最新代码，解决冲突后重新检查。

## 4. 修改完成：上传

### 普通项目

```text
使用 $repo-relay，把这个仓库本次完成的修改上传。
先整合远端最新 main，检查冲突、测试和最终差异；
通过后直接推 main，不用 PR。
```

Agent 可以先建本地分支做整合，通过后正常推 main。已经给出上传指令，普通步骤无需再问“确定要推吗”。没有 main 或仓库规则实际要求其他路线时，它会说明具体情况，不擅自关闭分支保护。

普通项目是否自动部署取决于该项目已有配置；推 main 不保证每个项目都有服务器轮询。

### zhyq

```text
使用 $repo-relay，上传 zhyq 本次修改并按项目约定上线。
先推 test，检查全部待上线改动及最终合并结果；
通过后合并并推 main，不用 PR、不用重复确认普通上传。
涉及直接数据库修改时，先提醒我向 xjxl 索要服务器密钥并完成必要核验。
最后报告提交号、检查结果和实际部署状态。
```

当前核对的流程是：

```text
本地修改 → 同步最新 test → 检查并推 test
        → Agent 审查 main 将收到的全部改动
        → 合并后端 test-compile / 前端 build / 相关测试通过
        → 正常推 main → 服务器轮询构建 → 验证运行版本
```

**2026-09-15 核对：仓库没有 GitHub Actions 工作流。** 项目文档要求 Agent 自查并合并 main；服务器负责后续轮询部署。因此，不能只推 test 就结束，也不能等待一个并不存在的 Actions 自动合并任务。

最低构建检查是后端 `mvn -B test-compile`、前端 `pnpm build`，并运行与改动有关的测试。只有 compile 不够；纯文档可以免应用构建，但同批还有他人代码改动时不能免。

项目文档记载服务器每 5 分钟检查 main，整体约 10 分钟；实际仍要看本次版本和部署证据。本技能发布时没有登录服务器复验实时状态。详见 [zhyq 专用说明](skills/repo-relay/references/zhyq.md) 和 [项目分支规则](https://github.com/xjxl0104/zhyq/blob/test/CLAUDE.md)。

## 5. 数据库改动怎么处理

迁移脚本、直接执行 SQL、数据清理／修复／导入、会在上线时写库的启动任务，都可能涉及直接数据库修改。Agent 会提前说：

> 这次包含直接数据库修改，请向 xjxl 索要该项目的服务器访问密钥／授权。

由 xjxl 通过你们约定的私密渠道提供；保存在本机受保护的位置，只告诉 Agent 文件路径。不要把私钥、数据库密码或生产 `.env` 放进聊天或 GitHub。

Agent 先核对生产迁移历史、目标库和影响范围。修改／删除已有数据时，先给出可审阅的方案并完成备份，按适用授权执行。只有密钥不代表可以随意删除数据。

必要条件不齐时保留成果，暂停会触发改库的发布。确认 test 不会自动上线后可以只推 test；普通纯代码上传不需要大家都拿到服务器密钥。

## 6. 完成后看这几项

| 项目 | 你应收到的结果 |
|---|---|
| GitHub | 使用的账号、目标仓库、写权限核验 |
| 本地 | 目录、分支、依赖版本、访问地址和启动／停止方式 |
| 修改 | 本次改动摘要、测试／构建结果及未完成项 |
| 上传 | test/main 的实际提交号和 GitHub 链接 |
| 线上 | 已验证上线、部署中、失败，或代码已推但线上待验证 |

“GitHub 已上传”“构建成功”“生产已运行新版本”是三种结果。没有本次运行版本证据时，Agent 应明确说“代码已推，线上待验证”。

## 常见情况

| 情况 | 处理 |
|---|---|
| 登录成功但无写权限 | 确认账号、接受仓库邀请，必要时完成组织 SSO，再查权限 |
| 缺依赖／管理员密码 | Agent 安装缺项；你只完成系统要求的授权 |
| 已有未提交改动 | 保留原目录，先厘清归属，必要时另开干净工作目录 |
| non-fast-forward | 同事先推了；重新拉取、整合、检查，不强推 |
| 分支保护拒绝上传 | 报告具体规则，由仓库管理员处理；不擅改规则 |
| 推 test 后没上线 | 继续检查和 main 合并步骤，核对是否已完成 |
| main 已推，超过预期没变化 | 查本次部署日志、运行版本和回滚状态；没有访问权则说明待验证 |
| 想更新技能 | 让 Agent 从本仓库获取最新版，备份并替换原技能目录，不重复安装 |

## 给维护者

- [技能入口](skills/repo-relay/SKILL.md)
- [GitHub 授权](skills/repo-relay/references/github-auth.md)
- [环境与本地启动](skills/repo-relay/references/environment.md)
- [同步、上传与部署验证](skills/repo-relay/references/publishing.md)
- [zhyq 项目参考](skills/repo-relay/references/zhyq.md)

本仓库按团队提出的四步需求编写。zhyq 仓库与提供的 agent.md 仅用于核对，未复制个人环境清单、账号凭据或整份附件。适用规则会随项目变化，执行时应读远端最新版本。
