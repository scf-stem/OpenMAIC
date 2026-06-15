# Zeabur 适配任务日志

日期：2026-06-15

## 目标

- 将上游 `THU-MAIC/OpenMAIC` fork 到 `scf-stem/OpenMAIC`。
- 适配 Zeabur 部署路径，保证 Zeabur 从 GitHub 导入后按 Dockerfile 构建运行。

## 输入来源

- 用户请求：部署 OpenMAIC 到 Zeabur，并先 fork 到 `scf-stem`。
- 本地仓库：`work/OpenMAIC`，origin 为 `https://github.com/scf-stem/OpenMAIC.git`，upstream 为 `https://github.com/THU-MAIC/OpenMAIC.git`。
- Zeabur 官方文档：见 `evidence/zeabur-sources.csv`。

## 关键判断

- OpenMAIC 是 Next.js 16 + pnpm workspace 项目，`package.json` 固定 `pnpm@10.28.0`，Dockerfile 已覆盖 workspace 安装、本地包构建、Next standalone 输出与运行时原生依赖。
- Zeabur 官方文档说明根目录存在 `Dockerfile` 时会自动使用 Docker 部署；为降低误判风险，新增 `zbpack.json` 显式指定 `Dockerfile` 路径。
- Zeabur 当前不支持直接部署 Docker Compose YAML，因此 `docker-compose.yml` 仅作为本地容器运行参考。
- 无数据库迁移，直接替换；密钥通过 Zeabur Variables 配置，不写入仓库。

## 变更清单

- 新增 `zbpack.json`：显式指定根目录 `Dockerfile`。
- 新增 `docs/deployment/zeabur.md`：中文部署步骤、端口、变量、回滚说明。
- 更新 `README-zh.md`：在部署章节加入 Zeabur 快速入口。
- 更新 `.dockerignore`：中文化注释并排除 `evidence/`。
- 新增 `evidence/zeabur-sources.csv` 与本任务日志。

## 不确定性

- 未访问用户 Zeabur 控制台，无法创建实际 Zeabur Project 或确认套餐资源限制。
- 本机未安装 `docker` 命令，无法执行本地镜像构建；已改用项目原生安装、测试与生产构建验证 Zeabur Dockerfile 中的核心 `pnpm build` 路径。

## 验证记录

- `node -e "const fs=require('fs'); JSON.parse(fs.readFileSync('zbpack.json','utf8')); console.log('zbpack.json ok')"`：通过，`zbpack.json ok`。
- `git diff --check`：通过，无空白错误。
- `pnpm install --frozen-lockfile`：通过，postinstall 完成；存在上游 workspace pnpm 配置 warning 与依赖 build script 审批 warning。
- `pnpm lint`：退出码 0；保留 19 个既有 warning，无 error。
- `pnpm test`：通过，108 个测试文件、806 个测试全部通过。
- `pnpm build`：通过，Next.js 16.1.2 生产构建完成；存在上游 middleware 文件约定弃用 warning。
- `docker version`：失败，`zsh:1: command not found: docker`。
