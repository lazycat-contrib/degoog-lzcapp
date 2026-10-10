# degoog-lzcapp

[Degoog](https://github.com/degoog-org/degoog)（搜索引擎聚合器 + 插件系统）的懒猫微服打包：**只发喵喵商店，镜像模式**（`delivery.mode: mirror`）。

- **镜像**：`ghcr.io/degoog-org/degoog:1.1.0`（容器内监听 **4444**，数据在 `/app/data`）。镜像模式只做只读校验，交付时把 manifest 的镜像行改写成 `ghcr.1ms.run/degoog-org/degoog:1.1.0`，不搬镜像到懒猫镜像源。
- **版本跟踪**：`channel: stable` + `sort: semver` + `tag_regex: '^[0-9]+\.[0-9]+\.[0-9]+$'`（排除 `1.1.0-amd64/-arm64` 变体与 `develop` / `main` / `latest`）。
- **运行用户**：入口脚本（`/entrypoint.sh`）以 root 起——先 `mkdir -p /app/data && chown -R 1000:1000 /app/data`，再 `su-exec 1000:1000`。因为平台建的 `/lzcapp/var/degoog` 属主是 root，manifest 里写 **`user: root`** 让这个 chown 生效（不用 `run_as`，也不用自己 chown）。
- **`DEGOOG_SETTINGS_PASSWORDS`**：安装向导填的**设置密码**，保护「设置」与插件安装。**别留空**——未上锁的实例任何人都能装插件，而插件在服务器上执行代码（等于 RCE）。所以这里没用自动生成：用户必须知道这个密码才能解锁后台。
- **`DEGOOG_PUBLIC_INSTANCE: "true"`**：微服域名访问不算"本地网络"，强制所有管理/插件动作都过密码（上游 simple.yml 写的是 `false`，那是给纯本地/局域网用的）。
- **路由**：`/` → `degoog:4444`，`public_path: [/]`（入口放行，管理动作靠上面的密码）。
- **健康检查**：`curl -fsS http://127.0.0.1:4444/readyz`（镜像自带 curl，端点与镜像自身 HEALTHCHECK 一致）。
- **持久化**：`/lzcapp/var/degoog:/app/data`（设置、SQLite 索引、已装插件）。
- **图标**：上游 `src/public/favicon/icon-512.png`。
- 单容器 + 内存缓存（个人/低流量形态，即上游 `simple.yml` 的拓扑）；要共享缓存或多副本再考虑 Valkey/Postgres 版本。

## 商店状态

- **喵喵商店**：`cloud.lazycat.app.degoog` 由 Action 发布（`stores.private`）。
- **官方商店**：镜像模式不满足官方商店「镜像必须来自 `registry.lazycat.cloud`」的要求，只发喵喵商店。
