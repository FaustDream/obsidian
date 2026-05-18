✅ 结论：你现在不是“创建插件”的问题，而是“让一个已经做好的插件被 Codex 正确识别”。最小白做法就是只检查 3 个地方。你现在一直失败，通常就是少了其中 1 个，或者放错位置。

假设你现成插件目录是：

```text
D:\gitHub\my_skills\plugins\workflow
```

这个目录里面已经有：
- `.codex-plugin\plugin.json`
- `skills\...`

那你只做下面 3 步。

**第 1 步：把现成插件放到 Codex 认的固定位置**

Codex 不认你随便放的目录，它更稳妥认的是：

```text
C:\Users\Lynn\plugins\workflow
```

所以你要把现成插件复制或联接到这里。

最简单是复制：

```powershell
New-Item -ItemType Directory -Force "C:\Users\Lynn\plugins"
Copy-Item `
  -Path "D:\gitHub\my_skills\plugins\workflow" `
  -Destination "C:\Users\Lynn\plugins\workflow" `
  -Recurse
```

如果你不想复制，想保持和原目录同步，就用联接：

```powershell
New-Item -ItemType Directory -Force "C:\Users\Lynn\plugins"
New-Item -ItemType Junction `
  -Path "C:\Users\Lynn\plugins\workflow" `
  -Target "D:\gitHub\my_skills\plugins\workflow"
```

注意：
- 不能只放到 `C:\Users\Lynn\.codex\plugins\cache\...`
- `cache` 只是缓存，不是正式安装位置

**第 2 步：告诉 Codex 这个插件存在**

必须有这个文件：

```text
C:\Users\Lynn\.agents\plugins\marketplace.json
```

如果没有，就新建。

内容写这个：

```json
{
  "name": "faustdream-local",
  "interface": {
    "displayName": "FaustDream Local"
  },
  "plugins": [
    {
      "name": "workflow",
      "source": {
        "source": "local",
        "path": "./plugins/workflow"
      },
      "policy": {
        "installation": "INSTALLED_BY_DEFAULT",
        "authentication": "ON_INSTALL"
      },
      "category": "Productivity"
    }
  ]
}
```

如果目录不存在，先建目录：

```powershell
New-Item -ItemType Directory -Force "C:\Users\Lynn\.agents\plugins"
```

这里最容易错的是：
- `name` 必须和插件目录名一致，这里是 `workflow`
- `path` 必须是 `./plugins/workflow`

**第 3 步：在 Codex 配置里启用它**

打开这个文件：

```text
C:\Users\Lynn\.codex\config.toml
```

确认里面有这两段：

```toml
[marketplaces.faustdream-local]
source_type = "local"
source = '\\?\C:\Users\Lynn'

[plugins."workflow@faustdream-local"]
enabled = true
```

少了就补上。

**最后一步：完全退出 Codex，再重开**

不是新建聊天，是：
1. 关闭整个 Codex
2. 重新打开

然后测试：

```text
@workflow
```

如果 `@workflow` 还是不弹，继续试：

```text
[@workflow](plugin://workflow@faustdream-local)
```

如果这个能用，说明插件已经装进去了，只是 `@` 自动补全没刷新。

**你现在只要检查这 3 件事**

1. 插件有没有在这里  
`C:\Users\Lynn\plugins\workflow`

2. marketplace 有没有在这里  
`C:\Users\Lynn\.agents\plugins\marketplace.json`

3. config 有没有这两段启用配置  
`C:\Users\Lynn\.codex\config.toml`

**一句话判断失败原因**

- 只有原插件目录：不行
- 只有 `.codex\plugins\cache`：不行
- 只有 marketplace：不行
- 只有 config：不行
- 这 3 个都齐了，再重启，才最容易成功
