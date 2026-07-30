# STAR_Assistant
AI-energized material learning assistant
# STAR 学习助手

> 面向《机器学习预测材料性质》课程的 AI 辅助学习智能体
>
> 一句话描述：**越来越懂你的 AI 课堂助教** —— 基于课程私有知识、融合知识图谱与大模型对话的个性化学习智能体

---

## 1. 项目简介（Project Overview）

- **项目定位**：单课垂直领域的 AI 辅助学习智能体，不是通用聊天机器人。严格基于教师上传的私有课件、讲稿、视频进行教学，杜绝"幻觉"。
- **项目名称寓意**：STAR = **S**mart **T**utoring with **A**daptive **R**easoning（自适应推理的智能辅导），也寓意"星光引路"——为每位学生点亮个性化学习的导航星。
- **目标用户**：材料学科学生 & 青年老师。学生获得"越来越懂自己"的 AI 助教；老师获得全班学情概览与共性薄弱点分析。
- **合作单位**：指南针学院 × 深圳理工大学（暑期合作项目）。
- **核心愿景**：让 AI 不只"回答问题"，更能像一位真正的助教——依据课堂资料（而非凭空而论）、了解学生学习习惯、规划学习路径、在互动中越用越懂你。
- **部署形态**：全班共用一个链接，学生凭学习码登录；管理员（老师）管理课程资料。已在阿里云 Ubuntu 服务器跑通。

---

## 2. 项目背景与痛点（Background & Motivation）

### 核心矛盾：材料+X 交叉学科的教与学困境

- **知识跨度大，基础差异显著**：《机器学习预测材料性质》横跨材料科学与机器学习两大领域。选课学生背景多元（材料/化学/物理），编程与 ML 基础参差不齐，传统"统一进度"课堂难以兼顾。
- **课后缺乏即时辅导**：青年老师资源有限，无法为每位学生提供一对一答疑。学生在课后遇到问题时往往只能依赖搜索引擎或通用 AI——后者不懂课程具体内容，常给出似是而非的回答。
- **通用大模型的"幻觉"问题**：ChatGPT 等通用助手在专业课程中缺乏课程私有知识（特定课件、讲稿、讲义），答非所问甚至编造概念，无法承担"课程助教"职责。
- **现有在线学习平台交互割裂**：视频观看（被动接收）与答疑（主动求助）分离；学习平台不知道"你是谁、会什么、哪里薄弱"——缺乏个性化。
- **老师对学生学情"看不见"**：传统课堂只能通过考试了解学生，缺乏对全班认知共性与薄弱点的实时把握。

### 我们的回答：STAR 学习助手

一个部署在服务器上、全班共用一个链接的 AI 课堂助教。它读取老师上传的真实课件来回答学生问题（杜绝幻觉）；维护每个学生自己的三维画像（越用越懂你）；为每次学习设计个性化学习流；并帮老师看到全班的认知共性与薄弱点。

---

## 3. 核心功能（Key Features）

### 3.1 多用户与角色系统

- **学习码 + 姓名登录（学生）**：无需注册、无需密码，老师分发学习码即可。
- **管理员密码登录（教师）**：管理全部课程资料与学生信息。
- **角色权限严格分离**：学生只能学习/看图谱/管自己的画像；管理员才能建课/上传资料/建 RAG 索引。后端所有写接口用 `Depends(require_admin)` 守卫。
- **数据按 user_id 隔离**：每个学生的画像、学习会话、模型配置存放在独立目录（`data/users/{uid}/`），互不可见；课程、材料、视频、RAG 索引全班共享。
- **httpOnly 签名 Cookie 会话**：防 XSS 窃取，30 天有效期，生产环境支持 secure 标志。

### 3.2 智能选课与目标设定

- 管理员将课程组织为「课程 → 课时（节）」结构，为每课时上传讲稿/课件/资料与视频。
- 学生在「学习」页单选某一课时，可补充学习意图并标注熟悉程度（0–4 五级）。
- **课堂概要自动注入**：若该课时有讲稿，系统已自动生成详细概要（核心要点、关键知识点、学习顺序），AI 据此设计更贴合本节内容的学习清单。

### 3.3 多维基础评估（首次画像构建）

- **学习目标识别**：想学什么？为什么学？当下要解决什么问题？
- **基础摸底**：专业方向、编程基础、ML 基础自评。
- **知识点 0–4 分掌握度量表**：不依赖考试，以学生自我认知为起点。
- 评估后即刻生成三份画像（`USER.md` / `knowledge.md` / `study.md`），为后续个性化学习奠基。

### 3.4 可视化知识图谱导航

- **课程知识点拓扑网络**：节点 + 边（先修/相关/归属关系）。
- **五级掌握度色彩映射**（灰→绿→蓝绿→蓝→紫），从"不了解"到"熟知原理"一目了然。
- **学习后图谱动态生长**——知识被细化、新节点加入、掌握度颜色更新。
- 图谱是 `knowledge.md` 的可视化呈现，而非独立系统——画像即图谱的数据源。

### 3.5 互动式 AI 对话辅导（七步学习闭环）

这是 STAR 的核心学习引擎。每次学习遵循以下流程：

1. **画像注入**：每次 LLM 调用前，自动读取学生的三份 `.md` 画像 + 历史对话，拼入 Prompt——让 AI "认识"这个学生。
2. **学习规划（Plan）**：AI 根据课时材料 + 课堂概要 + 学生画像，生成一份 4–7 步的交互学习清单（`step_plan`），每步标注使用 RAG 还是直接讲解。
3. **情景带入（Scenario）**：通过开放性提问让学生主动进入思考状态，而非被动接收。
4. **知识讲解（Explain）**：将核心内容拆成小知识点逐步呈现；涉及材料时自动用 RAG 检索课件原文——若来源是课件 PDF，句末标注可点击的出处编号（如 ①②）。
5. **练习反馈（Practice）**：提供即时习题检验掌握程度，帮助发现盲区。
6. **视频推荐（Video）**：当本课时有视频时，AI 在合适位置据当前知识点与视频的「讲解角度/重点/时长」智能挑选最匹配的 1–3 个，页面内 `<video>` 直接播放。
7. **画像更新**：清单走完后「下一步」变为「完成学习」——由学生决定何时结束。点击后 AI 综合全程表现更新三份画像（`knowledge` 取最高掌握度不降级）。

全流程可随时追问、质疑、展开——学生不是被动的"下一步"点击者。

### 3.6 PDF 课件出处标注（可追溯的 AI）

- 当 AI 回答引用了课件 PDF（`role=slides`），句末自动标注可点击的编号引文 ①②…
- 点击编号 → 右侧抽屉面板从右滑入（framer-motion 动画），正文自动左移不被遮挡。
- 面板内嵌 PDF iframe，自动跳转到被引用的那一页。
- 讲稿/资料作为 RAG 上下文进入但不标引文——只有课件出处被编号，保证引文精确可追溯。
- 刷新页面后历史回答中的引文编号仍保留。
- 这是"课堂资料驱动、而非凭空而论"的落地证据——让 AI 说的每句话都能追到课件第几页。

### 3.7 视频推荐与评分反馈闭环

- 教师上传视频时填写名称、讲解角度、讲解重点、时长——为智能推荐提供元数据。
- 学习过程中 AI 根据当前知识点 + 视频元数据 + 历史评分，挑选最匹配的 1–3 个视频。
- 学生看完后打 1–5 分（反馈帮助程度），一人一票、可修改。
- **差评视频自动过滤**：累计 ≥3 人打分且均分 < 2.5 的视频不再推荐（除非该课时仅此一个视频，则回退兜底）。
- 教师端视频名旁显示全班均分 ★4.2(12)，一目了然哪些视频最受欢迎。
- 这是典型的 AI 反馈循环——推荐质量随使用不断优化，越用越准。

### 3.8 课堂概要（讲稿驱动，自动生成）

- 教师上传"讲稿"类型材料时，系统首次自动调用 LLM 生成详细课堂概要。
- 概要内容：核心要点、解决的问题、关键知识点、重点难点、建议学习顺序等（Markdown 格式）。
- 支持手动重新生成 / 导入（.md）/ 导出（.md）。
- 学生学习该课时时，概要注入学习规划 Prompt——AI 据此设计更详细、更贴合本节要点的学习清单。
- 概要缺失时自动降级走原逻辑，不报错。

### 3.9 班级学情总览（教师端）

- **学生管理面板**：列表展示全班学生姓名、知识点数量、平均掌握度、学习会话数、最近活跃时间。
- 点击学生可查看其完整三份画像（`USER.md` / `knowledge.md` / `study.md`）。
- 一键「生成班级共性报告」：LLM 聚合全班 `USER.md`（截断 ~600 字/人）和薄弱知识点（level ≤ 1），输出认知共性、共性薄弱点、学习习惯共性、教学建议——帮助老师把握班级全局、调整教学重点。

### 3.10 课程管理（管理员）

- **课程 CRUD**：新建/重命名/删除课程与课时；删除级联清理材料、视频、课堂概要。
- **材料上传**：支持 PDF/PPTX/DOCX 等格式，上传时标注角色（讲稿/课件/资料）。
- **视频上传**：本地 mp4/webm 文件，管理员填写名称/角度/重点/时长（或点「重新分析」让 AI 据标题生成草稿）。
- **RAG 索引构建**：一键为材料建立 FAISS 向量索引（sentence-transformers 本地嵌入）。
- **模型配置**：学生在「我的」页可自配 API 地址/密钥/模型名；未自配时使用系统预配的课堂共用 DeepSeek key，登录即用。

---

## 4. 核心设计（Core Design）

### 4.1 用户画像建模：`USER.md` / `knowledge.md` / `study.md`

仿照 Claude Code 的记忆机制，STAR 为每个学生维护三份 Markdown 画像文件。Markdown 格式兼顾人类可读与 LLM 可解析：

- **`USER.md`** —— 认知水平、学习习惯、思维模式（叙述式）。
  > 例："该生倾向于先理解直观含义再接触公式；在面对抽象概念时偏好类比解释。"
- **`knowledge.md`** —— 知识账本（知识图谱的唯一数据源）。固定 `knowledge.json` 代码块内含结构化 JSON：节点（id/title/level/parent/tags）+ 边（source/target/rel），掌握度 0–4 五级。LLM 只维护块内 JSON，其余部分自由叙述。
- **`study.md`** —— 适合该生的学习流构建策略（叙述式）。
  > 例："对该生应减少一次性知识灌输，增加情景带入步骤；练习以选择题为主。"

画像随学习进程持续演化：每次"完成学习"触发 summarizer 更新——`knowledge` 走 `safe_update`（diff，level 取 max 不降），`USER`/`study` 整篇重写。更新前自动备份原文件。

### 4.2 知识图谱引擎

- **数据源**：`knowledge.md` 的 JSON 块（非独立图数据库）。
- **构建**：后端 `graph_builder` 解析 `knowledge.md` → 提取节点/边 → 按 `user_id` 内存缓存（`dict[uid]`）→ 返回 Graph 对象。
- **社区结构**：沿 `parent` 字段回溯到根知识点，自动聚类形成社区（用于图谱可视化布局）。
- **前端渲染**：`@react-sigma/core` + `graphology`，力导向布局（ForceAtlas2），节点颜色由 level 映射（两端一致的 `MASTERY_COLORS` 常量）。
- 画像更新后调用 `reload_graph(uid)` 失效缓存，图谱同步刷新。

### 4.3 上下文感知对话

- 每次 LLM 调用前，`agent/context.py` 按 uid 读取三份 `.md` 画像 + 历史对话摘要，拼入 Prompt。
- **画像注入按场景区分**：plan 步注入 USER + knowledge + study + 课堂概要；explain/followup 步注入 USER + knowledge（不注入 study，避免干扰讲解纪律）。
- **Prompt 模板化**：`prompts/*.txt` 用 `{var}` 占位，结构化输出末尾标注"只输出 JSON"。
- **SSE 流式返回**：token 级实时推送，前端逐字渲染 Markdown + KaTeX 公式。

### 4.4 RAG 检索增强生成（已完整实现）

- **提取层（extractor）**：`pdfplumber`（PDF，按页存储 `page` 字段）/ `python-pptx` / `python-docx`，统一输出文本 + 页码。
- **分块层（chunker）**：固定大小切分，透传 `page` 字段。
- **嵌入层（embedder）**：`sentence-transformers` 本地模型（lazy 加载 torch，启动不占显存），通过 HF 镜像下载。
- **索引层（index_manager）**：FAISS 每材料一索引，存于 `data/rag_index/{material_id}/`，搜索返回 `material_id` + `page`。
- **检索层（retriever）**：检索结果经 `build_rag_context` 过滤——仅 `role=slides` 课件分配引文编号 `[1..n]`；讲稿/资料作无编号上下文。
- **不是"可扩展 RAG 接口"——是已完整实现的生产级 RAG 管线**，PDF/PPTX/DOCX 全链路打通。

### 4.5 多用户数据隔离

- **隔离边界**：画像/会话/模型配置按 uid 分目录（`data/users/{uid}/`）；课程/材料/视频/RAG 索引全班共享。
- **显式 uid 透传（不用 contextvars）**：SSE 流式 `gen()` 内 Starlette 中间件 + `StreamingResponse` 对 `contextvar` 跨 task 传播不可靠，改用 `session.user_id` 显式传入——避免多用户并发时数据串扰（教学产品不可接受）。
- **会话越权保护**：`get_session(id, uid)` 先按用户目录读，再校验 `user_id == uid`，否则 403。

---

## 5. 技术架构（Architecture）

> 建议在此处放置整体架构图，推荐使用 Excalidraw / Draw.io 绘制。以下为文字版架构描述，可作为绘图参考。

### 【客户端层】

- 浏览器（React 19 SPA）：登录 → 知识图谱首页 → 学习选择 → 互动对话（SSE 流式）→ 视频播放 → PDF 引文面板。
- Tailwind CSS 4 + framer-motion：页面切换过渡动画（横向滑动）、PDF 抽屉滑入/滑出。

### 【接入层】

- Uvicorn（ASGI）：单 worker 起服务，`--host 0.0.0.0 --port 8000`。
- 中间件链：CORS → `auth_guard`（`/api/material` 鉴权）→ `SessionMiddleware`（itsdangerous Cookie）。
- StaticFiles：前端 `dist/` 单页应用 + `/api/material` 课程文件/视频（同源自动带 httpOnly Cookie）。

### 【业务层（FastAPI 路由）】

- **认证路由（auth）**：学习码/管理员密码校验 + Cookie 签发与验证。
- **画像路由（onboarding / profile_io）**：评估 + 三 `.md` 导入/导出/替换。
- **学习路由（session）**：SSE 流式 `step` / `token` / `practice` / `video` / `rag_context` / `done`，`start` / `end` / `followup`。
- **图谱路由（graph）**：按 uid 返回知识图谱数据。
- **课程路由（course / material / video）**：管理员 CRUD 课程/课时/材料/视频 + RAG 索引。
- **课堂概要路由（lesson_summary）**：生成/重新生成/导入/导出。
- **管理路由（admin）**：学生列表 + 个人画像 + 班级共性报告（LLM 聚合）。

### 【AI 引擎层】

- **LLM 客户端（`llm/llm_client.py`）**：OpenAI 兼容 SDK，默认 DeepSeek（预配课堂共用 key），学生可自配覆盖。
- **Agent 模块（agent/）**：`onboarding`（评估）→ `planner`（规划）→ `steps`（执行：情景/讲解/练习/视频）→ `grader`（判题）→ `summarizer`（结课更新画像）→ `graph_builder`（图谱构建）→ `memory_update`（画像更新）→ `class_report`（班级报告）→ `lesson_summary`（课堂概要）。
- **Prompt 模板（`prompts/*.txt`）**：`{var}` 占位，结构化输出末尾"只输出 JSON"。
- **视频智能挑选（`agent/video.py` + `prompts/video_pick.txt`）**：据当前知识点 + 视频角度/重点/时长 + 历史评分，挑最匹配 1–3 个。

### 【RAG 引擎层】

`extractor` → `chunker` → `embedder` → `index_manager` → `retriever`（详见 4.4 节）。

### 【数据层】

- **JSON 文件存储（`store/json_store.py`）**：`courses.json` / `lessons.json` / `materials_meta.json` / `videos.json` / `lesson_summaries.json` / `video_ratings.json`。
- **Markdown 文件存储**：`data/users/{uid}/memory/{USER,knowledge,study}.md`。
- **FAISS 索引**：`data/rag_index/{material_id}/`。
- **会话文件**：`data/users/{uid}/sessions/{session_id}.json`。
- **模型配置**：`data/users/{uid}/model_config.json`。

---

## 6. 技术栈（Tech Stack）

| 层次 | 技术选型 |
|------|----------|
| 前端框架 | React 19 + TypeScript + Vite + Tailwind CSS 4 + React Router 6 + Zustand |
| UI 动效 | framer-motion（页面切换过渡 / PDF 抽屉滑入）+ lucide-react（图标） |
| Markdown / 公式 | react-markdown + remark-gfm + remark-math + rehype-katex + KaTeX |
| 知识图谱可视化 | @react-sigma/core + graphology + graphology-layout-forceatlas2 + sigma |
| 后端框架 | Python + FastAPI + Uvicorn + Pydantic + sse-starlette（SSE 流式） |
| AI / 对话引擎 | LLM（OpenAI 兼容 SDK，默认 DeepSeek）+ Prompt Engineering（20+ 模板文件） |
| RAG 检索引擎 | sentence-transformers（本地文本嵌入，lazy 加载 torch）+ FAISS（向量索引）+ pdfplumber / python-pptx / python-docx（文档提取） |
| 认证与安全 | itsdangerous（URLSafeTimedSerializer 签名 Cookie，httpOnly）+ 学习码 / 管理员密码 |
| 数据存储 | JSON 文件（课程/材料/视频/评分元数据）+ Markdown 文件（用户画像）+ FAISS 索引文件（向量库） |
| 部署方式 | systemd + Uvicorn（单 worker）→ 阿里云 Ubuntu 22.04（2 核 2G）→ deploy.sh 一键部署；可选 Nginx 反向代理（HTTPS/域名） |

---

## 7. 安全设计（Security）

- **认证机制**：学习码（降低学生使用门槛，无需密码）+ 管理员密码；itsdangerous `URLSafeTimedSerializer` 签名 Cookie（`star_session`），httpOnly（防 XSS 窃取）、`samesite=lax`、生产环境 secure 标志。
- **数据隔离**：学生画像/会话/模型配置按 `user_id` 分目录存储（`data/users/{uid}/`），后端所有读/写操作显式传入 uid、校验归属；会话越权访问返回 403。
- **角色鉴权**：管理员专属接口（课程/材料/视频/学生管理）全部用 `Depends(require_admin)` 守卫；学生调用直接返回 403。
- **静态文件鉴权**：`/api/material/*` 由 FastAPI 中间件 `auth_guard` 校验 Cookie——图片/视频/PDF 等静态资源同源自动带 httpOnly Cookie，未登录无法直接访问。
- **API Key 安全**：学生自配的 API Key 存于服务器文件（`data/users/{uid}/model_config.json`），属主收紧（`chmod 600`）；生产环境建议进一步加密存储。
- **PDF 防下载（轻量）**：前端 PDF 预览面板隐藏 Chrome/Edge PDFium 的下载/打印按钮，禁用右键菜单；彻底防护需后端 PDF 转图片（未做）。

---

## 8. 项目结构（Project Structure）

以下为项目实际目录结构（与代码仓库一致）：

```
Star-Multi-User/
├── backend/
│   ├── main.py                # FastAPI 应用入口，路由注册 + 中间件
│   ├── config.py              # 路径常量 / 5 级掌握度色彩 / 默认 LLM key / 多用户配置
│   ├── deps.py                # 依赖注入：current_user / require_admin / require_llm
│   ├── requirements.txt
│   ├── agent/                 # AI 引擎模块
│   │   ├── onboarding.py      # 首次评估 → 生成三份画像
│   │   ├── planner.py         # 学习清单规划（注入课堂概要）
│   │   ├── steps.py           # 学习步执行（情景 / 讲解 / 练习 / 视频推荐）
│   │   ├── grader.py          # 练习判题
│   │   ├── summarizer.py      # 结课 → 更新画像
│   │   ├── context.py         # 按 uid 构建 LLM 上下文
│   │   ├── graph_builder.py   # knowledge.md → 知识图谱（按 uid 缓存）
│   │   ├── video.py           # 视频分析 + 智能挑选
│   │   ├── lesson_summary.py  # 课堂概要生成
│   │   ├── class_report.py    # 班级共性报告生成
│   │   └── memory_update.py   # 画像 diff 更新
│   ├── rag/                   # RAG 检索引擎
│   │   ├── extractor.py       # 文档提取（PDF/PPTX/DOCX，含 page 字段）
│   │   ├── chunker.py         # 文本分块（透传 page）
│   │   ├── embedder.py        # sentence-transformers 嵌入（lazy torch）
│   │   ├── index_manager.py   # FAISS 索引管理
│   │   └── retriever.py       # 检索 + 上下文组装
│   ├── routes/                # API 路由
│   │   ├── auth.py            # 登录 / 登出 / 会话检查
│   │   ├── session.py         # SSE 流式学习（start/step/followup/end）
│   │   ├── graph.py           # 知识图谱数据
│   │   ├── course.py          # 课程 / 课时 CRUD
│   │   ├── material.py        # 材料上传 + RAG 索引构建
│   │   ├── video.py           # 视频上传 / 分析 / 评分
│   │   ├── lesson_summary.py  # 课堂概要 CRUD
│   │   ├── admin.py           # 学生管理 + 班级报告
│   │   ├── onboarding.py      # 评估接口
│   │   ├── profile_io.py      # 画像导入 / 导出 / 替换
│   │   ├── model_config.py    # LLM 模型配置
│   │   ├── health.py          # 健康检查
│   │   └── memory.py          # 画像文件直接读写
│   ├── store/                 # 数据持久化
│   │   ├── json_store.py          # 通用 JSON 文件读写（线程安全锁）
│   │   ├── auth.py                # Cookie 签发 / 校验
│   │   ├── session_store.py       # 学习会话（按 uid 分目录）
│   │   ├── material_store.py      # 材料元数据
│   │   ├── video_store.py         # 视频元数据
│   │   ├── rating_store.py        # 视频评分聚合
│   │   ├── lesson_summary_store.py# 课堂概要
│   │   ├── model_config_store.py  # 模型配置（按 uid）
│   │   └── profile_store.py       # 学生身份信息
│   ├── memory/                # 画像文件读写
│   │   ├── md_store.py            # 通用 .md 读写 + 备份
│   │   ├── knowledge_md.py        # knowledge.md 解析 / 序列化 / safe_update
│   │   ├── user_md.py             # USER.md 处理
│   │   └── study_md.py            # study.md 处理
│   ├── prompts/               # Prompt 模板（.txt，{var} 占位）
│   │   ├── plan.txt / step_explain.txt / step_scenario.txt
│   │   ├── step_practice.txt / followup.txt / judge.txt
│   │   ├── memory_update.txt / persona.txt / onboarding_*.txt
│   │   ├── video_analyze.txt / video_pick.txt
│   │   ├── lesson_summary.txt / class_report.txt
│   │   └── ...
│   ├── models/                # Pydantic 数据模型
│   ├── data/                  # 运行时数据（不提交 git）
│   │   ├── users/{uid}/           # 画像 / 会话 / 模型配置（按用户隔离）
│   │   ├── courses.json / lessons.json / materials_meta.json
│   │   ├── videos.json / lesson_summaries.json / video_ratings.json
│   │   └── rag_index/             # FAISS 索引文件
│   └── material/              # 课程文件 + 视频（全班共享）
├── frontend/
│   ├── src/
│   │   ├── pages/             # 6 个页面
│   │   │   ├── LoginPage.tsx          # 学习码 + 管理员密码登录
│   │   │   ├── GraphPage.tsx          # 知识图谱首页（默认）
│   │   │   ├── LearnEntryPage.tsx     # 选择课时 + 学习目标
│   │   │   ├── SessionPage.tsx        # 互动学习主界面（SSE 流式）
│   │   │   ├── OnboardingPage.tsx     # 首次评估
│   │   │   └── MinePage.tsx           # 我的（画像 + 模型配置 + 教师端：课程管理 / 学生管理）
│   │   ├── components/        # 可复用组件
│   │   │   ├── graph/                # 知识图谱渲染
│   │   │   ├── course/               # 课程管理器（含课堂概要面板 + 视频匹配度）
│   │   │   ├── video/                # VideoPlayer + RatingBar
│   │   │   ├── admin/                # StudentManager（教师学生管理）
│   │   │   ├── ui/                   # Button / ChipInput 等通用 UI
│   │   │   ├── AssistantText.tsx     # AI 回答流式渲染 + 引文解析
│   │   │   ├── CitationDot.tsx       # 可点击引文小圆
│   │   │   ├── PdfDrawer.tsx         # PDF 侧滑面板（framer-motion）
│   │   │   ├── Markdown.tsx          # Markdown + KaTeX 渲染
│   │   │   ├── CapsuleNav.tsx        # 底部胶囊导航
│   │   │   └── PageTransition.tsx    # 页面切换过渡
│   │   ├── api/              # 前端 API 层
│   │   │   ├── client.ts             # HTTP 请求封装
│   │   │   └── sse.ts                # SSE 流式接收（token 级 + rag_context/video 事件）
│   │   ├── lib/              # 工具函数 / 常量 / 动效
│   │   ├── hooks/            # 自定义 hooks（useSlideDirection 等）
│   │   ├── store/            # Zustand 全局状态（app-store）
│   │   └── types/            # TypeScript 类型定义
│   ├── dist/                 # Vite 构建产物（生产部署）
│   ├── package.json / vite.config.ts / tsconfig*.json / tailwind.config.ts
│   └── index.html
├── deploy.sh                  # 一键部署脚本（阿里云 Ubuntu）
├── DEPLOY.md                  # 部署分步说明
├── run.py                     # 本地一键启动（自动建 .venv → 装依赖 → 起服务）
├── README.md                  # 完整项目文档
├── CLAUDE.md                  # 开发规范文档
└── environment.yml            # conda 环境（可选）
```

---

## 9. 使用说明（Usage）

<img width="400" height="400" alt="image" src="https://github.com/user-attachments/assets/407c93d7-061e-4c0c-9432-a6080ad570ca" />
###                         图 1：登录页


学生凭"学习码 + 姓名"登录（无需密码）；教师额外填写管理员密码。`httpOnly` Cookie 自动签发，30 天有效。登录后按角色进入不同界面。


### 图 2：知识图谱首页

登录后的默认首页。节点表示知识点，颜色从灰到紫映射 0–4 级掌握度。图谱随学习进程动态生长——新知识加入、掌握度颜色更新。

### 图 3：首次评估与画像生成

学生首次使用时，AI 引导完成学习目标识别 + 基础摸底（专业/编程/ML）。评估后即时生成三份画像（`USER.md` / `knowledge.md` / `study.md`），无需等待。

### 图 4：选择课时与学习目标

「学习」页列表展示课程的全部课时。单选某一课时后，可补充学习意图并标注熟悉程度。右侧/下方展示课堂概要（如有），帮助了解本节要点。

### 图 5：互动学习对话（核心）

AI 按学习清单逐步推进（情景带入 → 知识讲解 → 练习 → 视频）。左上角清单图标可展开查看进度，已完成步骤显示删除线。随时可追问、质疑、展开——输入框始终可用。

### 图 6：PDF 课件出处标注

当 AI 回答引用了课件 PDF 时，句末出现可点击的 ①② 编号。点击后右侧面板从右滑入（framer-motion 动效），正文自动左移不被遮挡；iframe 内嵌 PDF 并跳到对应页。讲稿/资料作为上下文但不标引文——只有课件出处被编号。

### 图 7：视频推荐与评分

学习清单走到 video 步时，AI 据当前知识点推荐最匹配的 1–3 个视频，页面内 `<video>` 直接播放。每个视频下方有 1–5 星评分条——学生打分反馈帮助程度，分数聚合进全班评分，影响后续推荐。

### 图 8：课程管理（管理员）

「我的 → 课程」：管理员新建/管理课程与课时，上传讲稿/课件/资料（PDF/PPTX/DOCX），上传视频并填写角度/重点/时长。首次上传讲稿自动生成课堂概要，支持重新生成/导入/导出。视频名旁显示全班均分 ★4.2(12)。

### 图 9：学生管理与班级报告（管理员）

「我的 → 学生管理」：表格展示全班学生姓名、知识点数、平均掌握度、会话数、最近活跃。点击学生可查看其三份画像全文。点击「生成班级共性报告」——AI 聚合全班画像输出认知共性、薄弱点、教学建议。

### 图 10：我的页面（画像与配置）

「我的」页：查看/导出/替换三份画像（替换前自动备份）；配置 LLM 模型（API 地址/密钥/模型名，未配置时使用系统预配的课堂共用 key）。教师端额外显示「课程管理」与「学生管理」入口。

---

## 10. 创新点（Innovation Highlights）

### ① 单课深耕 × 私有知识驱动，杜绝幻觉

严格限定课程边界，AI 的所有回答基于教师上传的真实课件、讲稿、资料——通过 RAG 检索原文片段注入 Prompt。不是"大模型听说过这门课"，而是"每次回答都先查课件里怎么写的"。这是解决专业课程 AI 幻觉的根本方案。

### ② 三维画像 + 知识图谱双驱动，越用越懂学生

仿照 Claude Code 记忆机制，维护 `USER.md`（认知习惯）+ `knowledge.md`（知识掌握，五级量化）+ `study.md`（学习策略）。三份画像随每次学习自动更新（`knowledge` 只升不降），图谱同步刷新。学生不需要手动"设置偏好"——AI 在互动中观察、总结、适应。

### ③ 七步个性化学习闭环

评估 → 规划 → 情景带入 → 知识讲解 → 练习反馈 → 视频推荐 → 画像更新。每一步根据学生画像动态调整（如对该生减少灌输、增加情景），远超市面上"统一课件 + 课后选择题"的刻板在线学习。

### ④ 课件引用可追溯——让 AI "有据可查"

AI 回答引用课件 PDF 时，句末自动标注可点击编号 ①②，点击跳到 PDF 对应页。讲稿/资料作为背景上下文但不标引文——只有课件出处被精确编号。这是"课堂资料驱动、而非凭空而论"的落地证据，也是消除 AI 幻觉的可信手段。

### ⑤ 视频推荐 × 学生评分反馈循环

视频不按固定列表播放——AI 根据当前知识点 + 视频角度/重点/时长 + 全班历史评分，智能挑选最匹配的视频。学生打分后，差评视频（≥3 人、均分 < 2.5）自动过滤。推荐质量随使用不断优化，典型的 AI 反馈循环。

### ⑥ 从"个人工具"到"课堂基础设施"

多用户架构：学生用学习码登录、全班共用一个链接。教师端：学生列表 + 画像查看 + 一键生成班级共性报告（LLM 聚合全班薄弱点 + 教学建议）。不只服务个别学生——赋能老师把握全班学情、调整教学策略。这是从"AI 学习工具"到"AI 教学基础设施"的升维。

---

## 11. 预期成果与验证指标（Expected Outcomes）

### 交付物清单

- Web 应用 MVP（含前后端完整代码、一键部署脚本，已在阿里云跑通）
- 知识图谱 V1.0（课程知识点拓扑网络，掌握度五级色彩映射）
- 用户画像模板（`USER.md` / `knowledge.md` / `study.md` 三件套）
- 完整设计文档 & 用户手册
- 软著申请材料

### 量化指标（预期目标）

| 指标 | 目标值 | 说明 |
|------|--------|------|
| 知识图谱覆盖率 | ≥ 90% | 课程大纲知识点被图谱覆盖的比例 |
| 画像完整率 | ≥ 85% | 完成首次评估并生成三份画像的学生比例 |
| AI 响应时间 | ≤ 3s（首 token） | SSE 流式，首个 token 到达时间 |
| 平均会话轮数 | ≥ 5 轮 | 单次学习中学生的追问/互动次数 |
| 视频点击率 | ≥ 50% | 学习清单走到 video 步时学生实际播放视频的比例 |
| 次日留存率 | ≥ 35% | 首次使用后第二天再次登录的学生比例 |

### 验证方案

- 深理工 10–20 名师生 2 周封闭试用。
- SUS（系统可用性量表）+ NPS（净推荐值）评估。
- 前测/后测对比（知识掌握度变化）。
- 教师半结构化访谈（课堂概要 / 班级报告有用性评估）。

---

## 12. 开发路线图（Roadmap）

### 已完成 ✅

- 需求确认、课程资料结构化、知识图谱 V0.1。
- 前端 6 个核心页面开发（登录 / 图谱 / 选课 / 学习 / 评估 / 我的）。
- 对话引擎接入（SSE 流式）、RAG 检索引擎（sentence-transformers + FAISS，PDF/PPTX/DOCX 全链路）。
- 三维画像系统（首次评估 + 学习后自动更新 + 导入/导出/替换）。
- 多用户与角色系统（学习码登录 / 管理员 / 数据隔离 / Cookie 鉴权）。
- 视频推荐 + 评分反馈闭环。
- PDF 课件出处标注（引文编号 + 侧滑面板 + 页码跳转）。
- 课堂概要（讲稿驱动自动生成，支持重新生成/导入/导出）。
- 教师学生管理面板 + 班级共性报告。
- 课程管理（CRUD + RAG 索引构建）。
- 阿里云 Ubuntu 部署跑通（deploy.sh 一键部署 + systemd 服务）。

### 进行中 🔄

- 用户试用启动（深理工 10–20 名师生）。
- Bug 修复与体验优化。

### 后续规划 📋

- 数据分析与报告撰写。
- 软著申请。
- 论文辅助阅读模块（文件夹式论文管理 + AI 翻译/问答）——原开发文档中的"论文阅读界面"。
- Nginx 反向代理 + HTTPS/域名（正式上线）。
- 异步 LLM 客户端（解决同步阻塞，提升多人并发体验）。
- 流式文件上传（当前整文件读入内存，大文件需优化）。

---

## 13. 团队与致谢（Team & Acknowledgements）

- **开发团队**：（成员姓名 + 分工）
- **指导老师**：
- **合作单位**：深圳理工大学、指南针学院
- **参考资料与开源项目致谢**：
  - Claude Code（Anthropic）—— 画像系统核心机制参考
  - React / FastAPI / sentence-transformers / FAISS / graphology 等开源项目

---

## 14. 许可证（License）

MIT / Apache 2.0（根据实际情况选择）

---

## 15. 联系方式（Contact）

- **邮箱**：
- **项目主页 / Demo 链接**：
