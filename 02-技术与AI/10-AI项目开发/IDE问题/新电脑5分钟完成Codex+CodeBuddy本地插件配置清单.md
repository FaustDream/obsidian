 新电脑 5 分钟完成 Codex + CodeBuddy 本地插件配置清单

这份文档只做一件事：换新电脑时，按顺序把本地插件市场快速配起来。

本文不写死盘符和用户名，统一使用下面两个占位符：

- `<仓库根目录>`：你 clone 下来的仓库根目录
- `<用户目录>`：当前电脑的用户主目录

如果你想先看完整原理，请看：

- [Codex和CodeBuddy插件市场正确配置.md](E:/gitHub/Obsidian/02-技术与AI/10-AI项目开发/IDE问题/Codex和CodeBuddy插件市场正确配置.md)

---

## 1. 先做这三件事

1. 安装并登录 Codex
2. 安装并登录 CodeBuddy
3. clone 仓库到本地：`<仓库根目录>`

---

## 2. Codex 5 分钟配置（推荐直接配全局版）

### 第一步：确认仓库文件存在

至少确认这两个文件：

- `<仓库根目录>\.agents\plugins\marketplace.json`
- `<仓库根目录>\plugins\workflow\.codex-plugin\plugin.json`

### 第二步：同步到全局目录

执行：

```powershell
pwsh.exe -File <仓库根目录>\scripts\sync-codex-home-workflow.ps1
```

这一步会把插件同步到：

- `<用户目录>\plugins\workflow`
- `<用户目录>\.agents\plugins\marketplace.json`

### 第三步：重启 Codex

全局 marketplace 新增后，建议直接重启 Codex 一次。

### 第四步：打开任意项目测试

打开：

```text
任意项目目录
```

### 第五步：看是否出现 workflow

如果没出现，先检查：

- `<用户目录>\.agents\plugins\marketplace.json` 是否存在
- `<用户目录>\plugins\workflow\.codex-plugin\plugin.json` 是否存在

### 只想当前仓库可见怎么办

如果你不需要全局版，只需要：

1. 用 Codex 打开 `<仓库根目录>`
2. 确认仓库里有 `.agents/plugins/marketplace.json`
3. 确认仓库里有 `plugins/workflow/.codex-plugin/plugin.json`

---

## 3. CodeBuddy 5 分钟配置

### 第一步：确认 marketplace 包装目录存在

至少确认这两个文件：

- `<仓库根目录>\codebuddy-marketplace\.codebuddy-plugin\marketplace.json`
- `<仓库根目录>\codebuddy-marketplace\plugins\workflow\.codebuddy-plugin\plugin.json`

### 第二步：必要时同步 skills

如果你刚更新过 `plugins/workflow/skills`，先执行：

```powershell
pwsh.exe -File <仓库根目录>\codebuddy-marketplace\scripts\sync-workflow.ps1
```

### 第三步：手动注册本地 marketplace

编辑这个文件：

```text
<用户目录>\.codebuddy\plugins\known_marketplaces.json
```

加入或确认存在下面这项：

```json
{
  "faustdream-local-marketplace": {
    "type": "directory",
      "source": {
        "source": "directory",
      "path": "<仓库根目录>\\codebuddy-marketplace"
    },
    "installLocation": "<仓库根目录>\\codebuddy-marketplace",
    "description": "Local CodeBuddy marketplace for FaustDream workflow plugins",
    "autoUpdate": false
  }
}
```

如果原文件里已经有别的 marketplace，就把这项并进现有 JSON 对象里，不要覆盖其他条目。

### 第四步：重启 CodeBuddy

重启后，打开插件管理界面，检查是否出现：

- `faustdream-local-marketplace`
- `workflow`

---

## 4. 30 秒快速验证命令

在 PowerShell 里执行：

```powershell
Get-Content -LiteralPath '<仓库根目录>\.agents\plugins\marketplace.json' -Raw | ConvertFrom-Json | Out-Null
Get-Content -LiteralPath '<仓库根目录>\plugins\workflow\.codex-plugin\plugin.json' -Raw | ConvertFrom-Json | Out-Null
Get-Content -LiteralPath '<仓库根目录>\codebuddy-marketplace\.codebuddy-plugin\marketplace.json' -Raw | ConvertFrom-Json | Out-Null
Get-Content -LiteralPath '<仓库根目录>\codebuddy-marketplace\plugins\workflow\.codebuddy-plugin\plugin.json' -Raw | ConvertFrom-Json | Out-Null
Get-Content -LiteralPath '<用户目录>\.codebuddy\plugins\known_marketplaces.json' -Raw | ConvertFrom-Json | Out-Null
```

如果没有报错，说明 JSON 至少是可解析的。

---

## 5. 最常见问题

### Codex 看不到插件

先查两件事：

1. 打开的是否是 `<仓库根目录>`
2. `.agents/plugins/marketplace.json` 里的 `source.path` 是否是 `./plugins/workflow`

### CodeBuddy 看不到插件

先查三件事：

1. `<用户目录>\.codebuddy\plugins\known_marketplaces.json` 里是否真的注册了本地 marketplace
2. `installLocation` 是否指向真实目录
3. 重启 CodeBuddy 后是否重新读取了配置

### 两边技能内容不一致

通常是因为改了：

```text
<仓库根目录>\plugins\workflow\skills
```

但没同步到：

```text
<仓库根目录>\codebuddy-marketplace\plugins\workflow\skills
```

直接执行：

```powershell
pwsh.exe -File <仓库根目录>\codebuddy-marketplace\scripts\sync-workflow.ps1
```

---

## 6. 最短记忆版

只记这四句：

1. Codex 主要靠仓库内约定路径自动发现
2. CodeBuddy 还要手动改一次 `known_marketplaces.json`
3. 核心 skills 只维护 `plugins/workflow/skills`
4. CodeBuddy 那份通过同步脚本刷新
