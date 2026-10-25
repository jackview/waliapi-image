# waliapi-image

自动构建 [fuzhengwei/WaLiAPI](https://github.com/fuzhengwei/WaLiAPI) 最新版 Docker 镜像。

## 工作方式

`.github/workflows/build.yml` 每 6 小时（及手动触发时）执行：

1. 通过 GitHub API 获取上游最新 tag（`waliapi-v*`）
2. 与 `.last-built-version` 对比，无新版本则跳过
3. 有新版本：checkout 上游对应 tag 源码 → Buildx 构建 → 推送到 GHCR
4. 把已构建版本写回 `.last-built-version` 并提交

## 镜像地址

```bash
docker pull ghcr.io/jackview/waliapi:latest        # 最新版
docker pull ghcr.io/jackview/waliapi:v0.4.5        # 指定版本
```

## fnOS 自动升级（配合 Watchtower）

```yaml
services:
  waliapi:
    image: ghcr.io/jackview/waliapi:latest
    container_name: waliapi
    restart: unless-stopped
    ports:
      - "8080:8080"        # 按上游 docker-compose.yml 实际端口调整
    volumes:
      - ./data:/data       # 按上游 docker-compose.yml 实际路径调整，升级不丢数据

  watchtower:
    image: containrrr/watchtower
    container_name: watchtower
    restart: unless-stopped
    command: waliapi --interval 3600
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
```

## 注意

- 首次构建成功后，GHCR 包默认是**私有**的，需到 https://github.com/jackview?tab=packages
  → `waliapi` → Package settings → Change visibility → **Public**，fnOS 才能匿名拉取
- 构建包含 Rust 编译，单次约 20~40 分钟
- 手动触发：Actions → Auto build upstream image → Run workflow
