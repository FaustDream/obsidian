# qq_xiaoqi_bot 部署说明

## 服务器信息

- Tailscale 节点名：`hcss-ecs-b26a`
- Tailscale IP：`100.124.65.93`
- 登录用户：`root`
- 系统：`Ubuntu`
- Python：`3.10.6`

## 当前部署路径

- 本地项目目录：`D:\WorkCode\Python\qq_xiaoqi_bot`
- 本地部署压缩包：`D:\WorkCode\Python\qq_xiaoqi_bot_deploy.zip`
- 服务器目录：`/opt/qq_xiaoqi_bot`
- 服务文件：`/etc/systemd/system/qq-xiaoqi-bot.service`

## 当前保留文件

本地目录只保留部署必要文件：

- `.env`
- `.env.example`
- `.gitignore`
- `bot_service.py`
- `config.py`
- `llm_client.py`
- `memory_store.py`
- `qa_logger.py`
- `README.md`
- `requirements.txt`
- `deploy/qq-xiaoqi-bot.service`

已清理内容：

- `.vendor/`
- `__pycache__/`
- `logs/`
- `botpy.log`

## 连接服务器

优先尝试：

```bash
tailscale ssh root@100.124.65.93
```

本次实际可用方式：

```bash
ssh root@100.124.65.93
```

连通性检查：

```powershell
tailscale status
tailscale ping 100.124.65.93
Test-NetConnection 100.124.65.93 -Port 22
```

## 打包与上传

本次实际使用的是排除无关目录后的 zip 包上传。

上传命令：

```powershell
scp D:\WorkCode\Python\qq_xiaoqi_bot_deploy.zip root@100.124.65.93:/opt/qq_xiaoqi_bot/qq_xiaoqi_bot_deploy.zip
```

## 服务器部署命令

解压并安装依赖：

```bash
cd /opt/qq_xiaoqi_bot
unzip -o qq_xiaoqi_bot_deploy.zip
python3 -m pip install -r requirements.txt
```

注册并启动服务：

```bash
cp deploy/qq-xiaoqi-bot.service /etc/systemd/system/qq-xiaoqi-bot.service
systemctl daemon-reload
systemctl enable --now qq-xiaoqi-bot
systemctl start qq-xiaoqi-bot.service
```

## 状态检查命令

查看服务状态：

```bash
systemctl status qq-xiaoqi-bot.service --no-pager -l
```

查看是否运行：

```bash
systemctl is-active qq-xiaoqi-bot.service
```

查看最近日志：

```bash
journalctl -u qq-xiaoqi-bot.service -n 50 --no-pager
```

查看完整部署目录：

```bash
ls -la /opt/qq_xiaoqi_bot
```

## 当前验证结果

- `py_compile` 已在服务器通过
- `qq-botpy` 已安装到 `/usr/local/lib/python3.10/dist-packages`
- 服务状态：`active`
- 启动日志包含：`机器人「小琪」启动成功`

## 关键日志特征

正常启动会出现这些关键信息：

- `登录机器人账号中`
- `access_token expires_in 7200`
- `程序启动`
- `会话启动中`
- `鉴权中`
- `机器人「小琪」启动成功`
- `心跳维持启动`

## 更新服务的最短流程

1. 在本地修改 `D:\WorkCode\Python\qq_xiaoqi_bot`
2. 重新打包为 `D:\WorkCode\Python\qq_xiaoqi_bot_deploy.zip`
3. 重新执行 `scp` 上传到 `/opt/qq_xiaoqi_bot/`
4. 在服务器执行：

```bash
cd /opt/qq_xiaoqi_bot
unzip -o qq_xiaoqi_bot_deploy.zip
python3 -m pip install -r requirements.txt
systemctl restart qq-xiaoqi-bot.service
systemctl status qq-xiaoqi-bot.service --no-pager -l
```

## 备注

- 这次 `tailscale ssh` 包装器返回过 `502 Bad Gateway`，但 Tailscale 网络与 `22` 端口本身是通的，所以改用 `ssh/scp + Tailscale IP` 完成部署。
- 如果后续服务不回消息，先查 `systemctl status`，再查 `journalctl`。
