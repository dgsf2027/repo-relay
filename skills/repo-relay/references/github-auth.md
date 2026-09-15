# GitHub 授权与写权限

## 新机浏览器登录

1. 确认 `gh --version` 和 `git --version` 能运行；缺失时按目标 OS 的官方渠道安装。浏览器能登录 GitHub 不代表本机 Git 已获授权。
2. 用 `gh auth status --hostname github.com` 检查已有登录，不使用 `--show-token`。检查 `GH_TOKEN` / `GITHUB_TOKEN` 是否**存在**，不输出值。它们可能覆盖保存的身份；若与用户要用的账号冲突，只在本次子进程中排除覆盖，保留用户原配置。
3. 需要登录时，在支持交互的持久终端启动：

```bash
gh auth login --hostname github.com --git-protocol https --web
```

4. 读取 gh 返回的**本次**设备代码与授权 URL。GitHub 常见代码是 `XXXX-XXXX`：8 个字母或数字，中间有连字符；不是手机短信的 6 位验证码。不要编造代码或复用旧代码。
5. 使用浏览器工具新建可见的授权会话／窗口，打开 gh 提供的 GitHub 官方设备授权页；常见为 `https://github.com/login/device`。工具只支持标签页时新建任务专用标签并检查账号，不接管别人的页面。没有浏览器工具时用系统浏览器打开该链接，仍完成终端侧流程。
6. 告诉用户：“请在新打开的 GitHub 页面登录你自己的账号，输入终端显示的这次 8 位代码，核对 GitHub CLI 的授权信息后确认。”密码、2FA、passkey 和最终授权由用户操作。等待时继续 OS／工具盘点，不启动多个登录流程。
7. 保持原终端进程存活，等其明确成功；过期或拒绝就结束该次流程，待用户准备好后重新生成。不要把代码写入仓库或交接文档。
8. 检查凭据存储状态；优先系统凭据管理器，不主动使用 `--insecure-storage`。若环境只能明文保存，说明实际方式及改用系统凭据管理器的办法，不复制该认证文件到其他设备。

已有正确身份可复用。多账号需要切换时使用 `gh auth switch --hostname github.com --user 实际登录名`，随后再次用 API 核验；不因为机器保存了另一个高权限账号就擅自使用。

## 授权后的只读核验

下面 `OWNER/REPO` 是经用户确认并验证过的目标，替换后逐条运行；不把未验证的 URL 直接拼接进 shell。

```bash
gh api user --jq '{login,id}'
gh api repos/OWNER/REPO --jq '{full_name,default_branch,permissions}'
gh api repos/OWNER/REPO/rules/branches/main
```

zhyq 还核对 test 的规则。必要时检查 classic branch protection；404 可能是权限不足或未设置 classic protection，不能据此说“无保护”。

- `permissions.push == true`：当前 API 身份具备仓库写能力，包括合作者、团队或拥有者等来源。记录 login、full_name、push；不要求必须以“直接 collaborator”形式被添加。
- push 为 false／缺失／请求失败：写权限**未验证通过**。让管理员给这个准确登录名授予 Write 或更高权限，用户接受邀请后重新核验。能 clone 公开仓库只是读权限证明。
- 有 Write 仍可能被分支规则、SSO 或令牌授权范围阻止。读取规则，保留必要检查；`git push --dry-run` 也不能完全证明服务端所有接收规则会通过。
- 修改 `.github/workflows` 需要相应范围时，按 gh 提示请求补充 `workflow` 授权；不一开始索要管理组织／删除仓库权限。

## Git 使用同一身份

授权后用 `gh auth setup-git --hostname github.com` 连接 Git HTTPS 凭据。它会配置该主机的 Git 凭据助手；多账号机器先检查影响，上传前再次核对活动身份。

现有仓库检查 fetch URL、push URL、`pushurl`、URL 重写及凭据覆盖。结果中的令牌必须遮蔽。确认实际推送目标与用户仓库一致；不把含 token 的 URL 写成 remote，也不把浏览器／GitHub App 连接成功当成本地 Git 凭据成功。

提交作者只影响归属显示：保留已确认身份；缺失时使用该账号公开信息或 GitHub noreply 邮箱，在当前仓库设置 `git config --local user.name` 和 `user.email`。作者字段不会赋予推送权限。

参考：[gh auth login](https://cli.github.com/manual/gh_auth_login)、[gh auth setup-git](https://cli.github.com/manual/gh_auth_setup-git)、[GitHub 设备授权](https://docs.github.com/en/apps/oauth-apps/building-oauth-apps/authorizing-oauth-apps#device-flow)、[仓库 API](https://docs.github.com/en/rest/repos/repos#get-a-repository)。
