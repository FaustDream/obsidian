# 前端 UI 资源索引

> 按生态分类收录主流 UI 组件库与 CSS 动画库，含简要描述和官方链接。

---

## 一、React / Next.js 生态

| # | 库名 | 类型 | 描述 |
|---|------|------|------|
| 1 | **shadcn/ui** | 组件库（复制即用） | 基于 Radix UI + Tailwind CSS，非传统 npm 包，而是复制源码到项目中自由修改。视觉现代简洁，最推荐用于 SaaS / 后台 / Next.js 项目。MIT 开源。 |
| 2 | **MUI (Material UI)** | 组件库（npm 包） | Google Material Design 的 React 实现，组件最全（表格、日期选择、自动完成等），适合企业级后台系统。付费主题模板丰富。MIT 开源。 |
| 3 | **Ant Design** | 组件库（npm 包） | 蚂蚁金服出品的中后台 UI 库，Form/Table/DatePicker 功能极强。生态系统庞大（含图表、地图、可视化）。MIT 开源。 |
| 4 | **Arco Design React** | 组件库（npm 包） | 字节跳动开源的企业级 UI 库，60+ 组件，设计统一，适合审批、表单、表格类系统。支持主题变量定制。MIT 开源。 |
| 5 | **Mantine** | 组件库（npm 包） | 功能丰富、文档优秀，hooks 齐全（use-form、use-notifications 等）。内置暗色模式、日期、富文本等开箱即用。MIT 开源。 |
| 6 | **HeroUI (原 NextUI)** | 组件库（npm 包） | 颜值极高，支持 React + React Native 跨平台，beautiful by default。提供 Dashboard/Mail/Finance 等 Pro 模板。YC S24。MIT 开源。 |
| 7 | **Chakra UI** | 组件库（npm 包） | 强调可访问性（a11y），API 设计直观，主题系统灵活。适合中小型项目快速搭建。MIT 开源。 |
| 8 | **Radix UI** | 无样式原语（Primitives） | 提供弹窗、下拉、选择器等无样式无障碍组件，需自行搭配样式。shadcn/ui 底层即 Radix。MIT 开源。 |
| 9 | **React Bootstrap** | 组件库（npm 包） | Bootstrap 5 的 React 封装，栅格系统成熟，适合从 Bootstrap 迁移过来的项目。MIT 开源。 |
| 10 | **shadcnspace** | 组件/区块市场 | shadcn/ui 的第三方资源汇聚站。提供免费/付费的 Blocks、Templates、Dashboard、AI Tools，可直接复制使用。 |

---

## 二、Vue 3 生态

| # | 库名 | 类型 | 描述 |
|---|------|------|------|
| 1 | **Naive UI** | 组件库（npm 包） | Vue 3 首选推荐。视觉干净现代，TypeScript 支持优异，树摇优化，主题系统灵活，可直接用 `<n-theme-editor>` 可视化定制。MIT 开源。 |
| 2 | **Element Plus** | 组件库（npm 包） | Vue 3 版 Element UI，社区最大，表格/表单/弹窗功能成熟。适合中后台快速开发。MIT 开源。 |
| 3 | **Arco Design Vue** | 组件库（npm 包） | 字节 Arco Design 的 Vue 3 实现，60+ 组件，与 React 版共享设计语言。适合企业级系统。MIT 开源。 |
| 4 | **Vuetify** | 组件库（npm 包） | Material Design 的 Vue 实现，组件丰富，文档详尽，栅格系统和主题定制成熟。适合大型项目。MIT 开源。 |
| 5 | **PrimeVue** | 组件库（npm 包） | 组件覆盖极广（含图表、甘特图、组织图等），适合需要复杂控件的场景。MIT 开源（部分组件需 Pro 版）。 |

---

## 三、Tailwind CSS 生态

| # | 库名 | 类型 | 描述 |
|---|------|------|------|
| 1 | **daisyUI** | 组件库（Tailwind 插件） | Tailwind 上层的组件类库，无需手写大量 class，即用即取。多主题系统（light/dark/cupcake 等）。MIT 开源。 |
| 2 | **Flowbite** | 组件库（Tailwind 插件） | Tailwind 组件+交互（JS），包含导航、弹窗、轮播等。提供 Figma 设计稿。MIT 开源。 |
| 3 | **Preline UI** | 组件/区块库 | 640+ 免费组件、944+ UI Blocks、21 模板、27 个 Headless 插件。兼容 React/Vue/Next.js，支持 Figma。MIT 开源。 |
| 4 | **Headless UI** | 无样式原语 | Tailwind CSS 官方出品，提供无样式可访问组件（Listbox、Dialog、Popover 等），需自行搭配样式。MIT 开源。 |
| 5 | **Tailwind Variants** | 工具库 | 管理 Tailwind 组件变体的工具（类似 Class Variance Authority 但更灵活），适合封装可复用组件。MIT 开源。 |

---

## 四、CSS 动画库

| # | 库名 | 类型 | 描述 |
|---|------|------|------|
| 1 | **Animate.css** | 通用动画库 | 经典 CSS 动画库，通过添加 class 即可使用 fadeIn、bounce、slideIn 等动画。适合按钮、弹窗、卡片入场。MIT 开源。 |
| 2 | **Animista** | 在线动画生成器 | 可视化调参网站，选定动画类型和参数后生成 CSS 片段复制使用。适合定制单个动画效果。免费。 |
| 3 | **CSS Loaders** | 加载动画合集 | 600+ 纯 CSS 加载动画（单 `<div>` 实现），按风格分类（Dots / Spinners / Bars / 3D 等），点击一键复制 CSS。完全无依赖。免费。 |
| 4 | **Hover.css** | 悬停动画库 | 专为按钮、链接、图标、Logo 设计的 CSS3 hover 效果，含 2D 变换、背景过渡、阴影发光、气泡等类别。MIT / 商业双授权。 |
| 5 | **cssanimation.io** | 文字/通用动画库 | 500+ CSS 动画，按 40+ 类别组织（glitch、blob、orbit 3D、text 等），支持交互预览和复制。免费开源。 |
| 6 | **Magic Animations** | 夸张特效动画库 | 偏夸张视觉冲击的动画（puffIn、vanishOut、swap、twisterInDown），适合落地页、品牌展示等需要视觉亮点的场景。免费开源。 |
| 7 | **Three Dots** | 极简加载点动画 | 纯 CSS 三点加载动画，极端简约，适合 loading 状态的占位指示。免费。 |
| 8 | **AOS (Animate On Scroll)** | 滚动触发动画库 | 用 `data-aos` 属性控制 fade、flip、zoom 等滚动进场动画，适合官网、落地页、营销页。轻量（~2KB gzipped）。MIT 开源。 |
| 9 | **CSShake** | 抖动动画库 | 纯 CSS 抖动效果集合（shake、shake-slow、shake-hard、shake-rotate 等），可通过 CSS 变量定制。适合表单错误提示、按钮提醒、警告状态。MIT 开源。 |
| 10 | **SpinKit** | 加载动画合集 | 经典纯 CSS loading spinner 集合（rotating plane、bouncing dots、chasing circles 等），零 JS。适合加载状态。MIT 开源。 |
| 11 | **WickedCSS** | 通用动画库 | 效果比 Animate.css 更夸张有趣（barrelRoll、pulse、spinner 等），适合营销页、娱乐场景。免费开源。 |
| 12 | **Vivify** | 通用动画库 | 68 种 CSS3 动画（淡入淡出、旋转、缩放等），用法与 Animate.css 类似，轻量无依赖。免费开源。 |
| 13 | **CSS3 Animation Cheat Sheet** | 动画速查库 | Justin Aguilar 出品的即用 CSS3 keyframe 动画集（slideUp / fadeIn / bounce / pulse / hatch 等），单个 CSS 文件，支持滚动或点击触发。适合快速原型。免费。 |

---

## 五、选型速查

### 按技术栈推荐

| 场景 | 首选 | 备选 |
|------|------|------|
| React 中后台 | **shadcn/ui** | Mantine、Arco Design |
| React SaaS / 现代风 | **shadcn/ui** | HeroUI |
| React 企业级后台 | **Ant Design** | MUI、Arco Design |
| Vue 3 通用 | **Naive UI** | Element Plus |
| Vue 3 企业级后台 | **Element Plus** | Arco Design Vue |
| Tailwind 快速原型 | **daisyUI** | Flowbite、Preline UI |
| Tailwind 无样式 | **Headless UI** | Radix UI |

### 按动画需求推荐

| 场景 | 首选 | 备选 |
|------|------|------|
| 通用入场动画 | **Animate.css** | Vivify、WickedCSS |
| 滚动进场动画 | **AOS** | CSS3 Animation Cheat Sheet |
| 在线生成定制 | **Animista** | — |
| Loading / Spinner | **CSS Loaders** | SpinKit、Three Dots |
| Hover 效果 | **Hover.css** | — |
| 文字/标题动画 | **cssanimation.io** | — |
| 视觉冲击特效 | **Magic Animations** | WickedCSS |
| 抖动/提示/警告 | **CSShake** | — |
| 极简 loading 点 | **Three Dots** | SpinKit |
| 快速原型 / 即用 | **CSS3 Animation Cheat Sheet** | Animate.css |

### 入手优先级

> 更知名、优先看的前 6 个动画库：**AOS、CSShake、SpinKit、WickedCSS、Animate.css、CSS Loaders**

---

## 附：库类型说明

| 类型 | 说明 |
|------|------|
| 组件库（npm 包） | 通过 npm 安装，import 组件使用，一般含完整样式和交互 |
| 组件库（复制即用） | 将源码复制到项目中使用，可自由修改，非传统包管理方式 |
| 无样式原语（Primitives） | 只提供交互逻辑和可访问性，样式由开发者自由发挥 |
| 组件/区块市场 | 聚合第三方 Blocks / Templates 的资源站 |
| 通用动画库 | 提供一批预定义动画 class，直接添加到元素使用 |
| 在线动画生成器 | 网页端可视化调参，生成定制 CSS |
| 加载动画合集 | 专门提供 loading / spinner 类动画 |
| 滚动触发动画库 | 元素随滚动进入视口时触发动画 |
| 抖动动画库 | 专门提供 shake 类效果 |
| 动画速查库 | 即用的 keyframe 集合，复制 class 即可 |

