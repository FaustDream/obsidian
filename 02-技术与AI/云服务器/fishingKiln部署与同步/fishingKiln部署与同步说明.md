# fishingKiln 部署与同步说明

## 当前部署结果

- 公网地址：`http://121.37.222.241:46817/`
- Tailscale 内网地址：`http://100.124.65.93:46817/`
- Nginx 站点目录：`/www/wwwroot/fishingKiln`
- Nginx 配置文件：`/www/server/panel/vhost/nginx/fishingKiln-46817.conf`
- GitHub 仓库：`https://github.com/FaustDream/fishingKiln`
- 服务器仓库远程：`git@github.com-fishing-kiln:FaustDream/fishingKiln.git`
- 同步分支：`master`
- 已验证监听端口：`46817`

## 本次接入方式

本项目按与 `website-styles` 相同的模式接入：

- 网站目录本身是 Git 仓库
- Nginx 直接指向网站目录
- 仓库被纳入 `/root/repo-sync/repos.conf`
- 定时同步继续复用服务器现有统一 cron

## 服务器 Git 认证

本项目单独使用了一把 deploy key：

- 私钥：`/root/.ssh/fishing_kiln_sync_ed25519`
- 公钥：`/root/.ssh/fishing_kiln_sync_ed25519.pub`
- SSH Host Alias：`github.com-fishing-kiln`
- GitHub Deploy Key 标题：`fishingKiln repo-sync@hcss-ecs-b26a`

服务器 SSH 配置中追加了：

```sshconfig
Host github.com-fishing-kiln
    HostName github.com
    User git
    IdentityFile /root/.ssh/fishing_kiln_sync_ed25519
```

## 同步脚本登记

`/root/repo-sync/repos.conf` 中当前项目登记为：

```txt
fishingKiln|/www/wwwroot/fishingKiln|master|git@github.com-fishing-kiln:FaustDream/fishingKiln.git
```

手动同步命令：

```bash
bash /root/repo-sync/sync-repos.sh fishingKiln
```

本次手动同步结果：

```txt
OK repo=fishingKiln head=f2dee50
```

## Nginx 配置

当前站点配置：

```nginx
server
{
    listen 46817;
    server_name 121.37.222.241 100.124.65.93 _;
    index index.html index.htm;
    root /www/wwwroot/fishingKiln;
}
```

reload 命令：

```bash
/www/server/nginx/sbin/nginx -t -c /www/server/nginx/conf/nginx.conf
/www/server/nginx/sbin/nginx -s reload
```

## 本地 Git 说明

- 本地仓库已 push 到 GitHub
- `docs/` 已加入 `.gitignore`
- 本地提交时不再把仓库内 `docs` 目录带上 GitHub

## 定时同步说明

服务器已有统一 cron：

```cron
17 3 * * * /bin/bash /root/repo-sync/sync-repos.sh >> /root/repo-sync/logs/cron.log 2>&1
```

因为 `fishingKiln` 已加入 `repos.conf`，所以已经自动纳入定时同步，无需再单独新增一条 cron。

## 常用维护命令

查看当前仓库状态：

```bash
cd /www/wwwroot/fishingKiln
git status
git log -1 --oneline
```

手动同步：

```bash
bash /root/repo-sync/sync-repos.sh fishingKiln
```

查看端口监听：

```bash
ss -ltn | grep 46817
```

检查 HTTP：

```bash
curl -I http://127.0.0.1:46817/
```
