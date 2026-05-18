### git同步时全是“查询远端状态失败，网络连接异常”
本机 Git全局 配置里还残留旧代理
执行命令把本机 Git 全局代理改成最新端口
```
git config --global http.proxy http://127.0.0.1:xxxxx
git config --global https.proxy http://127.0.0.1:xxxxx
```
