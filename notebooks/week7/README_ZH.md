# 第7周：使用LangGraph的智能体RAG + Telegram机器人

## 概述

第7周为arXiv论文管理器添加了两个主要增强功能：

1. **🤖 使用LangGraph的智能体RAG** - 具有决策能力的智能、自适应检索
2. **💬 Telegram机器人集成** - 用于移动/桌面访问的对话界面

---

## 🧠 第一部分：使用LangGraph的智能体RAG

### 什么是智能体RAG？

**传统RAG**（第5-6周）：
```
查询 → 始终检索 → 生成答案
```

**智能体RAG**（第7周）：
```
查询 → 智能体决定：
  ├─ 简单问题？→ 直接回答（更快！）
  └─ 需要研究？→ 检索
       ├─ 相关文档？→ 生成答案
       └─ 不相关？→ 重写查询 → 重试
```

### 关键功能

- **🎯 智能决策** - LLM决定何时真正需要检索
- **📊 文档评分** - 验证检索的论文是否相关
- **🔄 查询优化** - 重写模糊查询以获得更好结果
- **🔍 推理透明** - 展示智能体的决策步骤
- **♻️ 迭代改进** - 必要时可用更好的查询重试

### 我们构建了什么

```
src/services/agents/
├── tools.py            # 包装OpenSearch的检索器工具
├── nodes.py            # 4个图节点（查询、评分、重写、生成）
├── agentic_rag.py      # LangGraph工作流 + 服务
├── prompts.py          # LLM提示词模板
└── factory.py          # 依赖注入

src/routers/
└── agentic_ask.py      # FastAPI端点

总计：约750行代码，遵循SOLID、KISS、DRY原则
```

### 架构

```
LangGraph工作流：

开始
  ↓
generate_query_or_respond
  ├─ 不需要检索 → 结束（直接响应）
  └─ 需要检索 → retrieve（工具节点）
       ↓
     grade_documents
       ├─ 相关 → generate_answer → 结束
       └─ 不相关 → rewrite_query →（循环返回）
```

### 新API端点

**`POST /api/v1/ask-agentic`**

```json
// 请求
{
  "query": "机器学习中的transformers是什么？",
  "top_k": 3,
  "use_hybrid": true
}

// 响应
{
  "query": "机器学习中的transformers是什么？",
  "answer": "Transformers是神经网络架构...",
  "sources": ["https://arxiv.org/pdf/1706.03762.pdf"],
  "chunks_used": 3,
  "search_mode": "hybrid",
  "reasoning_steps": [
    "决定检索相关论文",
    "从数据库检索文档",
    "从相关文档生成答案"
  ],
  "retrieval_attempts": 1
}
```

### 快速开始：智能体RAG

**1. 确保服务运行：**
```bash
docker compose up --build -d
```

**2. 使用cURL测试：**
```bash
# 简单问题（应直接回答）
curl -X POST http://localhost:8000/api/v1/ask-agentic \
  -H "Content-Type: application/json" \
  -d '{
    "query": "2+2等于多少？",
    "top_k": 3,
    "use_hybrid": true
  }'

# 研究问题（应检索论文）
curl -X POST http://localhost:8000/api/v1/ask-agentic \
  -H "Content-Type: application/json" \
  -d '{
    "query": "什么是注意力机制？",
    "top_k": 3,
    "use_hybrid": true
  }'
```

**3. 交互式测试：**
```bash
# 打开Jupyter笔记本
jupyter notebook notebooks/week7/week7_agentic_rag.ipynb
```

### 比较：传统 vs 智能体

| 功能 | 传统RAG | 智能体RAG |
|---------|----------------|-------------|
| **检索** | 始终检索 | 按需决定 |
| **相关性检查** | 无 | 对文档评分 |
| **查询优化** | 无 | 必要时重写 |
| **迭代** | 单次通过 | 多次尝试 |
| **透明度** | 黑盒 | 展示推理 |
| **简单问题** | 约15-20s | 约2-5s（无检索） |
| **复杂问题** | 单次尝试 | 迭代优化 |

### 测试场景

**场景1：直接响应（无检索）**
- 查询："5 + 7等于多少？"
- 预期：智能体回答"12"，不检索论文
- 推理："无需检索直接回答"

**场景2：成功检索**
- 查询："机器学习中的transformers是什么？"
- 预期：智能体检索论文、评分为相关、生成答案
- 推理："决定检索" → "检索文档" → "生成答案"

**场景3：查询重写**
- 查询："告诉我一些ML的东西"（模糊）
- 预期：智能体检索、评分为不相关、重写查询、重试
- 推理："检索" → "不相关" → "重写查询" → "再次检索" → "生成答案"

### 遵循的设计原则

- ✅ **SOLID** - 单一职责、依赖倒置、组合
- ✅ **KISS** - 简单节点（<30行）、清晰逻辑
- ✅ **DRY** - 复用现有服务（OpenSearch、Ollama、Jina）
- ✅ **YAGNI** - 只实现需要的功能
- ✅ **显式** - 类型提示、文档字符串、清晰命名
- ✅ **2025最佳实践** - MessagesState、ToolNode、tools_condition

### 文档

- **实现计划**：`docs/AGENTIC_RAG_IMPLEMENTATION_PLAN.md`
- **测试计划**：`docs/AGENTIC_RAG_TESTING_PLAN.md`
- **LangGraph 2025模式**：`docs/LANGGRAPH_2025_BEST_PRACTICES.md`
- **设计原则**：`docs/DESIGN_PRINCIPLES.md`
- **交互式笔记本**：`notebooks/week7/week7_agentic_rag.ipynb`

---

## 💬 第二部分：Telegram机器人集成

### 我们构建了什么

- **🤖 完整Telegram机器人集成**：具有命令支持的对话界面
- **💬 自然语言查询**：用自然语言提问，获得带来源的答案
- **⚡ 所有第6周功能**：Redis缓存（150-400倍加速）和Langfuse追踪
- **🎯 交互式命令**：`/start`、`/help`、`/ask`、`/search`、`/settings`、`/status`
- **👤 用户会话管理**：每用户偏好和对话历史
- **📱 移动优先**：具有可点击arXiv链接的富消息格式
- **🔐 可选访问控制**：如需要可白名单特定用户

## 架构

### 数据流
```
Telegram用户
    ↓
Telegram机器人（轮询/Webhook）
    ↓
TelegramService + 处理器
    ↓ [Langfuse追踪]
缓存检查（Redis）
    ├─ 命中 → 即时响应（约100ms）
    └─ 未命中 → 完整RAG管道
        ↓
混合搜索（OpenSearch BM25 + 向量）
        ↓
LLM生成（Ollama）
        ↓
缓存存储（Redis）
        ↓
格式化响应 → 发送到Telegram
```

### 新组件

```
src/services/telegram/
├── client.py           # 主Telegram机器人服务
├── handlers.py         # 命令和消息处理器
├── formatters.py       # 消息格式化工具
├── keyboards.py        # 交互式内联键盘
├── user_manager.py     # 用户设置和会话
└── factory.py          # 工厂函数

src/schemas/telegram/
├── messages.py         # 消息验证模式
├── commands.py         # 命令模式
└── user_settings.py    # 用户偏好模式
```

## 关键功能

### **对话界面**
- 自然语言查询：只需发送消息
- 具有历史追踪的上下文对话
- 自动查询路由（命令 vs 常规消息）

### **丰富命令**
```
/start    - 显示机器人功能的欢迎消息
/help     - 详细使用说明
/ask      - 明确提出研究问题
/search   - 按关键词快速搜索论文
/settings - 自定义偏好（搜索模式、结果数量）
/status   - 检查系统健康和统计
/clear    - 清除对话历史
```

### **交互式设置**
- 搜索模式：混合（BM25 + 语义）或仅BM25
- 每次查询结果数：3、5或10篇论文
- 类别过滤器：全部、cs.AI、cs.LG、cs.CL等
- 模型选择：选择LLM模型
- 开关：流式、来源显示、紧凑模式

### **用户体验**
- 处理期间显示**输入指示器**
- 支持Markdown的**丰富格式**
- 可点击的arXiv论文**链接**
- 长响应的**自动消息分割**
- 带有论文元数据的**来源引用**
- 即时响应的**缓存指示器**（⚡）

## 快速开始

### 先决条件

1. **Telegram账户** - 在手机或电脑上安装Telegram
2. **所有第1-6周服务运行** - 完整RAG栈必须正常运行

### 步骤1：创建你的Telegram机器人

1. **打开Telegram**并搜索`@BotFather`
2. **发送**`/newbot`给BotFather
3. **按照提示操作**：
   - 选择名称（如"My arXiv Curator"）
   - 选择用户名（如"my_arxiv_curator_bot" - 必须以"bot"结尾）
4. **复制机器人token** - 你将收到类似：
   ```
   1234567890:ABCdefGHIjklMNOpqrsTUVwxyz-1234567
   ```

### 步骤2：配置环境

将以下变量添加到你的`.env`文件：

```bash
# 启用Telegram机器人
TELEGRAM__ENABLED=true
TELEGRAM__BOT_TOKEN=your_token_from_botfather_here

# 可选：限制特定用户（逗号分隔的Telegram用户ID）
# 留空允许所有用户
TELEGRAM__ALLOWED_USER_IDS=

# 开发使用轮询模式（webhook需要HTTPS）
TELEGRAM__USE_WEBHOOK=false
```

### 步骤3：安装依赖

```bash
# 安装python-telegram-bot库
uv sync
```

### 步骤4：启动服务

```bash
# 启动所有服务（包括Telegram机器人）
docker compose up --build -d

# 查看日志验证Telegram机器人已启动
docker compose logs -f api
```

你应该看到：
```
INFO - Telegram bot started successfully
INFO - Starting Telegram bot in polling mode
INFO - Bot commands set successfully
```

### 步骤5：测试你的机器人

1. **打开Telegram**并搜索你的机器人用户名
2. **发送**`/start`给你的机器人
3. **尝试提问**："机器学习中的transformers是什么？"
4. **验证**你收到了带来源的答案！

## 测试说明

### 手动测试场景

#### 场景1：基本命令

**测试`/start`命令：**
```
你：/start
机器人：👋 欢迎使用arXiv论文管理器！
     [显示功能和快捷命令]
```

**测试`/help`命令：**
```
你：/help
机器人：📚 arXiv论文管理器帮助
     [显示详细命令文档]
```

**测试`/status`命令：**
```
你：/status
机器人：🔧 系统状态
     ✅ OPENSEARCH
     ✅ OLLAMA
     ✅ CACHE
     [显示系统健康]
```

#### 场景2：RAG问答

**测试简单问题：**
```
你：什么是注意力机制？
机器人：[首次15-20s]
     *答案：*
     注意力机制允许模型动态聚焦于...

     📚 *来源：*
     [1] *Attention Is All You Need*
         🔗 在arXiv上阅读 - 1706.03762
         📊 评分：12.456

     [2] *Neural Machine Translation...*
         🔗 在arXiv上阅读 - 1409.0473
         📊 评分：11.234

     ⚙️ 模式：hybrid
```

**测试缓存查询：**
```
你：什么是注意力机制？
机器人：[第二次约100ms ⚡]
     [与上面相同的答案]
     ⚙️ 模式：hybrid ⚡ 已缓存
```

#### 场景3：搜索命令

**测试论文搜索：**
```
你：/search transformer神经网络
机器人：📖 找到145篇论文（显示前10篇）

     1. *Attention Is All You Need*
        🔗 在arXiv上阅读 - 1706.03762
        📊 评分：12.456

     2. *BERT: Pre-training of Deep Bidirectional...*
        🔗 在arXiv上阅读 - 1810.04805
        📊 评分：11.234
     [...]
```

#### 场景4：设置自定义

**测试设置命令：**
```
你：/settings
机器人：⚙️ 你的设置

     *搜索模式：* HYBRID
     *每次查询结果：* 3篇论文
     *模型：* llama3.2:1b
     *类别：* 全部

     [出现交互按钮]：
     [🔍 混合搜索] [⚡ 仅BM25]
     [3个结果] [5个结果] [10个结果]
     [全部类别] [cs.AI] [cs.LG]
```

**测试更改设置：**
```
你：[点击"5个结果"按钮]
机器人：✅ 每次查询结果：5

你：[点击"cs.AI"按钮]
机器人：✅ 类别过滤器：cs.AI
```

#### 场景5：自然对话

**测试多轮对话：**
```
你：告诉我关于BERT的信息
机器人：[提供关于BERT的答案和来源]

你：它与GPT有什么不同？
机器人：[回答BERT与GPT的区别]

你：/clear
机器人：🗑️ 对话历史已清除！
     你的设置已保留。
```

#### 场景6：错误处理

**测试无效查询：**
```
你：asdfghjkl
机器人：❌ 未找到相关论文。
     尝试不同的关键词或检查你的类别过滤器。
```

**测试服务宕机时：**
```
你：测试查询
机器人：❌ 处理你的问题时出错：
     [用户友好的错误消息]
```

### 验证清单

- [ ] **机器人响应`/start`** - 显示欢迎消息
- [ ] **机器人响应所有命令** - `/help`、`/ask`、`/search`、`/settings`、`/status`、`/clear`
- [ ] **自然语言查询工作** - 可以不用命令提问
- [ ] **RAG答案包含来源** - 论文引用带arXiv链接
- [ ] **缓存工作** - 重复查询即时返回（⚡指示器）
- [ ] **设置持久化** - 更改在对话间保留
- [ ] **交互按钮工作** - 可以点击内联键盘按钮
- [ ] **长响应正确分割** - 消息不超过Telegram限制
- [ ] **Markdown格式工作** - 粗体、斜体、链接正确渲染
- [ ] **Langfuse显示追踪** - 检查http://localhost:3000查看Telegram事件
- [ ] **输入指示器显示** - 处理期间出现"机器人正在输入..."
- [ ] **错误消息友好** - 不向用户暴露堆栈跟踪

### 性能测试

**缓存性能：**
```bash
# 第一次查询（缓存未命中）
你：什么是机器学习？
机器人：[约15-20秒响应]

# 相同查询（缓存命中）
你：什么是机器学习？
机器人：[约100ms响应 ⚡]

# 验证150-400倍加速！
```

**并发用户：**
```bash
# 使用多个Telegram账户测试
# 每个用户应有独立的设置和会话
```

**会话管理：**
```bash
# 停止使用机器人30分钟（默认超时）
# 验证会话清理自动发生
```

## 配置

### 环境变量

```bash
# 启用/禁用机器人
TELEGRAM__ENABLED=true  # 设置为false禁用

# 机器人Token（必需）
TELEGRAM__BOT_TOKEN=your_token_here

# 部署模式
TELEGRAM__USE_WEBHOOK=false  # 生产环境设为true
TELEGRAM__WEBHOOK_URL=https://your-domain.com
TELEGRAM__WEBHOOK_PATH=/telegram/webhook

# 访问控制（可选）
TELEGRAM__ALLOWED_USER_IDS=123456789,987654321  # 空允许所有

# 行为设置
TELEGRAM__MAX_MESSAGE_LENGTH=4000  # Telegram限制是4096
TELEGRAM__ENABLE_STREAMING=true
TELEGRAM__SESSION_TIMEOUT_MINUTES=30
TELEGRAM__RATE_LIMIT_MESSAGES_PER_MINUTE=20

# 默认用户偏好
TELEGRAM__DEFAULT_TOP_K=3
TELEGRAM__DEFAULT_USE_HYBRID=true
TELEGRAM__DEFAULT_MODEL=llama3.2:1b
```

### 用户设置（每用户可自定义）

每个用户可以自定义：
- **搜索模式**：混合（BM25 + 语义）或仅BM25
- **结果数量**：每次查询3、5或10篇论文
- **类别**：全部、cs.AI、cs.LG、cs.CL、cs.CV、cs.NE
- **模型**：用于答案生成的LLM模型
- **显示**：显示来源、紧凑模式、流式

设置在会话间持久化，存储在内存中（生产环境考虑Redis/PostgreSQL存储）。

## 架构详情

### 服务集成

Telegram机器人与所有现有第1-6周服务集成：

1. **OpenSearch** - 相关论文的混合搜索
2. **Jina嵌入** - 语义搜索能力
3. **Ollama LLM** - 答案生成
4. **Redis缓存** - 重复查询150-400倍加速
5. **Langfuse** - Telegram交互的完整追踪
6. **PostgreSQL** - 论文元数据（通过OpenSearch）

### 消息流

```python
# 简化的消息流
async def handle_message(update, context):
    1. 提取user_id、chat_id、text
    2. 检查速率限制
    3. 获取/创建用户设置
    4. 显示"输入中..."指示器
    5. 检查缓存（Redis）
    6. 如果缓存命中 → 格式化并发送
    7. 如果缓存未命中：
        a. 生成嵌入（Jina）
        b. 搜索论文（OpenSearch）
        c. 使用上下文构建提示词
        d. 生成答案（Ollama）
        e. 缓存结果（Redis）
        f. 追踪交互（Langfuse）
    8. 格式化响应（Markdown）
    9. 如果太长则分割
    10. 发送到Telegram
    11. 更新对话历史
```

### 错误处理

机器人实现优雅降级：

- **Markdown格式失败** → 回退到纯文本
- **嵌入生成失败** → 回退到BM25搜索
- **缓存不可用** → 无缓存继续
- **Langfuse追踪失败** → 记录警告，继续
- **服务错误** → 用户友好的错误消息

### 速率限制

内置速率限制防止滥用：
- **每用户限制**：20条消息/分钟（可配置）
- **自动节流**：由Telegram库强制执行
- **会话超时**：30分钟不活动

## 故障排除

| 问题 | 解决方案 |
|-------|----------|
| **机器人不响应** | 检查`TELEGRAM__ENABLED=true`和有效的`BOT_TOKEN` |
| **"未授权"错误** | 机器人token无效，用@BotFather重新生成 |
| **机器人响应但没有答案** | 检查OpenSearch、Ollama和嵌入服务 |
| **响应缓慢** | 首次查询总是慢，后续查询使用缓存 |
| **Markdown格式损坏** | 机器人自动回退到纯文本 |
| **找不到机器人** | 确保机器人用户名正确，以"bot"结尾 |
| **"禁止"错误** | 检查`ALLOWED_USER_IDS`中的user_id限制 |
| **内存问题** | 用户会话存储在内存中，监控RAM使用 |

### 调试模式

启用详细日志：

```bash
# 在.env中
DEBUG=true

# 查看机器人日志
docker compose logs -f api | grep telegram
```

### 健康检查

```bash
# 检查机器人是否运行
curl http://localhost:8000/api/v1/health

# 应显示telegram_service状态
```

### 常见修复

**机器人无法启动：**
```bash
# 1. 检查token是否有效
# 2. 重启API服务
docker compose restart api

# 3. 查看日志中的错误
docker compose logs api
```

**响应太慢：**
```bash
# 1. 检查缓存是否工作
docker exec rag-redis redis-cli ping  # 应返回PONG

# 2. 检查缓存命中率
# 在机器人响应中查找⚡指示器

# 3. 验证Langfuse追踪没有阻塞
LANGFUSE__ENABLED=false  # 临时禁用
```

## 性能基准

| 指标 | 值 | 备注 |
|--------|-------|-------|
| **首次查询** | 15-20s | 完整RAG管道执行 |
| **缓存查询** | 50-100ms | **快150-400倍** |
| **输入指示器** | <500ms | 立即显示 |
| **仅搜索** | 2-3s | `/search`命令 |
| **状态检查** | <1s | `/status`命令 |
| **设置更新** | <100ms | 即时按钮响应 |
| **并发用户** | 10+ | 同时测试 |

### 缓存命中率（预期）

- **重复精确查询**：100%命中率
- **热门问题**：60-80%命中率
- **独特查询**：0%命中率（首次）

### 资源使用

- **内存**：每个活动用户会话约50MB
- **CPU**：最小（空闲<5%，生成期间峰值）
- **网络**：每条消息约10KB（不包括LLM生成）

## 生产部署

### Webhook模式（生产环境推荐）

```bash
# 生产环境.env
TELEGRAM__USE_WEBHOOK=true
TELEGRAM__WEBHOOK_URL=https://your-domain.com
TELEGRAM__WEBHOOK_PATH=/telegram/webhook

# 需要：
# - 具有有效证书的HTTPS域名
# - 用于TLS终止的Nginx/Caddy
# - 公网IP或反向代理
```

### 安全最佳实践

1. **限制用户**：私有机器人使用`ALLOWED_USER_IDS`
2. **速率限制**：保持默认限制（20条消息/分钟）
3. **环境变量**：永远不要提交带有真实token的`.env`
4. **仅HTTPS**：生产环境使用webhook模式
5. **监控**：通过Langfuse仪表板追踪使用情况

### 扩展考量

对于高流量部署：

1. **用户存储**：从内存迁移到Redis/PostgreSQL
2. **缓存**：增加Redis内存分配
3. **负载均衡**：多个API实例（webhook模式）
4. **速率限制**：根据需求调整
5. **监控**：设置错误和延迟告警

## 下一步

### 可选增强（第7.1周+）

- **📸 图片支持**：上传论文PDF，获取摘要
- **🗣️ 语音消息**：通过语音提问（语音转文字）
- **📊 用户分析**：使用模式仪表板
- **🤝 群聊**：多用户讨论
- **🌍 多语言**：国际化支持
- **🔔 通知**：新论文推送提醒
- **📈 个性化**：基于ML的论文推荐
- **🔗 语义缓存**：相似查询的模糊匹配

### 集成想法

- **Slack机器人**：使用相同架构移植到Slack
- **Discord机器人**：扩展到Discord社区
- **WhatsApp机器人**：使用WhatsApp Business API
- **Web组件**：嵌入网站
- **API访问**：暴露RESTful API用于集成

## 资源

- **Telegram Bot API**：https://core.telegram.org/bots/api
- **python-telegram-bot**：https://python-telegram-bot.org
- **@BotFather**：https://t.me/botfather
- **Langfuse文档**：https://langfuse.com/docs
- **Redis文档**：https://redis.io/docs

## 代码结构

```python
# 入口点：src/main.py
telegram_service = make_telegram_service(...)
await telegram_service.start()

# 服务：src/services/telegram/client.py
class TelegramService:
    async def start() -> 以轮询/webhook模式启动机器人
    async def stop() -> 优雅停止机器人
    async def health_check() -> 检查机器人状态

# 处理器：src/services/telegram/handlers.py
class TelegramHandlers:
    async def start_command() -> /start
    async def help_command() -> /help
    async def ask_command() -> /ask
    async def search_command() -> /search
    async def settings_command() -> /settings
    async def handle_message() -> 常规文本消息

# 格式化器：src/services/telegram/formatters.py
format_rag_response() -> 丰富的Markdown格式化
format_search_results() -> 搜索结果显示
format_welcome_message() -> /start消息
escape_markdown_v2() -> Telegram MarkdownV2转义
split_long_message() -> 自动分割>4000字符
```

## 成功标准

第7周完成标志：

- ✅ 机器人响应所有命令
- ✅ 自然语言查询返回RAG答案
- ✅ 缓存提供150-400倍加速
- ✅ 设置在会话间持久化
- ✅ 交互式键盘工作
- ✅ Langfuse显示Telegram追踪
- ✅ 错误处理优雅
- ✅ Markdown格式正确渲染
- ✅ 长消息自动分割
- ✅ 多用户可同时使用

---

**第7周将你的RAG系统转变为移动优先、可通过Telegram随时随地访问的对话式研究助手！** 🚀

---

## FAQ

**问：我需要公网IP才能使用Telegram机器人吗？**
答：不需要！轮询模式非常适合开发和低流量部署。Webhook需要HTTPS。

**问：多个用户可以同时使用机器人吗？**
答：可以！每个用户有独立的设置和会话。

**问：运行机器人的成本是多少？**
答：免费！Telegram机器人API完全免费，没有限制。

**问：我可以限制特定用户使用机器人吗？**
答：可以！设置`TELEGRAM__ALLOWED_USER_IDS=123456,789012`，用逗号分隔Telegram用户ID。

**问：如何获取我的Telegram用户ID？**
答：在Telegram上给`@userinfobot`发消息，或发送消息后查看机器人日志。

**问：机器人会存储对话历史吗？**
答：会，存储在内存中，每个用户最近10条消息。生产环境建议迁移到Redis/PostgreSQL。

**问：我可以在没有Docker的服务器上部署吗？**
答：可以！只需安装依赖`uv sync`后运行`python src/main.py`。

**问：如何更新机器人token？**
答：更新`.env`中的`TELEGRAM__BOT_TOKEN`并重启：`docker compose restart api`

**问：我可以自定义机器人的个性吗？**
答：可以！编辑`src/services/ollama/prompts/`中的提示词和`src/services/telegram/formatters.py`中的消息模板。

**问：这适用于其他LLM模型吗？**
答：可以！更改`OLLAMA_MODEL`或使用`/settings`命令选择不同的Ollama模型。

---

享受你的新对话式RAG界面！🎉