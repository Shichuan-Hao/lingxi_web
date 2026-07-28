# 灵犀 (Lingxi) — AI 对话平台

基于 Vue 3 + TypeScript + Vite 构建的 AI 对话前端应用。

## 功能模块

- **AI 对话** — 流式对话、深度思考、联网搜索、知识库问答（RAG）、多会话管理
- **电商客服** — 商品展示、智能客服聊天、图片上传

## 技术栈

| 类别 | 技术 |
|------|------|
| 框架 | Vue 3 (Composition API) |
| 语言 | TypeScript |
| 构建 | Vite 5 |
| 状态管理 | Pinia |
| 路由 | Vue Router 4 |
| UI | 纯 CSS（无第三方 UI 库） |

## 快速开始

```bash
# 安装依赖
npm install

# 启动开发服务器
npm run dev

# 构建生产版本
npm run build

# 预览生产构建
npm run preview
```

## 项目结构

```
lingxi_web/
├── src/
│   ├── views/          # 页面组件（Home、EcommerceService）
│   ├── api/            # Axios 接口层
│   ├── services/       # 业务服务层（SSE 流式对话）
│   ├── stores/         # Pinia 状态管理
│   ├── router/         # Vue Router 路由配置
│   └── components/     # 公共组件
├── index.html
├── vite.config.ts
└── package.json
```

## 环境要求

- Node.js >= 18
- 后端服务：[lingxi_backend](https://github.com/Shichuan-Hao/lingxi_backend)
