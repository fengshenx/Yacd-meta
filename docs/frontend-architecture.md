# YACD 前端页面架构文档

## 1. 技术栈

| 类别 | 技术 |
|------|------|
| 框架 | React 18 + TypeScript |
| 路由 | React Router v6 (HashRouter) |
| 状态管理 | Recoil + 自定义 StateProvider (类 Redux) |
| 样式方案 | SCSS Modules (`.module.scss`) |
| 主题系统 | CSS 自定义属性 (CSS Variables)，通过 `data-theme` 属性切换 |
| 字体 | Inter (正文)、Roboto Mono (等宽) |
| 图标 | react-feather + react-icons/fc |
| 构建工具 | Vite |
| 国际化 | react-i18next |

---

## 2. 应用外壳 (App Shell)

```
┌─────────────────────────────────────────────────┐
│  App (src/App.tsx)                               │
│  └─ AppProviders (Recoil + QueryClient + I18n)   │
│     └─ AppRouter (src/app/router.tsx)            │
│        ├─ Head (document title)                  │
│        └─ HashRouter                             │
│           ├─ /backend → BackendPage (独立全屏)    │
│           └─ /* → DashboardRouter                │
│              ├─ APIDiscovery                     │
│              ├─ SideBar (左侧/底部导航)           │
│              └─ div.content (主内容区, 懒加载)     │
└─────────────────────────────────────────────────┘
```

**布局模式**：
- 桌面端：`flex` 水平排列，SideBar 在左，content 在右
- 移动端 (≤768px)：`flex-direction: column`，SideBar 固定在顶部，content 下方带 `padding-top: 64px`

**文件**：
- `src/App.tsx` — 根组件，包裹 Providers
- `src/app/router.tsx` — 路由定义 + DashboardRouter 布局
- `src/App.module.scss` — `.app` (全屏 flex 容器) + `.content` (可滚动内容区)

---

## 3. 导航栏 (SideBar)

**文件**：`src/components/SideBar.tsx` + `SideBar.module.scss`

**结构**：
```
SideBar (.root)
├── Logo (.logo_meta / .logo_singbox)
├── Rows (.rows)
│   └── SideBarRow × 6 (.row / .rowActive)
│       ├── Icon (react-icons/fc 彩色图标)
│       └── Label (.label)
├── Footer (.footer)
│   ├── ThemeSwitcher
│   └── About Link (Info icon)
```

**导航项**：

| 路径 | 图标 | 标签 |
|------|------|------|
| `/` | FcAreaChart | Overview |
| `/proxies` | FcGlobe | Proxies |
| `/rules` | FcRuler | Rules |
| `/connections` | FcLink | Conns |
| `/logs` | FcDocument | Logs |
| `/configs` | FcSettings | Config |

**响应式**：桌面端为左侧垂直栏，移动端为顶部水平栏 (固定定位)。

---

## 4. 页面组件

### 4.1 Overview (`/`)

**文件**：`src/components/Home.tsx` + `Home.module.scss`

```
Home
├── ContentHeader (空标题)
└── TrafficNow (.TrafficNow)
    ├── Memory Usage (Sparkline 图表)
    ├── Download (Sparkline 图表)
    ├── Upload (Sparkline 图表)
    ├── Download Total
    ├── Upload Total
    └── Active Connections
```

**布局**：CSS Grid `repeat(auto-fill, minmax(240px, 1fr))`，每个 `.sec` 卡片包含 header/value/chart。

---

### 4.2 Proxies (`/proxies`)

**文件**：`src/components/proxies/Proxies.tsx` + `Proxies.module.scss`

```
Proxies
├── Settings Modal (BaseModal)
├── TopBar (.topBar, sticky, 毛玻璃背景)
│   └── ContentHeader
│       ├── TabsContainer (.tabsContainer)
│       │   ├── Tab: Proxies (+ count badge)
│       │   └── Tab: Providers (+ count badge, 条件渲染)
│       ├── Spacer (flex: 1)
│       └── TopBarRight
│           ├── TextFilter (搜索框)
│           └── Settings Button (Equalizer icon)
├── Content (.groupsContainer)
│   └── ProxyGroup × N (.group)
│       ├── GroupHeader (名称 + 操作按钮)
│       ├── AvailBar (可用率进度条)
│       ├── NowRow (当前选中节点)
│       └── ProxyList (Collapsible)
│           └── Proxy × N (.proxy 卡片)
│               ├── ProxyName
│               ├── ProxyType + UdpType
│               └── ProxyLatency
├── ProxyPageFab (浮动操作按钮)
└── ClosePrevConns Modal
```

**关键样式**：
- Tab 切换：pill 形状容 (`border-radius: 12px`)，激活态蓝色背景
- Proxy 卡片：`border-radius: var(--border-radius)` (12px)，hover 上浮 + 阴影
- 双列布局：`proxiesLayout === 'double'` 时在 ≥1200px 启用 flex 双列

---

### 4.3 Rules (`/rules`)

**文件**：`src/components/Rules.tsx` + `Rules.module.scss`

```
Rules (.container)
├── ContentHeader
│   ├── TabsContainer (Rules / Providers 切换)
│   ├── Spacer
│   └── TextFilter
├── ListWrapper (.listWrapper, 虚拟滚动)
│   └── VariableSizeList (react-window)
│       ├── Rule × N
│       └── RuleProviderItem × N
└── RulesPageFab
```

**特点**：使用 `react-window` 虚拟列表处理大量规则，Tab 样式与 Proxies 页一致。

---

### 4.4 Connections (`/connections`)

**文件**：`src/components/Connections.tsx` + `Connections.module.scss`

```
Connections
└── Tabs (react-tabs)
    ├── ContentHeader
    │   └── Controls (.controls)
    │       ├── TabGroup
    │       │   ├── TabList (Active / Closed + count)
    │       │   └── Source IP Select
    │       ├── Spacer
    │       ├── Input (搜索过滤)
    │       └── Toolbar (.toolbar)
    │           ├── Close All / Close Filtered
    │           ├── Divider
    │           └── Settings / Reset / Tag
    ├── ContentWrapper (.contentWrapper, 圆角卡片容器)
    │   ├── TabPanel: Active → ConnectionTable
    │   └── TabPanel: Closed → ConnectionTable
    ├── Fab (暂停/恢复刷新)
    └── Modals (关闭确认 / 列管理 / Source IP)
```

**特点**：使用 `react-tabs` 库，移动端 controls 切换为 grid 布局。

---

### 4.5 Logs (`/logs`)

**文件**：`src/components/Logs.tsx` + `Logs.module.scss`

```
Logs
├── ContentHeader
│   └── HeaderControls
│       ├── LogSearch (搜索 + 日志级别选择)
│       └── Clear Button (Trash2 icon)
├── LogsWrapper (.logsWrapper, 可滚动)
│   ├── LogLine × N
│   │   ├── LogMeta (时间 + 类型标签)
│   │   └── LogText (日志内容)
│   └── Placeholder (无日志时显示 SvgYacd)
├── ScrollToBottom Button (条件显示)
└── Fab (暂停/恢复)
```

**特点**：日志类型标签按 `data-type` 属性着色 (debug=绿, info=蓝, warning=黄, error=红)。

---

### 4.6 Config (`/configs`)

**文件**：`src/components/Config.tsx` + `Config.module.scss`

```
Config
├── ContentHeader
└── Root (.root, max-width: 1000px, 居中)
    ├── Card: General (.card)
    │   ├── SectionTitle (Settings icon + "General")
    │   └── Section (.section, grid 布局)
    │       ├── Port Fields (Input × N)
    │       ├── Mode (Select)
    │       ├── Log Level (Select)
    │       ├── Allow LAN (Switch)
    │       └── TLS Sniffing (Switch)
    ├── Card: TUN (条件渲染)
    ├── Card: Management
    │   └── Reload / Upgrade / Restart Buttons
    └── Card: Dashboard
        ├── Latency Test URL (Input)
        ├── Language (Select)
        ├── Chart Style (Selection2)
        └── Switch Backend (Button)
```

**布局**：卡片式布局，每个 `.card` 有圆角 12px + 阴影 + hover 边框高亮。Section 内部使用 `grid-template-columns: repeat(auto-fill, minmax(320px, 1fr))`。

---

### 4.7 About (`/about`)

**文件**：`src/components/about/About.tsx` + `About.module.scss`

简单的版本信息展示页，包含 Core 版本和 Yacd 版本，各带 GitHub 链接。

---

## 5. 共享组件

| 组件 | 文件 | 说明 |
|------|------|------|
| ContentHeader | `ContentHeader.tsx` | 页面顶部栏，高度 60px，包含标题和子元素 |
| Button | `Button.tsx` | 胶囊按钮 (`border-radius: 100px`)，支持 `minimal` 变体 |
| Input | `Input.tsx` | 输入框，圆角 8px，focus 蓝色光晕 |
| Select | `shared/Select.tsx` | 下拉选择器 |
| BaseModal | `shared/BaseModal.tsx` | 模态框基础组件，半透明遮罩 + 居中内容 |
| TextFilter | `shared/TextFitler.tsx` | 搜索过滤输入框 (Recoil atom 绑定) |
| ThemeSwitcher | `shared/ThemeSwitcher.tsx` | 主题切换器 (auto/light/dark) |
| Fab | `shared/Fab.tsx` | 浮动操作按钮，固定右下角 |
| Collapsible | `Collapsible.tsx` | 可折叠容器 |
| Sparkline | `Sparkline.tsx` | 迷你折线图 |
| SwitchThemed | `SwitchThemed.tsx` | 主题化开关组件 |

---

## 6. 样式系统

### 6.1 主题变量 (CSS Custom Properties)

定义在 `src/styles/main.scss`，通过 SCSS mixin `@mixin dark` / `@mixin light` 生成，挂载在 `:root[data-theme]` 上。

**核心变量**：

| 变量 | Light | Dark | 用途 |
|------|-------|------|------|
| `--color-background` | `#eef2f6` | `#111827` | 页面主背景 |
| `--color-bg-card` | `#f8fafc` | `#1f2937` | 卡片背景 |
| `--color-bg-sidebar` | `#f1f5f9` | `#1f2937` | 侧边栏/Tab 容器背景 |
| `--color-text` | `#334155` | `#e5e7eb` | 主文本色 |
| `--color-text-secondary` | `#64748b` | `#9ca3af` | 次要文本色 |
| `--color-text-highlight` | `#0f172a` | `#f3f4f6` | 高亮文本色 |
| `--color-focus-blue` | `#3b82f6` | `#3b82f6` | 主交互色/焦点色 |
| `--color-separator` | `#e2e8f0` | `#374151` | 分割线 |
| `--color-input-bg` | `#f8fafc` | `#374151` | 输入框背景 |
| `--color-input-border` | `#cbd5e1` | `#4b5563` | 输入框边框 |
| `--color-btn-bg` | `#f8fafc` | `#374151` | 按钮背景 |
| `--color-btn-fg` | `#334155` | `#e5e7eb` | 按钮文字 |
| `--shadow-card` | 轻阴影 | 轻阴影 | 卡片投影 |
| `--border-radius` | `12px` | `12px` | 全局圆角 |

### 6.2 字体

```scss
--font-normal: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif, ...;
--font-mono: 'Roboto Mono', Menlo, monospace;
```

### 6.3 断点

定义在 `src/styles/utils/custom-media.scss`，主要断点：
- `768px` — 移动端/桌面端分界
- `1200px` — 双列布局触发点
- `--breakpoint-not-small` — 自定义媒体查询 (≥30em)

### 6.4 通用样式模式

- **卡片**：`border-radius: 12px` + `box-shadow` + `border: 1px solid var(--color-separator)` + hover 上浮
- **Tab 切换**：pill 容器 (`border-radius: 12px`, 背景色 `--color-bg-sidebar`)，激活项蓝色填充
- **按钮**：胶囊形 (`border-radius: 100px`)，hover 变蓝 + 上浮
- **输入框**：`border-radius: 8px`，focus 蓝色光晕 (`box-shadow: 0 0 0 3px rgba(59,130,246,0.3)`)
- **Sticky 顶栏**：`backdrop-filter: blur(36px)` 毛玻璃效果

---

## 7. 页面路由映射

| 路径 | 懒加载页面 | 核心组件 |
|------|-----------|---------|
| `/` | — | `Navigate → /proxies` |
| `/home` | `HomePage` | `Home` → `TrafficNow` |
| `/proxies` | `ProxiesPage` | `Proxies` → `ProxyGroup` → `Proxy` |
| `/rules` | `RulesPage` | `Rules` → `Rule` / `RuleProviderItem` |
| `/connections` | `ConnectionsPage` | `Connections` → `ConnectionTable` |
| `/logs` | `LogsPage` | `Logs` → `LogLine` |
| `/configs` | `ConfigPage` | `Config` |
| `/about` | `AboutPage` | `About` |
| `/backend` | `BackendPage` | `BackendList` (独立页面，无 SideBar) |
| `/style` | `StyleGuidePage` | `StyleGuide` (仅开发环境) |

---

## 8. 文件目录结构

```
src/
├── api/                    # Clash REST API 封装
├── app/
│   ├── router.tsx          # 路由 + DashboardRouter 布局
│   ├── providers.tsx       # Recoil + QueryClient + I18n
│   ├── bootstrap.ts        # 应用初始化
│   └── APIDiscovery.tsx    # API 自动发现
├── components/
│   ├── SideBar.tsx         # 导航栏
│   ├── ContentHeader.tsx   # 页面标题栏
│   ├── Button.tsx          # 通用按钮
│   ├── Input.tsx           # 通用输入框
│   ├── Home.tsx            # Overview 页
│   ├── TrafficNow.tsx      # 流量/内存卡片
│   ├── Connections.tsx     # 连接页
│   ├── ConnectionTable.tsx # 连接表格
│   ├── Logs.tsx            # 日志页
│   ├── Config.tsx          # 配置页
│   ├── Rules.tsx           # 规则页
│   ├── Modal.tsx           # 通用模态框
│   ├── proxies/            # 代理相关组件
│   │   ├── Proxies.tsx     # 代理主页面
│   │   ├── Proxy.tsx       # 单个代理卡片
│   │   ├── ProxyGroup.tsx  # 代理分组
│   │   ├── ProxyList.tsx   # 代理列表
│   │   └── ...
│   ├── about/              # 关于页
│   ├── rules/              # 规则子组件
│   ├── shared/             # 共享基础组件
│   │   ├── BaseModal.tsx
│   │   ├── Fab.tsx
│   │   ├── Select.tsx
│   │   ├── TextFitler.tsx
│   │   └── ThemeSwitcher.tsx
│   └── svg/                # SVG 图标组件
├── hooks/                  # 自定义 Hooks
├── modules/                # 业务逻辑模块 (hooks + utils)
├── pages/                  # 页面入口 (懒加载包装)
├── store/                  # Recoil atoms + 自定义 store
├── styles/
│   ├── main.scss           # 全局样式 + 主题变量
│   └── utils/
│       └── custom-media.scss  # 响应式断点
└── main.tsx                # 应用入口
```
