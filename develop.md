# AlphaAnalyst 开发文档
版本 1.0.0 | 技术架构与实现细节

## 1. 项目概览
AlphaAnalyst 是一个基于 Web 的 AI 金融分析平台，旨在为用户提供即时的市场洞察。项目采用 macOS 设计美学，完全运行在客户端（Client-side），利用 IndexedDB 进行本地数据持久化，并通过 Google Gemini API 获取实时的金融数据分析和预测。

## 2. 技术栈 (Tech Stack)
- **前端框架**: React 19 (ESM Modules via index.html)
- **样式库**: Tailwind CSS (CDN引入) + Lucide React 图标
- **AI 引擎**: Google GenAI SDK (@google/genai) - 模型: gemini-3-flash-preview
- **本地存储**: IndexedDB (原生 API 封装)

## 3. 核心架构设计

### 3.1 组件结构
- **App (Root)**: 管理全局状态（用户会话、模态框可见性、当前视图）。
- **DashboardView**: 主要业务视图，包含搜索栏、数据展示卡片和图表。
- **SimpleChart**: 自定义 SVG 图表组件，支持 K线数据、MA均线、布林带 (Bollinger Bands) 等技术指标绘制。
- **AuthModal / ProfileModal**: 处理用户认证、注册及个人信息更新。

### 3.2 数据流向
```
User Input (Ticker) -> Gemini API (Generate Content + Google Search Tool)
-> JSON Response Parsing -> State Update (Data)
-> IndexedDB (Save Search History) -> UI Render
```

### 3.3 账户隔离机制
为了确保多用户环境下的数据独立性，系统实现了以下机制：
- **组件重置 (Key Prop)**: 在 `DashboardView` 和模态框上绑定 `key={user.email}`。当用户切换时，React 会强制销毁并重新挂载组件，彻底清除内存中的临时状态（如输入框内容、图表状态）。
- **本地数据库**: 使用 `email` 作为 IndexedDB 的主键，确保每个用户的历史记录、头像和设置物理隔离。
- **会话管理**: 登录成功后将用户 Email 存入 localStorage，退出时清除，实现简单的持久化登录状态。

## 4. 数据存储 (IndexedDB)
使用 `LocalDB` 类封装了原生 IndexedDB 操作。

| Store Name | Key Path | 存储内容 |
|------------|----------|----------|
| users      | email    | 用户配置对象 (密码、头像Base64、搜索历史数组、创建时间) |

## 5. 安全说明
- **数据隐私**: 所有用户数据（包括密码）仅存储在用户浏览器的 IndexedDB 中，未上传至任何服务器。注意：这是一个演示应用，生产环境应使用后端数据库并对密码进行哈希处理。
- **API 密钥**: 通过 `process.env.API_KEY` 注入，避免在前端代码中硬编码。
- **输入验证**: 注册时强制执行密码复杂度检查（8位以上，包含字母、数字、特殊字符）。
