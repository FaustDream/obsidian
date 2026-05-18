# Codex 和 CodeBuddy 插件市场正确配置

这份文档只讲一件事：**怎样按正确结构配置 Codex 和 CodeBuddy 的插件市场**。

如果你想快速排查问题，请看：

- [Codex和CodeBuddy插件市场问题解决.md](E:/gitHub/Obsidian/02-技术与AI/10-AI项目开发/IDE问题/Codex和CodeBuddy插件市场问题解决.md)

如果你是换新电脑想 5 分钟搞定，请看：

- [新电脑5分钟完成Codex+CodeBuddy本地插件配置清单.md](E:/gitHub/Obsidian/02-技术与AI/10-AI项目开发/IDE问题/新电脑5分钟完成Codex+CodeBuddy本地插件配置清单.md)

---

## 1. 先分清三种配置

### Codex 全局插件

作用：

- 在任意项目里都能看到插件
- 适合 `@workflow` 这种希望跨项目复用的插件

配置位置：

- `<用户目录>\.agents\plugins\marketplace.json`
- `<用户目录>\plugins\workflow`

### Codex 项目插件

作用：

- 只在当前仓库里可见
- 适合某个仓库私有的插件或实验性插件

配置位置：

- `<仓库根目录>\.agents\plugins\marketplace.json`
- `<仓库根目录>\plugins\workflow`

### CodeBuddy 本地 marketplace

作用：

- 让 CodeBuddy 能识别你自己的本地插件市场
- 目前需要手动注册 marketplace

配置位置：

- `<仓库根目录>\codebuddy-marketplace`
- `<用户目录>\.codebuddy\plugins\known_marketplaces.json`

---

## 2. Codex 全局插件正确配置

如果你希望 `workflow` 在**任何项目**里都能 `@workflow`，应该配这一套。

### 目录结构

```text
<用户目录>
├─ .agents
│  └─ plugins
│     └─ marketplace.json
└─ plugins
   └─ workflow
      ├─ .codex-plugin
      │  └─ plugin.json
      └─ skills
```

### 推荐做法

不要手工维护两套 workflow 内容。  
推荐把仓库里的 `plugins/workflow` 作为唯一维护源，然后同步到全局目录。

本仓库已经有同步脚本：

```powershell
pwsh.exe -File <仓库根目录>\scripts\sync-codex-home-workflow.ps1
```

这个脚本会做两件事：

1. 把 `<仓库根目录>\plugins\workflow` 同步到 `<用户目录>\plugins\workflow`
2. 创建或更新 `<用户目录>\.agents\plugins\marketplace.json`

### 全局 marketplace 最小示例

文件：

```text
<用户目录>\.agents\plugins\marketplace.json
```

内容：

```json
{
  "name": "faustdream-local",
  "interface": {
    "displayName": "FaustDream Local Plugins"
  },
  "plugins": [
    {
      "name": "workflow",
      "source": {
        "source": "local",
        "path": "./plugins/workflow"
      },
      "policy": {
        "installation": "AVAILABLE",
        "authentication": "ON_INSTALL"
      },
      "category": "Productivity"
    }
  ]
}
```

### 配完后怎么生效

1. 执行同步脚本
2. 重启 Codex
3. 打开任意项目
4. 再试 `@workflow`

---

## 3. Codex 项目插件正确配置

如果你只希望插件在**当前仓库**里可见，应该配这一套。

### 目录结构

```text
<仓库根目录>
├─ .agents
│  └─ plugins
│     └─ marketplace.json
└─ plugins
   └─ workflow
      ├─ .codex-plugin
      │  └─ plugin.json
      └─ skills
```

### 项目 marketplace 最小示例

文件：

```text
<仓库根目录>\.agents\plugins\marketplace.json
```

内容：

```json
{
  "name": "faustdream-local",
  "interface": {
    "displayName": "FaustDream Local Plugins"
  },
  "plugins": [
    {
      "name": "workflow",
      "source": {
        "source": "local",
        "path": "./plugins/workflow"
      },
      "policy": {
        "installation": "AVAILABLE",
        "authentication": "ON_INSTALL"
      },
      "category": "Productivity"
    }
  ]
}
```

### 什么时候用项目配置

- 只想在一个仓库里试验插件
- 插件还没准备好全局复用
- 项目有专属 skill，不希望污染全局环境

---

## 4. CodeBuddy 正确配置

CodeBuddy 不读取 Codex 的 `.agents/plugins/marketplace.json`，也不读取 `.codex-plugin/plugin.json`。

它要走自己的本地 marketplace 结构。

### 目录结构

```text
<仓库根目录>
└─ codebuddy-marketplace
   ├─ .codebuddy-plugin
   │  └─ marketplace.json
   ├─ plugins
   │  └─ workflow
   │     ├─ .codebuddy-plugin
   │     │  └─ plugin.json
   │     └─ skills
   └─ scripts
      └─ sync-workflow.ps1
```

### 注册文件

```text
<用户目录>\.codebuddy\plugins\known_marketplaces.json
```

### 推荐做法

CodeBuddy 那一层只做包装，不直接维护 skills 内容。  
统一维护：

```text
<仓库根目录>\plugins\workflow\skills
```

同步到 CodeBuddy：

```powershell
pwsh.exe -File <仓库根目录>\codebuddy-marketplace\scripts\sync-workflow.ps1
```

然后保证 `known_marketplaces.json` 里有正确的本地 marketplace 注册项。

---

## 5. 推荐维护顺序

建议固定成下面这条链路：

1. 只维护 `<仓库根目录>\plugins\workflow`
2. Codex 项目版直接读取仓库里的 `plugins/workflow`
3. Codex 全局版通过 `sync-codex-home-workflow.ps1` 同步
4. CodeBuddy 版通过 `codebuddy-marketplace\scripts\sync-workflow.ps1` 同步
5. 改完后按需重启 Codex 或 CodeBuddy

这样做的好处是：

- 核心 skills 只维护一份
- 全局版、项目版、CodeBuddy 版都能共存
- 出问题时能快速判断是哪一层出了问题

---

## 6. 最小验证

### Codex 全局版

检查：

- `<用户目录>\.agents\plugins\marketplace.json`
- `<用户目录>\plugins\workflow\.codex-plugin\plugin.json`

然后重启 Codex，在别的项目里试 `@workflow`。

### Codex 项目版

检查：

- `<仓库根目录>\.agents\plugins\marketplace.json`
- `<仓库根目录>\plugins\workflow\.codex-plugin\plugin.json`

然后打开该仓库根目录，看插件市场是否出现 `workflow`。

### CodeBuddy

检查：

- `<仓库根目录>\codebuddy-marketplace\.codebuddy-plugin\marketplace.json`
- `<仓库根目录>\codebuddy-marketplace\plugins\workflow\.codebuddy-plugin\plugin.json`
- `<用户目录>\.codebuddy\plugins\known_marketplaces.json`

然后重启 CodeBuddy。
