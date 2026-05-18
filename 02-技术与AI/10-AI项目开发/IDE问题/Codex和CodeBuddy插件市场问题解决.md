# Codex 和 CodeBuddy 插件市场问题解决

这份文档只讲一件事：**插件市场为什么不显示、为什么 `@` 不出来、为什么配置了还是没生效**。

如果你想先按正确结构配置，请看：

- [Codex和CodeBuddy插件市场正确配置.md](E:/gitHub/Obsidian/02-技术与AI/10-AI项目开发/IDE问题/Codex和CodeBuddy插件市场正确配置.md)

---

## 1. 在别的项目里 `@workflow` 不出来

### 现象

在 `my_skills` 仓库里能看到 `workflow`，但打开其他项目后，`@workflow` 不显示。

### 根因

你现在配的是**项目插件**，不是**全局插件**。

项目插件只在当前仓库生效：

- `<仓库根目录>\.agents\plugins\marketplace.json`
- `<仓库根目录>\plugins\workflow`

如果想在别的项目里也能 `@workflow`，还需要配置：

- `<用户目录>\.agents\plugins\marketplace.json`
- `<用户目录>\plugins\workflow`

### 正确处理

执行：

```powershell
pwsh.exe -File <仓库根目录>\scripts\sync-codex-home-workflow.ps1
```

然后重启 Codex。

---

## 2. 当前仓库里也看不到 `workflow`

### 先查这三件事

1. 打开的是否是仓库根目录
2. `<仓库根目录>\.agents\plugins\marketplace.json` 是否存在
3. `source.path` 是否是 `./plugins/workflow`

### 常见错误

- 把插件目录放成了 `<仓库根目录>\workflow`
- `source.path` 写成 `./workflow`
- 打开的只是 `plugins/workflow` 子目录，不是仓库根目录

---

## 3. CodeBuddy 里找不到插件

### 先查这四件事

1. `<用户目录>\.codebuddy\plugins\known_marketplaces.json` 是否真的注册了本地 marketplace
2. `<仓库根目录>\codebuddy-marketplace\.codebuddy-plugin\marketplace.json` 是否存在
3. `<仓库根目录>\codebuddy-marketplace\plugins\workflow\.codebuddy-plugin\plugin.json` 是否存在
4. 重启 CodeBuddy 后是否重新读取了配置

### 常见误区

- 以为 CodeBuddy 会读取 `.agents/plugins/marketplace.json`
- 以为 CodeBuddy 会读取 `.codex-plugin/plugin.json`
- 只创建了插件目录，但没有注册 `known_marketplaces.json`

---

## 4. 改了 workflow，但全局版或 CodeBuddy 版没更新

### 根因

你真正维护的是：

```text
<仓库根目录>\plugins\workflow
```

但全局版和 CodeBuddy 版只是副本。

### 正确处理

刷新 Codex 全局版：

```powershell
pwsh.exe -File <仓库根目录>\scripts\sync-codex-home-workflow.ps1
```

刷新 CodeBuddy 版：

```powershell
pwsh.exe -File <仓库根目录>\codebuddy-marketplace\scripts\sync-workflow.ps1
```

---

## 5. JSON 看起来没问题，但就是不生效

### 先验证是不是合法 JSON

```powershell
Get-Content -LiteralPath '<仓库根目录>\.agents\plugins\marketplace.json' -Raw | ConvertFrom-Json | Out-Null
Get-Content -LiteralPath '<仓库根目录>\plugins\workflow\.codex-plugin\plugin.json' -Raw | ConvertFrom-Json | Out-Null
Get-Content -LiteralPath '<用户目录>\.agents\plugins\marketplace.json' -Raw | ConvertFrom-Json | Out-Null
Get-Content -LiteralPath '<用户目录>\plugins\workflow\.codex-plugin\plugin.json' -Raw | ConvertFrom-Json | Out-Null
Get-Content -LiteralPath '<仓库根目录>\codebuddy-marketplace\.codebuddy-plugin\marketplace.json' -Raw | ConvertFrom-Json | Out-Null
Get-Content -LiteralPath '<仓库根目录>\codebuddy-marketplace\plugins\workflow\.codebuddy-plugin\plugin.json' -Raw | ConvertFrom-Json | Out-Null
Get-Content -LiteralPath '<用户目录>\.codebuddy\plugins\known_marketplaces.json' -Raw | ConvertFrom-Json | Out-Null
```

如果报错，优先检查：

- 多余逗号
- 路径里的反斜杠是否转义
- 手工改 JSON 时是否破坏了对象层级

---

## 6. 全局配好了，还是不生效

### 优先判断

是不是**还没重启 Codex**。

因为全局 marketplace 新增后，通常要让 Codex 重新启动一次，才会重新扫描：

- `<用户目录>\.agents\plugins\marketplace.json`
- `<用户目录>\plugins\workflow`

### 顺序建议

1. 同步全局插件
2. 关闭 Codex
3. 重新打开 Codex
4. 打开任意项目
5. 再试 `@workflow`

---

## 7. 不要做的事

### 不要手工改缓存

不要手工改：

```text
<用户目录>\.codex\plugins\cache\...
```

原因：

- 这是缓存，不是源
- Codex 刷新后会被覆盖
- 改了也不稳定，容易误判为“已经修好”

### 不要混用两套规范

- Codex 用 `.agents/plugins/marketplace.json` 和 `.codex-plugin/plugin.json`
- CodeBuddy 用 `.codebuddy-plugin/marketplace.json`、`.codebuddy-plugin/plugin.json` 和 `known_marketplaces.json`

这两套结构不是一回事。

---

## 8. 最短判断法

只记三句：

1. 当前仓库能用，别的项目不能用：缺的是 Codex 全局 marketplace
2. CodeBuddy 看不到：优先查 `known_marketplaces.json`
3. 改了 workflow 不生效：优先查有没有跑同步脚本和重启应用
