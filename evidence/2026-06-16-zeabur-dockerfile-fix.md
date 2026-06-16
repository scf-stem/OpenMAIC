# Zeabur Dockerfile 构建失败修复日志

日期：2026-06-16

## 触发问题

Zeabur 部署 `deployment-6a30ff763850703aa49b535b` 在 Dockerfile `deps` 阶段失败：

```text
Error: Cannot find module '/app/scripts/sync-maic-importer.mjs'
ELIFECYCLE Command failed with exit code 1.
process "/bin/sh -c pnpm install --frozen-lockfile" did not complete successfully
```

## 根因

Dockerfile 的 `deps` 阶段只复制了：

- `package.json`
- `pnpm-lock.yaml`
- `pnpm-workspace.yaml`
- `packages/`

但根目录 `package.json` 的 `postinstall` 会执行：

```bash
node scripts/sync-maic-importer.mjs
```

因此 Zeabur 的 Docker 构建环境中缺少 `/app/scripts/sync-maic-importer.mjs`，导致 `pnpm install --frozen-lockfile` 在 postinstall 阶段失败。

进一步检查发现，`postinstall` 会生成 `public/vendor/maic-importer/`，而后续 `pnpm build` 会通过 `scripts/assert-vendor-maic-importer.mjs` 校验该产物存在。builder 阶段也需要继承 deps 阶段生成的 vendor 产物。

## 修复

- 在 Dockerfile `deps` 阶段新增复制 `scripts/sync-maic-importer.mjs`。
- 在 Dockerfile `builder` 阶段新增复制 `/app/public/vendor/maic-importer`。

## 验证记录

- RED 复现：按旧 deps 复制规则创建临时目录，确认 `scripts/sync-maic-importer.mjs` 缺失。
- GREEN 模拟：按新 deps 复制规则创建临时目录，确认 `scripts/sync-maic-importer.mjs` 存在。
- Zeabur 失败层模拟：在仅包含 deps 阶段输入文件的临时目录中执行 `pnpm install --frozen-lockfile`，通过，并生成 `public/vendor/maic-importer/index.js`。
- `pnpm build`：通过，Next.js 16.1.2 生产构建完成；保留上游 middleware 文件约定弃用 warning。
- `pnpm test`：通过，108 个测试文件、806 个测试全部通过。
- `pnpm lint`：退出码 0；保留 19 个既有 warning，无 error。
- `git diff --check`：通过，无空白错误。
- `docker version`：失败，`zsh:1: command not found: docker`；本机仍无法执行真实 Docker build，Zeabur 需重新部署确认平台端镜像构建。

## 迁移

无迁移，直接替换。
