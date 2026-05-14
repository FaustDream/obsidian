# website-styles 部署与同步说明

## 当前部署状态

- 公网地址：`http://121.37.222.241:45231/`
- Tailscale 内网地址：`http://100.124.65.93:45231/`
- Nginx 站点目录：`/www/wwwroot/website-styles`
- Nginx 配置文件：`/www/server/panel/vhost/nginx/website-styles-45231.conf`
- Git 仓库远程：`git@github.com:FaustDream/website-styles.git`
- 分支：`master`

## 服务器上的同步目录

- 同步主目录：`/root/repo-sync`
- 仓库清单：`/root/repo-sync/repos.conf`
- 批量同步脚本：`/root/repo-sync/sync-repos.sh`
- 一次性接管脚本：`/root/repo-sync/bootstrap-repo.sh`
- 查看最新日志脚本：`/root/repo-sync/show-latest-log.sh`
- 日志目录：`/root/repo-sync/logs`

## 仓库维护方式

服务器采用“网站根目录本身就是 Git 仓库”的方式维护：

- 网站目录 = `/www/wwwroot/website-styles`
- 这个目录现在就是 Git 仓库
- 执行 `git pull` 成功后，网页内容立即更新
- 不做 `git push`，不改远程 GitHub 内容

## 仓库清单格式

`repos.conf` 每行一个仓库，格式：

```txt
name|path|branch|remote
```

当前示例：

```txt
# name|path|branch|remote
website-styles|/www/wwwroot/website-styles|master|git@github.com:FaustDream/website-styles.git
```

字段含义：

- `name`：仓库标识，用于单仓库同步
- `path`：服务器本地仓库目录
- `branch`：同步分支
- `remote`：GitHub SSH 仓库地址

## 常用命令

同步全部仓库：

```bash
bash /root/repo-sync/sync-repos.sh
```

只同步 `website-styles`：

```bash
bash /root/repo-sync/sync-repos.sh website-styles
```

查看最近一次同步日志：

```bash
bash /root/repo-sync/show-latest-log.sh
```

## 命令效果

`bash /root/repo-sync/sync-repos.sh`

- 读取 `/root/repo-sync/repos.conf`
- 逐个仓库执行同步
- 只对“已存在且本身就是 Git 仓库”的目录执行同步
- 同步方式是：
  `fetch origin branch -> checkout branch -> merge --ff-only origin/branch`
- 不会 `clone` 新仓库
- 不会 `push` 到 GitHub
- 工作区有未提交改动时会跳过，避免覆盖本地内容

`bash /root/repo-sync/sync-repos.sh website-styles`

- 只同步名为 `website-styles` 的仓库
- 成功时日志会出现：
  `OK repo=website-styles head=<commit>`

`bash /root/repo-sync/show-latest-log.sh`

- 输出最新一次同步日志文件路径
- 紧接着输出日志正文

## 当前定时任务

服务器当前已配置：

```cron
17 3 * * * /bin/bash /root/repo-sync/sync-repos.sh >> /root/repo-sync/logs/cron.log 2>&1
```

含义：

- 每天 `03:17` 自动同步一次
- 结果追加写入 `/root/repo-sync/logs/cron.log`

## GitHub 访问方式

服务器使用 Deploy Key 只读访问 GitHub：

- Key 类型：`SSH Deploy Key`
- 仓库：`FaustDream/website-styles`
- 权限：只读
- 作用：允许服务器 `git fetch` / `git pull`

## DNS 与网络说明

服务器已经修过系统 DNS，当前可以解析 `github.com`。如果后续出现：

```bash
Could not resolve host: github.com
```

优先检查：

- `/etc/systemd/resolved.conf`
- `systemd-resolved` 状态
- 服务器是否还能访问公网 DNS

## 新增仓库的方式

如果后续要同步多个仓库，只需要在：

`/root/repo-sync/repos.conf`

继续追加一行，例如：

```txt
my-repo|/www/wwwroot/my-repo|master|git@github.com:FaustDream/my-repo.git
```

追加后可直接执行：

```bash
bash /root/repo-sync/sync-repos.sh
```

如果新目录还不是 Git 仓库，先单独处理为 Git 仓库，再纳入定时同步。

## 排查要点

`sync-repos.sh` 失败时，先看：

```bash
bash /root/repo-sync/show-latest-log.sh
```

常见原因：

- `path_not_found`：本地目录不存在
- `not_git_repo`：目录存在，但不是 Git 仓库
- `working_tree_dirty`：工作区有未提交修改
- `index_dirty`：暂存区有未提交修改
- `fetch_failed`：无法从 GitHub 拉取，通常是 SSH 权限或网络问题
- `checkout_failed`：目标分支不存在或仓库状态异常
- `merge_failed`：不能快进合并，需要人工处理
