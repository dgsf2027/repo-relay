# xjxl0104/zhyq 专用参考

仅匹配 `https://github.com/xjxl0104/zhyq` 的规范化身份。fork 或别人的同名 zhyq 不自动继承生产流程。

## 核对基线

日期：2026-09-15。读取基线 test/main 指向 `af4ebc0900d0aa03ef2d49817d59c9cd3ee95d55`；每次接手重新 fetch，不把此 SHA 当成永久版本。

- 默认开发分支 test，生产 main。
- GitHub Actions API 当时返回工作流数量 0，test 文件树没有 `.github/workflows`。
- main 标为 protected；可见有效 rules 只有禁止删除、禁止非 fast-forward 更新。未能读取 classic protection 详情。**protected 不等于必须 PR，也不保证所有账号都可直推。**
- 当前文档机制：**AI 本地自查并合并 test → main**，然后服务器 cron 每 5 分钟检查 main、构建部署。“自动检测”目前是 Agent 的检查，不是已存在的 GitHub Actions 合并机器人。
- 文档估计整体约 10 分钟，记载失败会回滚旧镜像。发布本 skill 时没有登录服务器验证 cron、日志或生产状态，不将文档估计作为上线承诺。

优先读取最新 `CLAUDE.md`、`PLAYBOOK.md`、`docs/PATTERN.md`、相关 `docs/memory/` 和配置。运维手册已经标明 ver* 自动上线段落作废；不执行旧版切 ver*、改默认分支等步骤。

## 本地开发

用户要改代码时，优先宿主机热更新前后端 + 本地 MySQL；只验整套部署时可选全 Docker。

| 项目 | 仓库证据及处理 |
|---|---|
| 后端 | backend/pom.xml：JDK 17，Spring Boot 3.2.5；Maven，命令在 backend 执行 |
| 前端 | frontend/package.json、pnpm 锁文件；使用 pnpm，不另生成 npm 锁文件 |
| Node / pnpm | 核对时 package.json 无 engines/packageManager，Dockerfile 使用 Node 20、pnpm 9；这是容器基线，宿主选兼容且受支持的版本并验证，不硬套旧机 Node 24 / pnpm 11 |
| MySQL | docker-compose.yml：mysql:8.0，宿主 3316，库 zhyq_park |
| 后端 | 默认 http://localhost:8090/api |
| 前端 | 默认 http://localhost:5273，Vite 代理后端 8090 |
| 完整容器版 | docker-compose.full.yml：MySQL + backend + frontend，默认前端 80 |

两套 Compose 有固定 `container_name`，换 Compose 项目名不能保证隔离；不要同时启动它们争用同名容器／数据。已有 zhyq 容器时先辨认用途及卷，只使用确认过的本地环境。

### 热更新开发

1. 验证 Git、gh、JDK 17、Maven、兼容 Node、pnpm 和可运行的 Docker。
2. 读当前 Compose 后，在仓库根启动 `docker compose up -d mysql`；检查 Docker context 是本机，必要时用本地配置限制数据库只监听 loopback，验证健康状态。
3. 当前 application.yml 要求 `DB_PASSWORD`、`JWT_SECRET`；加密功能还需 `ZHYQ_FIELD_ENCRYPTION_KEY` 或 `APP_ENCRYPTION_KEY`。数据库密码匹配实际本地库；已有数据卷不会因改环境变量就自动改密码。
4. 在被 Git 忽略的本地配置中生成并保留 JWT／字段加密密钥，显式注入后端进程。默认 DB_URL 是 localhost:3316、DB_USERNAME 是 root；不同配置就显式覆盖。JWT 与字段密钥分别生成，字段密钥按项目要求为 Base64 编码的 32 字节随机值。
5. 在 backend 运行 `mvn spring-boot:run`。Flyway 会修改目标本地库，先核对连接，不能误连生产。根目录 `.env` 不会自动被 Spring Boot 加载。
6. 在 frontend 运行 `pnpm install --frozen-lockfile`，核对必要依赖构建脚本获准执行，再运行 `pnpm dev --host 127.0.0.1`。端口占用时报告实际地址并同步本地代理／CORS，不杀其他进程。
7. 验证本地登录、API 和本次要改的页面。用当前种子或初始化说明，不依赖旧演示密码永远有效，不尝试生产账号。

### 全 Docker

旧文档“克隆后直接 up”已不完整：当前 full Compose 要求 `.env`，至少包含 `DB_ROOT_PASSWORD`、`JWT_SECRET`；加密功能另需字段密钥。为本机新库生成本地值，不搬生产 `.env`。

验证忽略规则、变量存在性、端口／卷后，在仓库根运行：

```bash
docker compose -f docker-compose.full.yml config --quiet
docker compose -f docker-compose.full.yml up -d --build
docker compose -f docker-compose.full.yml ps
```

不输出完整展开的 Compose 配置，以免泄露环境值。检查容器、后端日志和页面；本地 uploads 不随 Git 迁移。停止用对应 `docker compose ... down` 保留卷，不加 `-v`。

全新库如需 `space/reconcile` 和关系回填，按最新 DEPLOY.md 检查适用性；先证明目标仅为本机新建测试库，不复制旧命令到生产。

## 上传最低检查

遵循 [publishing.md](publishing.md) 的完整范围审查与并发处理，在最终合并候选目录执行：

```bash
# backend 目录
mvn -B test-compile

# frontend 目录
pnpm build
```

`test-compile` 不能降为 `compile`：服务器 Dockerfile 的 `package -DskipTests` 仍编译测试源码。`test-compile` 也不等于测试通过；运行改动相关后端测试、前端 `pnpm test` 和必要业务验收。集成测试可能需要本地 MySQL，缺库时不能说全量测试通过。

纯文档可免应用构建，仍检查 diff；整批 test → main 混有代码时，不能因“我只改文档”免构建。

金额／合同状态、权限／CORS／对外接口按最新项目规则补充审查，具体高风险问题先说明。数据库迁移提醒向 **xjxl** 索要服务器密钥／授权，先查生产 Flyway 历史，禁止照抄旧文档 V42／V51 猜下一个号。

## 生产验收

普通代码推 main 后依赖现有服务器轮询，不需要每位同事先持有私钥。没有服务器权限时可用已知正式网址和版本接口验收；只有页面可达时最多报告“页面可访问，版本待确认”。

有适用授权时只核对当前项目的日志、运行版本、健康和本次功能。项目文档记载部署日志为 `/opt/zhyq/deploy.log`；别沿用个人 SSH 别名，不重启共享服务器其他项目，不自动执行旧文档 `--force` 部署。

约 10–15 分钟仍无目标版本证据，报告“main 已推，部署待验证／失败”及具体日志或缺少权限。修复后以新提交走同一流程；数据库恢复单独评估。

来源：[项目分支纪律](https://github.com/xjxl0104/zhyq/blob/test/CLAUDE.md)、[开发手册](https://github.com/xjxl0104/zhyq/blob/test/PLAYBOOK.md)、[部署说明](https://github.com/xjxl0104/zhyq/blob/test/docs/DEPLOY.md)、[仓库文件](https://github.com/xjxl0104/zhyq)。
