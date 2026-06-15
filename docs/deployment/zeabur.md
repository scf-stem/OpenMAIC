# Zeabur 部署指南

最后验证日期：2026-06-15

## 适配结论

OpenMAIC 在 Zeabur 上推荐使用仓库根目录的 `Dockerfile` 部署。仓库已新增 `zbpack.json`，显式声明 Zeabur 使用根目录 `Dockerfile`，避免 pnpm workspace 项目被误判为通用 Node.js 自动构建。

该部署无数据库迁移；无迁移，直接替换。

## Zeabur 操作步骤

1. 在 Zeabur Dashboard 新建 Project。
2. 选择 Add Service -> Git，连接 GitHub。
3. 选择仓库 `scf-stem/OpenMAIC`，Root Directory 保持仓库根目录。
4. 保持默认 Dockerfile 构建；Zeabur 会读取 `zbpack.json` 中的 `Dockerfile` 路径。
5. 在 Variables 页面配置环境变量，至少配置一个 LLM Provider API Key，并按需配置 `DEFAULT_MODEL`。
6. 在 Networking 页面生成 `.zeabur.app` 域名或绑定自定义域名。

## 端口与启动

- Dockerfile 默认 `EXPOSE 3000`，Next.js standalone server 通过 `node server.js` 启动。
- Dockerfile 设置 `HOSTNAME=0.0.0.0` 与默认 `PORT=3000`；Zeabur 若注入运行时 `PORT`，Next.js standalone 会读取运行时变量。
- 不需要在 Zeabur 手动覆盖 Build Command 或 Start Command。

## 必填与常用变量

至少选择一个 LLM Provider：

```env
OPENAI_API_KEY=...
OPENAI_BASE_URL=https://api.openai.com/v1
DEFAULT_MODEL=openai:gpt-5.5
```

共享部署建议设置：

```env
ACCESS_CODE=your-secret-code
```

如果需要将 OpenMAIC 嵌入其他站点 iframe：

```env
ALLOWED_FRAME_ANCESTORS=https://your-site.example
```

更多变量以 `.env.example` 为准，密钥只在 Zeabur Variables 中配置，不要提交到仓库。

## 持久化与文件

当前 OpenMAIC 主服务不依赖数据库迁移。若后续启用需要落盘的服务端文件，可在 Zeabur Volumes 中挂载 `/app/data`；本仓库的 `docker-compose.yml` 已使用同一路径作为本地容器卷。

## 回滚

Zeabur 通过 Git 提交触发部署。若新版本异常，在 Zeabur Deployments 中回滚到上一成功部署，或将 Git 分支回退到上一提交后重新部署。

## 证据

- `evidence/zeabur-sources.csv`
- `evidence/2026-06-15-zeabur-adaptation.md`
