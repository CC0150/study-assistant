# Study Copilot 项目协作说明

本文件记录当前产品改造阶段需要遵守的约定，以及帮助开发者定位代码的基本信息。后端和前端的目录级协作说明见 `backend/AGENTS.md` 和 `frontend/AGENTS.md`；后端核心模块说明见对应目录下的 `AGENTS.md`。

## 当前产品目标

将 Study Copilot 逐步收敛为“学习资料问答系统”：用户管理自己的学习资料，针对资料提问，获得基于检索内容并附有来源依据的回答，也能查看问答记录。产品需求以 [`docs/product/学习资料问答系统-PRD-v1.1.md`](docs/product/学习资料问答系统-PRD-v1.1.md) 为准。

### 当前改造约定

- 一次只实现一个功能。交付时说明改动文件和验证结果，等待用户检验；用户明确同意前不提交该批次代码。
- 已确认下线的功能按用户验收节奏逐项移除代码；每次只处理一个功能点。不删除数据库表、迁移或用户数据，历史问答展示所需的兼容逻辑应保留。
- 当前产品主线是学习资料、资料库、常规资料问答、模型配置和任务管理。

### 已下线功能清单

以下功能当前不属于主产品流程。除“学习分析”专属代码已移除外，其余功能目前仅隐藏前端入口和页面；数据库结构和历史数据仍保留：

- 测验
- 学习分析（专属页面、API、服务和入口已移除）
- 课程与 AI 互动课堂
- 笔记
- 知识库（Wiki）
- 长期记忆管理
- 多角色讨论
- 深度研究模式
- 开始工作前阅读本文件及相关模块说明，并检查 `git status` 和差异。保护已有本地修改，只编辑当前任务所需文件。
- 不读取或输出 `.env`、`backend.env` 中的密钥和连接凭证；如需检查配置，只核对变量是否存在或是否配置，不回显值。
- 不删除用户文件或数据。启动开发服务时记录启动的端口，结束本次工作时只停止本次启动的服务，不误停用户已有进程。

## 项目架构

- **前端**：Vue 3、TypeScript、Vite、Pinia，位于 `frontend/`；默认开发端口为 3000。
- **后端**：FastAPI、Python、SQLAlchemy 2.0 异步模式，位于 `backend/`；默认开发端口为 8000。API 通过 HTTP 提供常规请求，通过 SSE 提供流式问答。
- **数据层**：PostgreSQL + pgvector；文件存储在项目配置指定的位置。仓库中也有历史 FAISS 检索实现，修改检索代码前先确认当前调用路径，避免把遗留实现当成生产主路径。
- **问答链路**：文档解析与分块、Embedding、向量检索、LLM 生成及来源返回。优先复用现有文档处理、RAG、认证和模型配置实现，避免重写架构。

## 常用代码位置

- 后端 API：`backend/app/api/`
- 后端业务服务：`backend/app/services/`
- 后端数据模型与数据库：`backend/app/models/`、`backend/app/database.py`
- 文档处理与检索：检查 `backend/app/` 下 document、embedding、vector store、RAG 相关模块及其实际引用
- 前端页面：`frontend/src/views/`
- 前端路由：`frontend/src/router/`
- 前端 API 与状态：`frontend/src/services/`、`frontend/src/stores/`
- 前端通用组件：`frontend/src/components/`
- 产品需求：`docs/product/学习资料问答系统-PRD-v1.1.md`

## 修改与验证

- 新增或修改 API 时，沿用现有路由、Pydantic 请求/响应模型、服务层和认证方式；检查 `backend/app/main.py` 中路由注册方式。
- 新增页面时，沿用现有 Vue 页面、路由、Pinia Store 和 API 封装方式。
- 修改前端时，按改动范围选择类型检查、构建或相关 Vitest 用例；修改后端时，选择相关 pytest 用例或 Ruff 检查。不要把未运行的检查描述为已通过。
- 不因文档中的旧测试数字推断当前测试总量；以当前仓库配置和实际命令输出为准。

### 交付格式

每次功能交付都按以下格式汇报；没有新增或删除文件时明确写“无”，未运行检查时说明原因：

1. **改动文件**：列出新增、修改、删除的文件。
2. **文件改动说明**：逐文件说明具体改了什么。
3. **验证结果**：列出实际运行的检查命令及输出摘要，不把未运行的检查写成通过。
4. **未覆盖项**：列出未运行或未覆盖的检查/场景及原因；没有则写“无”。
5. **用户检验步骤**：给出用户可以实际操作的检查步骤；不需要操作时说明原因。

```cmd
cd /d backend && pytest tests/ -v
cd /d frontend && npx vitest run
cd /d frontend && npm run typecheck
cd /d frontend && npm run build
```

## 项目文档

- 安装与快速开始：`docs/0-START-HERE/`、`docs/1-INSTALLATION/`
- 架构与 API：`docs/2-ARCHITECTURE/`、`docs/3-API-REFERENCE/`
- 开发与测试：`docs/4-DEVELOPMENT/`
