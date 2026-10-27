# Negative 前端部署说明（GitHub Pages 版）

本包内的 `index.html` 是引擎前端的 **GitHub Pages 托管版**：社员访问
`https://beixianggumao.github.io/Negative/` 直接使用系统，所有网盘 API 由
**Vercel 后端**（Negative-Server 仓库）处理，前端文件本身零凭据、零密钥。

## 与旧版（v10.4 启动器）的区别

| | 旧版 v10.4 | 本版 |
|---|---|---|
| 打开方式 | 启动器读 bootpool.json → 跳网盘拉引擎 | 直接就是引擎，打开即用 |
| 依赖 | bootpool 令牌 + 网盘直链 + 每日 Actions 续期 | 仅依赖 Vercel 后端一个地址 |
| 断链风险 | config 直链失效即瘫痪（10-05 事故） | 无引导链，后端随时自主铸令牌 |
| 管理密码 | 藏在公开源码里（已泄露） | 只存于 Vercel 环境变量 |
| 更新方式 | 网盘覆盖 SVG | git push 即全站更新 |

## 部署步骤（约 3 分钟）

> 前置条件：Vercel 后端已部署完成并拿到地址（形如 `https://negative-server.vercel.app`）。

1. **改 1 行配置**：用文本编辑器打开 `index.html`，搜索 `YOUR-PROJECT`
   （在第 758 行附近的「部署配置区」），把
   ```js
   const SRV = location.hostname.endsWith('.github.io') ? 'https://YOUR-PROJECT.vercel.app' : location.origin;
   ```
   中的 `YOUR-PROJECT` 替换为你的 Vercel 项目名，保存。

2. **替换仓库文件**（在本仓库 `BeiXiangGuMao/Negative` 根目录）：
   - 用本包的 `index.html` **覆盖**仓库现有的 `index.html`（旧启动器）
   - **删除** `bootpool.json`（启动池已退役，新引擎不再读取）
   - 旧的 `使用说明.md` 可保留或移入 `docs/` 目录存档

3. **停用旧续期任务**：进入仓库 `Settings → Actions`，把原来每日 08:40
   续期令牌的 workflow **disable**（或直接删除 `.github/workflows/` 里
   对应文件）。新架构下令牌由 Vercel 后端每日 cron 自动保活，不再需要。

4. **提交发布**：
   ```bash
   git add -A
   git commit -m "deploy: 引擎前端直驻 GitHub Pages，网盘 API 由 Vercel 后端处理"
   git push
   ```

5. **验证**：等 Pages 构建完成（1~2 分钟），打开
   `https://beixianggumao.github.io/Negative/` → 登录 → 确认文档目录
   正常加载、任选一个文档能打开、上传一张测试图。

## 兜底线路（自动生效，无需配置）

同一份 `index.html` 也随 Vercel 后端部署（`negative-vercel/public/index.html`）。
当 GitHub Pages 不可达时，社员可直接打开 `https://<你的项目>.vercel.app/` ——
该地址下页面自动同源调用 API，无需任何额外配置。两处代码完全一致，更新时同步替换即可。

## 常见问题

| 现象 | 原因 | 处理 |
|---|---|---|
| 登录后一直转圈 / 控制台 CORS 报错 | SRV 行没改或 Vercel 地址写错 | 核对第 1 步，确认后端 `/api/health` 返回正常 |
| 控制台报 `Failed to fetch` | Vercel 后端未部署或环境变量缺失 | 打开 `https://<后端>/api/health` 检查 `ok` 与 `error` 字段 |
| 页面能开但目录为空 | 后端令牌铸造失败 | 查看 `/api/health` 返回的 `error`（多为 `YST_ADMIN_PWD` 未配） |
| GitHub Pages 打开 404 | 仓库名或 Pages 分支变动 | `Settings → Pages` 确认部署分支与 `/Negative/` 路径 |
