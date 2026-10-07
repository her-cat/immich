# tzfix —— Immich 时区兜底修复

放在 fork（`her-cat/immich`）里，定时/手动构建打了补丁的 `immich-server` 镜像并推送到 GHCR。

## 组成
| 路径 | 作用 |
|---|---|
| `tzfix/tz-fix.patch` | 修复补丁（仅改 `server/src/services/metadata.service.ts` 的 `getDates()`） |
| `.github/workflows/tzfix-build.yml` | 主流程：取上游最新 release → 打补丁 → 构建 → 推 GHCR |
| `.github/workflows/sync-upstream.yml` | 可选：把 fork 的 main 与上游 merge 同步 |

## 产物
- `ghcr.io/her-cat/immich-server:tzfix-<version>`（如 `tzfix-3.2.4`）
- `ghcr.io/her-cat/immich-server:tzfix-latest`（跟随最新构建）

## 工作原理
1. 每天 03:17 UTC 触发（或用 `workflow_dispatch` 手动指定版本）。
2. 从 `immich-app/immich` 取**上游最新 release tag** 的源码（不依赖 fork 的分支状态）。
3. 应用 `tzfix/tz-fix.patch`；**打不上会直接失败**（说明上游改了这段代码，需要更新补丁）。
4. `docker/build-push-action` 用官方 `server/Dockerfile` 构建并推送。
5. 若该版本镜像已存在则跳过（可用 `force` 强制重建）。

## 维护
- 上游升级若改了 `getDates()`，workflow 会在 "Apply tz fix" 步骤失败，需要更新 `tzfix/tz-fix.patch`。
- 基线补丁针对 `v3.2.4`；对比参考：`yann117/immich-tz-fix` 的 `fix/tz-fix` 分支。
