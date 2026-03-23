# AI之母项目
## 第一阶段 RAG系统：arXiv论文管理器

<div align="center">
  <h3>面向学习者的生产级RAG系统实践之旅</h3>
  <p>通过动手实践，从零开始学习构建现代AI系统</p>
  <p>掌握最热门的AI工程技能：<strong>RAG（检索增强生成）</strong></p>
</div>

<p align="center">
  <img src="https://img.shields.io/badge/Python-3.12+-blue.svg" alt="Python版本">
  <img src="https://img.shields.io/badge/FastAPI-0.115+-green.svg" alt="FastAPI">
  <img src="https://img.shields.io/badge/OpenSearch-2.19-orange.svg" alt="OpenSearch">
  <img src="https://img.shields.io/badge/Docker-Compose-blue.svg" alt="Docker">
  <img src="https://img.shields.io/badge/状态-第7周高级功能-brightgreen.svg" alt="状态">
</p>

</br>

<p align="center">
  <a href="#-关于本课程">
    <img src="static/mother_of_ai_project_rag_architecture.gif" alt="RAG架构" width="700">
  </a>
</p>

## 📖 关于本课程

这是一个<strong>面向学习者的项目</strong>，你将构建一个完整的研究助手系统，它可以自动获取学术论文、理解其内容，并使用先进的RAG技术回答你的研究问题。

**arXiv论文管理器**将教你使用行业最佳实践构建<strong>生产级RAG系统</strong>。与那些直接跳到向量搜索的教程不同，我们遵循<strong>专业路径</strong>：先掌握关键词搜索基础，再用向量增强实现混合检索。

> **🎯 专业差异：** 我们按照成功公司的方式构建RAG系统——坚实的搜索基础加上AI增强，而不是忽视搜索基础的AI优先方法。

完成本课程后，你将拥有自己的AI研究助手，以及为任何领域构建生产RAG系统的深厚技术技能。

### **🎓 你将构建什么**

- **第1周：** 使用Docker、FastAPI、PostgreSQL、OpenSearch和Airflow的完整基础设施
- **第2周：** 自动获取和解析arXiv学术论文的数据管道
- **第3周：** 具有过滤和相关性评分的生产级BM25关键词搜索
- **第4周：** 智能分块 + 结合关键词与语义理解的混合搜索
- **第5周：** 具有本地LLM、流式响应和Gradio界面的完整RAG管道
- **第6周：** 使用Langfuse追踪和Redis缓存的生产监控，优化性能
- **第7周：** **使用LangGraph的智能体RAG和Telegram机器人实现移动访问**

---

## 🏗️ 系统架构演进

### 第7周：智能体RAG和Telegram机器人集成
<div align="center">
  <img src="static/week7_telegram_and_agentic_ai.png" alt="第7周Telegram和智能体AI架构" width="800">
  <p><em>完整的第7周架构，展示Telegram机器人与智能体RAG系统的集成</em></p>
</div>

### LangGraph智能体RAG工作流
<div align="center">
  <img src="static/langgraph-mermaid.png" alt="LangGraph智能体RAG流程" width="800">
  <p><em>详细的LangGraph工作流，展示决策节点、文档评分和自适应检索</em></p>
</div>


**第7周代码解析 + 博客：** [使用LangGraph和Telegram的智能体RAG](https://jamwithai.substack.com/p/agentic-rag-with-langgraph-and-telegram)

**第7周的关键创新：**
- **智能决策：** 智能体评估并调整检索策略
- **文档评分：** 使用语义评估的自动相关性判断
- **查询重写：** 当结果不足时自适应查询优化
- **护栏：** 域外检测防止幻觉
- **移动访问：** 在任何设备上通过Telegram机器人进行对话式AI
- **透明度：** 完整的推理步骤追踪，便于调试和信任

---

## 🚀 快速开始

### **📋 先决条件**
- **Docker Desktop**（包含Docker Compose）
- **Python 3.12+**
- **UV包管理器**（[安装指南](https://docs.astral.sh/uv/getting-started/installation/)）
- **8GB+内存**和**20GB+可用磁盘空间**

### **⚡ 开始使用**

```bash
# 1. 克隆并设置
git clone <repository-url>
cd arxiv-paper-curator

# 2. 配置环境（重要！）
cp .env.example .env
# .env文件包含OpenSearch、arXiv API和服务连接的所有必要配置
# 默认配置开箱即用
# 你需要添加Jina嵌入免费API密钥和langfuse密钥（查看博客）

# 3. 安装依赖
uv sync

# 4. 启动所有服务
docker compose up --build -d

# 5. 验证一切正常
curl http://localhost:8000/health
```

### **📚 每周学习路径**

| 周 | 主题 | 博客文章 | 代码发布 |
|------|-------|-----------|--------------|
| **第0周** | AI之母项目 - 6个阶段 | [AI之母项目](https://jamwithai.substack.com/p/the-mother-of-ai-project) | - |
| **第1周** | 基础设施基础 | [驱动RAG系统的基础设施](https://jamwithai.substack.com/p/the-infrastructure-that-powers-rag) | [week1.0](https://github.com/jamwithai/arxiv-paper-curator/releases/tag/week1.0) |
| **第2周** | 数据摄入管道 | [为RAG构建数据摄入管道](https://jamwithai.substack.com/p/bringing-your-rag-system-to-life) | [week2.0](https://github.com/jamwithai/arxiv-paper-curator/releases/tag/week2.0) |
| **第3周** | OpenSearch摄入和BM25检索 | [每个RAG系统需要的搜索基础](https://jamwithai.substack.com/p/the-search-foundation-every-rag-system) | [week3.0](https://github.com/jamwithai/arxiv-paper-curator/releases/tag/week3.0) |
| **第4周** | **分块和混合搜索** | [让混合搜索奏效的分块策略](https://jamwithai.substack.com/p/chunking-strategies-and-hybrid-rag) | [week4.0](https://github.com/jamwithai/arxiv-paper-curator/releases/tag/week4.0) |
| **第5周** | **完整RAG系统** | [完整的RAG系统](https://jamwithai.substack.com/p/the-complete-rag-system) | [week5.0](https://github.com/jamwithai/arxiv-paper-curator/releases/tag/week5.0) |
| **第6周** | **生产监控和缓存** | [生产就绪的RAG：监控和缓存](https://jamwithai.substack.com/p/production-ready-rag-monitoring-and) | [week6.0](https://github.com/jamwithai/arxiv-paper-curator/releases/tag/week6.0) |
| **第7周** | **智能体RAG和Telegram机器人** | [使用LangGraph和Telegram的智能体RAG](https://jamwithai.substack.com/p/agentic-rag-with-langgraph-and-telegram) | [week7.0](https://github.com/jamwithai/arxiv-paper-curator/releases/tag/week7.0) |

**📥 克隆特定周的发布版本：**
```bash
# 克隆特定周的代码
git clone --branch <WEEK_TAG> https://github.com/jamwithai/arxiv-paper-curator
cd arxiv-paper-curator
uv sync
docker compose down -v
docker compose up --build -d

# 将<WEEK_TAG>替换为：week1.0、week2.0等
```

### **📊 访问你的服务**

| 服务 | URL | 用途 |
|---------|-----|---------|
| **API文档** | http://localhost:8000/docs | 交互式API测试 |
| **Gradio RAG界面** | http://localhost:7861 | 用户友好的聊天界面 |
| **Langfuse仪表板** | http://localhost:3000 | RAG管道监控和追踪 |
| **Airflow仪表板** | http://localhost:8080 | 工作流管理 |
| **OpenSearch仪表板** | http://localhost:5601 | 混合搜索引擎UI |

#### **注意**：查看airflow/simple_auth_manager_passwords.json.generated获取Airflow用户名和密码
---

## 📚 第1周：基础设施基础 ✅

**从这里开始！** 掌握驱动现代RAG系统的基础设施。

### **🎯 学习目标**
- 使用Docker Compose完成基础设施设置
- FastAPI开发，包含自动文档和健康检查
- PostgreSQL数据库配置和管理
- OpenSearch混合搜索引擎设置
- Ollama本地LLM服务配置
- 服务编排和健康监控
- 具有代码质量工具的专业开发环境

### **🏗️ 架构概览**

<p align="center">
  <img src="static/week1_infra_setup.png" alt="第1周基础设施设置" width="800">
</p>

**基础设施组件：**
- **FastAPI**：具有异步支持的REST端点（端口8000）
- **PostgreSQL 16**：论文元数据存储（端口5432）
- **OpenSearch 2.19**：具有仪表板的搜索引擎（端口9200、5601）
- **Apache Airflow 3.0**：工作流编排（端口8080）
- **Ollama**：本地LLM服务器（端口11434）

### **📓 设置指南**

```bash
# 启动第1周笔记本
uv run jupyter notebook notebooks/week1/week1_setup.ipynb
```

**完成指南：** 按照[第1周笔记本](notebooks/week1/week1_setup.ipynb)进行动手设置和验证步骤。

### **📖 深入了解**
**博客文章：** [驱动RAG系统的基础设施](https://jamwithai.substack.com/p/the-infrastructure-that-powers-rag) - 详细讲解和生产洞察

---

## 📚 第2周：数据摄入管道 ✅

**建立在第1周基础设施之上：** 学习自动获取、处理和存储学术论文。

### **🎯 学习目标**
- 具有速率限制和重试逻辑的arXiv API集成
- 使用Docling进行科学PDF解析
- 使用Apache Airflow的自动化数据摄入管道
- 元数据提取和存储工作流
- 从API到数据库的完整论文处理

### **🏗️ 架构概览**

<p align="center">
  <img src="static/week2_data_ingestion_flow.png" alt="第2周数据摄入架构" width="800">
</p>

**数据管道组件：**
- **MetadataFetcher**：🎯 协调整个管道的主编排器
- **ArxivClient**：具有重试逻辑的速率限制论文获取
- **PDFParserService**：Docling驱动的科学文档处理
- **Airflow DAGs**：自动化每日论文摄入工作流
- **PostgreSQL存储**：结构化论文元数据和内容

### **📓 实现指南**

```bash
# 启动第2周笔记本
uv run jupyter notebook notebooks/week2/week2_arxiv_integration.ipynb
```

**完成指南：** 按照[第2周笔记本](notebooks/week2/week2_arxiv_integration.ipynb)进行动手实现和验证步骤。

### **📖 深入了解**
**博客文章：** [为RAG构建数据摄入管道](https://jamwithai.substack.com/p/bringing-your-rag-system-to-life) - arXiv API集成和PDF处理

---

## 📚 第3周：关键词搜索优先 - 关键基础

**建立在第1-2周基础之上：** 实现专业RAG系统所依赖的关键词搜索基础。

### **🎯 学习目标**
- 为什么关键词搜索对RAG系统至关重要（基础优先方法）
- OpenSearch索引管理、映射和搜索优化
- BM25算法和有效关键词搜索背后的数学原理
- 用于构建具有过滤和加权的复杂搜索查询的查询DSL
- 用于衡量相关性和性能的搜索分析
- 真实公司使用的生产模式

### **🏗️ 架构概览**

<p align="center">
  <img src="static/week3_opensearch_flow.png" alt="第3周OpenSearch流程架构" width="800">
</p>

**搜索基础设施组件：**
- **OpenSearch服务**：`src/services/opensearch/` - 专业搜索服务实现
- **搜索API**：`src/routers/search.py` - 具有BM25评分的搜索API端点
- **学习材料**：`notebooks/week3/` - 完整的OpenSearch集成指南
- **质量指标**：精确率、召回率和相关性评分

### **📓 设置指南**

```bash
# 启动第3周笔记本
uv run jupyter notebook notebooks/week3/week3_opensearch.ipynb
```

**完成指南：** 按照[第3周笔记本](notebooks/week3/week3_opensearch.ipynb)进行动手OpenSearch设置和BM25搜索实现。

### **📖 深入了解**
**博客文章：** [每个RAG系统需要的搜索基础](https://jamwithai.substack.com/p/the-search-foundation-every-rag-system) - 使用OpenSearch的完整BM25实现

---

## 📚 第4周：分块和混合搜索 - 语义层

**建立在第3周基础之上：** 添加让搜索真正智能的语义层。

### **🎯 学习目标**
- 具有智能文档分割的基于章节的分块
- 具有Jina AI集成和回退策略的生产级嵌入
- 使用RRF融合实现关键词+语义检索的混合搜索精通
- 支持多种搜索模式的统一API设计
- 搜索方法之间的性能分析和权衡

### **🏗️ 架构概览**

<p align="center">
  <img src="static/week4_hybrid_opensearch.png" alt="第4周混合搜索架构" width="800">
</p>

**混合搜索基础设施组件：**
- **文本分块器**：`src/services/indexing/text_chunker.py` - 具有重叠策略的章节感知分块
- **嵌入服务**：`src/services/embeddings/` - 使用Jina AI的生产级嵌入管道
- **混合搜索API**：`src/routers/hybrid_search.py` - 支持所有模式的统一搜索API
- **学习材料**：`notebooks/week4/` - 完整的混合搜索实现指南

### **📓 设置指南**

```bash
# 启动第4周笔记本
uv run jupyter notebook notebooks/week4/week4_hybrid_search.ipynb
```

**完成指南：** 按照[第4周笔记本](notebooks/week4/week4_hybrid_search.ipynb)进行动手实现和验证步骤。

### **📖 深入了解**
**博客文章：** [让混合搜索奏效的分块策略](https://jamwithai.substack.com/p/chunking-strategies-and-hybrid-rag) - 生产级分块和RRF融合实现

---

## 📚 第5周：具有LLM集成的完整RAG管道

**建立在第4周混合搜索之上：** 添加将搜索转化为智能对话的LLM层。

### **🎯 学习目标**
- 使用Ollama进行完全数据隐私的本地LLM集成
- 性能优化，提示词减少80%（6倍速度提升）
- 使用服务器发送事件实现流式响应
- 具有标准和流式端点的双API设计
- 具有高级参数控制的交互式Gradio界面

### **🏗️ 架构概览**

<p align="center">
  <img src="static/week5_complete_rag.png" alt="第5周完整RAG系统架构" width="900">
</p>

**完整RAG基础设施组件：**
- **RAG端点**：`src/routers/ask.py` - 双端点（`/api/v1/ask` + `/api/v1/stream`）
- **Ollama服务**：`src/services/ollama/` - 具有优化提示词的LLM客户端
- **系统提示词**：`src/services/ollama/prompts/rag_system.txt` - 针对学术论文优化
- **Gradio界面**：`src/gradio_app.py` - 具有流式支持的交互式Web UI
- **启动脚本**：`gradio_launcher.py` - 简易启动脚本（运行在端口7861）

### **📓 设置指南**

```bash
# 启动第5周笔记本
uv run jupyter notebook notebooks/week5/week5_complete_rag_system.ipynb

# 启动Gradio界面
uv run python gradio_launcher.py
# 打开 http://localhost:7861
```

**完成指南：** 按照[第5周笔记本](notebooks/week5/week5_complete_rag_system.ipynb)进行动手LLM集成和RAG管道实现。

### **📖 深入了解**
**博客文章：** [完整的RAG系统](https://jamwithai.substack.com/p/the-complete-rag-system) - 具有本地LLM集成和优化技术的完整RAG系统

---

## 📚 第6周：生产监控和缓存

**建立在第5周完整RAG系统之上：** 添加可观测性、性能优化和生产级监控。

### **🎯 学习目标**
- Langfuse集成实现端到端RAG管道追踪
- 具有智能缓存键和TTL管理的Redis缓存策略
- 具有实时仪表板的性能监控，用于延迟和成本
- 可观测性和优化的生产模式
- 成本分析和LLM使用优化（缓存带来150-400倍加速）

### **🏗️ 架构概览**

<p align="center">
  <img src="static/week6_monitoring_and_caching.png" alt="第6周监控和缓存架构" width="900">
</p>

**生产基础设施组件：**
- **Langfuse服务**：`src/services/langfuse/` - 具有RAG特定指标的完整追踪集成
- **缓存服务**：`src/services/cache/` - 具有精确匹配缓存和优雅回退的Redis客户端
- **更新端点**：`src/routers/ask.py` - 集成追踪和缓存中间件
- **Docker配置**：`docker-compose.yml` - 添加Redis服务和Langfuse本地实例
- **学习材料**：`notebooks/week6/` - 完整的监控和缓存实现指南

### **📓 设置指南**

```bash
# 启动第6周笔记本
uv run jupyter notebook notebooks/week6/week6_cache_testing.ipynb
```

**完成指南：** 按照[第6周笔记本](notebooks/week6/week6_cache_testing.ipynb)进行动手Langfuse追踪和Redis缓存实现。

### **📖 深入了解**
**博客文章：** [生产就绪的RAG：监控和缓存](https://jamwithai.substack.com/p/production-ready-rag-monitoring-and) - 具有监控和缓存的生产就绪RAG

---

## 📚 第7周：使用LangGraph的智能体RAG和Telegram机器人

**建立在第6周生产系统之上：** 添加智能推理、多步决策和Telegram机器人集成实现移动优先AI交互。

### **🎯 学习目标**
- 使用LangGraph工作流实现具有决策节点的基于状态的智能体编排
- 用于查询验证和领域边界检测的护栏实现
- 使用语义相关性评估的文档评分
- 用于自动查询优化和更好检索的查询重写
- 具有多尝试检索和智能回退的自适应检索
- 具有异步操作和错误处理的Telegram机器人集成
- 暴露智能体决策过程的推理透明性

### **🏗️ 架构概览**

<p align="center">
  <img src="static/week7_telegram_and_agentic_ai.png" alt="第7周智能体RAG和Telegram架构" width="900">
</p>

**智能体RAG基础设施组件：**
- **智能体节点**：`src/services/agents/nodes/` - 护栏、检索、评分、重写和生成节点
- **工作流编排**：`src/services/agents/agentic_rag.py` - LangGraph工作流协调
- **Telegram机器人**：`src/services/telegram/` - 命令处理器和消息处理
- **智能体端点**：`src/routers/agentic_ask.py` - 智能体RAG API端点
- **学习材料**：`notebooks/week7/` - 第7周学习材料和示例

### **📓 设置指南**

```bash
# 启动第7周笔记本
uv run jupyter notebook notebooks/week7/week7_agentic_rag.ipynb
```

**完成指南：** 按照[第7周笔记本](notebooks/week7/week7_agentic_rag.ipynb)进行动手LangGraph智能体RAG和Telegram机器人实现。

### **📖 深入了解**
**博客文章：** [使用LangGraph和Telegram的智能体RAG](https://jamwithai.substack.com/p/agentic-rag-with-langgraph-and-telegram) - 构建具有决策、自适应检索和移动访问的智能智能体

---

## ⚙️ 配置

**设置：**
```bash
cp .env.example .env
# 根据你的环境编辑.env
```

**关键变量：**
- `JINA_API_KEY` - 第4周+必需（使用嵌入的混合搜索）
- `TELEGRAM__BOT_TOKEN` - 第7周必需（Telegram机器人集成）
- `LANGFUSE__PUBLIC_KEY` 和 `LANGFUSE__SECRET_KEY` - 第6周可选（监控）

**完整配置：** 查看[.env.example](.env.example)获取所有可用选项和详细文档。

---

## 🔧 参考和开发指南

### **🛠️ 技术栈**

| 服务 | 用途 | 状态 |
|---------|---------|--------|
| **FastAPI** | 具有自动文档的REST API | ✅ 就绪 |
| **PostgreSQL 16** | 论文元数据和内容存储 | ✅ 就绪 |
| **OpenSearch 2.19** | 混合搜索引擎（BM25 + 向量） | ✅ 就绪 |
| **Apache Airflow 3.0** | 工作流自动化 | ✅ 就绪 |
| **Jina AI** | 嵌入生成（第4周） | ✅ 就绪 |
| **Ollama** | 本地LLM服务（第5周） | ✅ 就绪 |
| **Redis** | 高性能缓存（第6周） | ✅ 就绪 |
| **Langfuse** | RAG管道可观测性（第6周） | ✅ 就绪 |

**开发工具：** UV、Ruff、MyPy、Pytest、Docker Compose

### **🏗️ 项目结构**

```
arxiv-paper-curator/
├── src/                    # 主应用代码
│   ├── routers/            # API端点（search、ask、papers）
│   ├── services/           # 业务逻辑（opensearch、ollama、agents、cache）
│   ├── models/             # 数据库模型（SQLAlchemy）
│   ├── schemas/            # Pydantic验证模式
│   └── config.py           # 环境配置
├── notebooks/              # 每周学习材料（week1-7）
├── airflow/                # 工作流编排（DAGs）
├── tests/                  # 测试套件
└── compose.yml             # Docker服务编排
```

### **📡 API端点参考**

| 端点 | 方法 | 描述 | 周 |
|----------|--------|-------------|------|
| `/health` | GET | 服务健康检查 | 第1周 |
| `/api/v1/papers` | GET | 列出存储的论文 | 第2周 |
| `/api/v1/papers/{id}` | GET | 获取特定论文 | 第2周 |
| `/api/v1/search` | POST | BM25关键词搜索 | 第3周 |
| `/api/v1/hybrid-search/` | POST | 混合搜索（BM25 + 向量） | **第4周** |

**API文档：** 访问 http://localhost:8000/docs 使用交互式API浏览器

### **🔧 常用命令**

#### **使用Makefile**（推荐）
```bash
# 查看所有可用命令
make help

# 快速工作流
make start         # 启动所有服务
make health        # 检查所有服务健康状态
make test          # 运行测试
make stop          # 停止服务
```

#### **所有可用命令**
| 命令 | 描述 |
|---------|-------------|
| `make start` | 启动所有服务 |
| `make stop` | 停止所有服务 |
| `make restart` | 重启所有服务 |
| `make status` | 显示服务状态 |
| `make logs` | 显示服务日志 |
| `make health` | 检查所有服务健康状态 |
| `make setup` | 安装Python依赖 |
| `make format` | 格式化代码 |
| `make lint` | 代码检查和类型检查 |
| `make test` | 运行测试 |
| `make test-cov` | 运行测试并生成覆盖率报告 |
| `make clean` | 清理所有内容 |

#### **直接命令**（替代方案）
```bash
# 如果你更喜欢直接使用命令
docker compose up --build -d    # 启动服务
docker compose ps               # 检查状态
docker compose logs            # 查看日志
uv run pytest                 # 运行测试
```

### **🎓 目标受众**
| 人群 | 原因 |
|-----|-----|
| **AI/ML工程师** | 学习超越教程的生产RAG架构 |
| **软件工程师** | 使用最佳实践构建端到端AI应用 |
| **数据科学家** | 使用现代工具实现生产AI系统 |

---

## 🛠️ 故障排除

**常见问题：**
- **服务无法启动？** 等待2-3分钟，检查`docker compose logs`
- **端口冲突？** 停止使用端口8000、8080、5432、9200的其他服务
- **内存问题？** 增加Docker Desktop的内存分配

**获取帮助：**
- 查看第1周笔记本中全面的故障排除章节
- 查看服务日志：`docker compose logs [service-name]`
- 完全重置：`docker compose down --volumes && docker compose up --build -d`

---

## 💰 成本结构

**本课程完全免费！** 你只需要为可选服务支付极少费用：
- **本地开发：** $0（所有内容本地运行）
- **可选云API：** 约$2-5用于外部LLM服务（如果选择使用）

---

<div align="center">
  <h3>🎉 准备好开始你的AI工程之旅了吗？</h3>
  <p><strong>从第1周设置笔记本开始，构建你的第一个生产RAG系统！</strong></p>

  <p><em>专为想要掌握现代AI工程的学习者打造</em></p>
  <p><strong>由 <a href="https://www.linkedin.com/in/shirin-khosravi-jam/">Shirin Khosravi Jam</a> 和 <a href="https://www.linkedin.com/in/shantanuladhwe/">Shantanu Ladhwe</a> 用心构建</strong></p>
</div>

---

## Star历史

[![Star历史图表](https://api.star-history.com/svg?repos=jamwithai/production-agentic-rag-course&type=Date)](https://star-history.com/#jamwithai/production-agentic-rag-course&Date)

---

## 📄 许可证

MIT许可证 - 详情请查看[LICENSE](LICENSE)文件。