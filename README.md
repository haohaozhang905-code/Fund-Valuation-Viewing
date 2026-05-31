# FinMate — 持仓看板 Web 版

一站式 A 股基金 + 美股持仓看板，纯前端单页应用，浏览器打开即用。实时估值、当日盈亏、累计收益一目了然，告别频繁登录多款券商/银行 App。

---

## 为什么需要这个项目？

> 个人持有 A 股基金 + 美股股票，每次想看一下今日收益，得分别打开支付宝/天天基金 + 富途/雪盈，每个都要登录、人脸验证、短信验证码。本项目的目标是：**打开浏览器，一眼看清所有资产**。

- 基金数据通过 ths-bridge 后端聚合天天基金估值 + 东方财富净值
- 美股数据通过 ths-bridge 后端聚合同花顺/Westock 等上游行情源
- 账号系统支持将持仓同步到云端，换设备无需重新录入

---

## 功能特性

### 🏠 持仓看板

- **资产摘要卡片**：基金资产总值、当日盈亏、累计盈亏 / 美股资产总值、当日盈亏、累计盈亏 / 持仓数量统计
- **基金持仓表格**：名称、代码、现价、今日涨跌、今日收益、累计收益、收益率
- **美股持仓表格**：名称、代码、现价、今日涨跌、持仓市值、累计收益、收益率
- **表格排序**：点击表头按任意列排序（升序/降序切换）
- **右键菜单**：桌面端右键行弹出详情/编辑/删除快捷操作
- **滑动详情面板**：点击行展开，查看基金净值走势图 / 美股 K 线图（含区间切换）

### 🔐 账号系统

- **邮箱注册**：邮箱 + 密码注册，自动登录
- **邮箱登录**：邮箱 + 密码登录，记住登录态（localStorage token）
- **找回密码**：邮箱验证码重置密码（支持两步流程）
- **退出登录**：清除本地 token，回到登录页

### ➕ 持仓管理

- **添加基金**：填写基金代码（6 位）、基金名称、成本价、份额
- **编辑基金**：修改已有基金的任意字段
- **删除基金**：面板内删除或右键删除
- **添加美股**：填写股票代码（symbol）、名称、成本价、份额
- **编辑/删除美股**：与基金操作一致
- **数据同步**：登录后持仓自动上传云端；登出时保留本地缓存不丢失

### 🔄 自动刷新

| 触发机制 | 基金 | 美股 |
|---|---|---|
| 当前 Tab 定时刷新 | 每 5 分钟 | 每 60 秒 |
| 页面切回前台 | 强制刷新 | 强制刷新 |
| 手动刷新 | 按 `R` 键刷新 | 按 `R` 键刷新 |

### 🤖 AI 分析面板

- 兼容 OpenAI 协议的 AI API 接入（如通义千问、DeepSeek）
- 可自定义 API 端点、模型名称、API Key
- 一键分析当前基金持仓组合，输出：组合概览、收益分析、风险提示、优化建议
- 配置保存在 localStorage，本地隐私可控

### ⚙️ 设置页面

- AI 配置：API Endpoint、Model、API Key
- 数据源配置（预留扩展）
- 配置持久化到 localStorage

### 📦 加载体验

- **骨架屏加载态**：登录后首次进入看板，展示资产卡片骨架 + spinner，避免闪现空表格
- **动态渐进渲染**：基金和美股数据并行加载，谁先到谁先展示，不互相阻塞
- **添加基金/美股即时响应**：保存后立刻在表格中插入占位行，后台刷新真实数据

### ⌨️ 键盘快捷键

| 快捷键 | 功能 |
|---|---|
| `N` | 添加当前 Tab 对应资产（基金/美股） |
| `R` | 手动刷新行情数据 |

---

## 技术架构

```
┌─────────────────────────────────────────┐
│              index.html                  │
│  (HTML + CSS + JS, 单文件 SPA)          │
│  - Hash 路由 (#login / #dashboard ...)  │
│  - 全局 state 对象驱动 render()         │
│  - Fetch API 调用后端 REST              │
└──────────────┬──────────────────────────┘
               │ HTTPS
┌──────────────▼──────────────────────────┐
│        ths-bridge (FastAPI 后端)        │
│  - 用户认证 / 持仓 CRUD                 │
│  - 天天基金 / 东方财富数据聚合          │
│  - 同花顺 / Westock 美股行情            │
│  - SQLite 存储                          │
└─────────────────────────────────────────┘
```

### 前端技术细节

| 层面 | 方案 |
|---|---|
| 框架 | 无框架，原生 HTML/CSS/JS |
| 路由 | `window.location.hash` 驱动 |
| 状态管理 | 全局 `state` 对象，`render()` 全量重绘 |
| UI 设计 | 暗色主题，CSS 变量驱动，PingFang SC 字体 |
| 网络 | `Fetch API`，并发批处理（concurrency=5） |
| 存储 | `localStorage`（token、持仓缓存、AI 配置） |
| 图表 | 内嵌 SVG 折线图 / K 线图（无第三方图表库） |
| 缓存策略 | 基金持仓/美股持仓本地缓存，登录后同步云端 |

### 后端依赖

本前端依赖 [ths-bridge](https://github.com/billzhang/Fund-Valuation-iOS) 后端服务，当前部署于 `https://thsbridge.zeabur.app`。

核心 API 端点：
- `POST /api/v1/auth/register` — 注册
- `POST /api/v1/auth/login` — 登录
- `GET /api/v1/auth/me` — 获取当前用户
- `POST /api/v1/auth/request-reset` — 请求重置密码
- `POST /api/v1/auth/confirm-reset` — 确认重置密码
- `GET/PUT /api/v1/portfolio` — 获取/更新持仓
- `GET /api/v1/funds/valuation/:code` — 基金估值
- `GET /api/v1/funds/nav-trend/:code` — 基金净值走势
- `GET /api/v1/funds/nav-latest/:code` — 基金最新净值
- `GET /api/v1/stocks/quote/:symbol` — 美股实时行情
- `GET /api/v1/stocks/search` — 美股搜索
- `GET /api/v1/stocks/kline/:symbol` — 美股 K 线
- `GET /api/v1/index/csi300` — 沪深 300 走势

---

## 快速开始

### 1. 直接打开

```bash
open index.html
```

### 2. 本地静态服务器

```bash
python3 -m http.server 8080
# 浏览器访问 http://localhost:8080
```

无需 `npm install`、无需构建步骤。首次打开为 **Demo 模式**，展示示例基金/美股数据，可体验基本看板功能。

### 3. 注册账号

点击右上角 **🔐 登录** → 注册账号 → 登录后添加自己的持仓 → 数据自动同步云端。

---

## 项目结构

```
Fund Valuation Viewing/
├── index.html    # 唯一源文件：HTML + CSS + JS 全内联
└── README.md     # 本文件
```

---

## 与 iOS App 的关系

本项目是 [Fund Valuation iOS](https://github.com/billzhang/Fund-Valuation-iOS) 的 Web 端实现，两者共享同一套 `ths-bridge` 后端 API：

| | Web 版 (本项目) | iOS 版 |
|---|---|---|
| 平台 | 浏览器（桌面/移动端） | iOS (SwiftUI) |
| 启动方式 | 打开 HTML 文件 | Xcode 构建安装 |
| 优势 | 零安装、跨平台、即开即用 | 原生体验、后台刷新、通知 |
| 技术栈 | HTML/CSS/JS | SwiftUI + Combine |

---

## 浏览器兼容性

需要支持以下特性的现代浏览器：
- CSS Variables (`var(--xxx)`)
- Fetch API
- `async/await`
- `Promise.all`
- Flexbox / Grid

测试通过：Chrome 90+ / Safari 15+ / Edge 90+ / Firefox 90+
