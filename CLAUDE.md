# 本仓库是带自定义补丁的 fork（先读这里）

`IShinji/Lite-web` 是 `nuomiiiii/Lite-web`（Lite 管理端前端）的 fork，叠了两层**本地补丁**，上游不接受，由本 fork 自己维护。
**完整的同步上游、构建、发布流程在后端仓库的 `/Users/wisely/Documents/GitHub/Lite/CLAUDE.md`，先读那份。**

## 补丁（分支栈：`main` → `local-patches/browser-tz` → `local-patches/no-self-update`）

- `local-patches/browser-tz`：仪表盘三个请求（`/api/admin/dashboard`、`/charts`、`/traffic-day`）带浏览器时区 `tz`；缓存键按时区隔离（`key@tz`）；`shortDashboardDay` 不再按 `+08:00` 解析；后端标记 `partial` 的天在 tooltip 提示“数据不完整”。
  文件：`src/utils/dashboard.ts`、`src/utils/dashboardApi.ts`、`DashboardTraffic30d.tsx`、`DashboardPanels.tsx`、四个语言文件、`tests/dashboard.test.ts`。
- `local-patches/no-self-update`：后端返回 `reason=local_patch` 时隐藏“立即更新”，显示“请自行同步上游并重新构建”。
  文件：`UpdateReleaseDialog.tsx`、`adminShellModel.ts`（`isLocalPatchSelfUpdate`）、四个语言文件、`tests/adminShellModel.test.ts`。

计费/到期/周期重置里的 `Asia/Shanghai` 是有意保留的，不要改。

## 构建与同步要点

- fork 的 `Snapshot` 分支 = `local-patches/no-self-update`。后端 Lite 的 `snapshot.yml` 构建会拉本 fork 的 `Snapshot`，所以**先推本仓库的 `Snapshot`，再推后端的**。
- 同步上游：`git fetch upstream`，`main` 做 `merge --ff-only`，再依次 rebase 两层补丁。上游的 i18n 机器人会往 `main` 提交语言文件，四个语言文件在 `self_update_failed` 后和 dashboard 相关键处最容易冲突，注意补回 `partial_day`、`self_update_local_patch` 两个键。
- 复测：`npm ci`、`npx tsc -b`、`node --test tests/dashboard.test.ts tests/adminShellModel.test.ts`。已知 `tests/dashboardSettings.test.ts` 里 2 个延迟排行测试和 `npx eslint src tests` 的 22 个错误在上游 main 上就存在，与补丁无关。
- 不要自己 push：把命令列成“每条一行”的 `! ` 命令交给用户执行。不 ssh、不碰线上、不读凭据。
