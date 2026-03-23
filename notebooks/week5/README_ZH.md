# 第5周：具有LLM集成的完整RAG系统

## 概述

第5周通过集成Ollama LLM和混合搜索完成了我们的**生产级RAG系统**。系统实现了**6倍更快的性能**（120s → 15-20s）、实时流式响应，并包含Gradio Web界面。

## 我们构建了什么

- **本地LLM集成**：具有llama3.2模型的Ollama服务
- **性能优化**：提示词减少80%，速度提升6倍
- **流式API**：通过服务器发送事件实现实时响应
- **Gradio界面**：具有流式支持的交互式Web UI
- **生产就绪**：整洁的API设计，两个专注的端点

## 架构

<p align="center">
  <img src="../../static/week5_rag_architecture.png" alt="第5周完整RAG系统架构" width="900">
  <br>
  <em>具有LLM生成层（Ollama）、混合检索管道和Gradio界面的完整RAG系统</em>
</p>

## 快速开始

### 1. 启动服务
```bash
docker compose up --build -d
```

### 2. 测试RAG端点
```bash
curl -X POST "http://localhost:8000/api/v1/ask" \
  -H "Content-Type: application/json" \
  -d '{"query": "什么是transformers？", "top_k": 3, "use_hybrid": true}'
```

### 3. 启动Gradio界面
```bash
uv run python gradio_launcher.py
# 打开 http://localhost:7861
```

## API端点

### 标准RAG - `/api/v1/ask`
- **用途**：带有元数据的完整响应
- **响应时间**：15-20秒
- **用例**：批处理、API集成

### 流式RAG - `/api/v1/stream`
- **用途**：实时token生成
- **首个token时间**：2-3秒
- **用例**：交互式UI、更好用户体验

### 请求格式
```json
{
    "query": "你的问题",
    "top_k": 3,              // 检索的块数（1-10）
    "use_hybrid": true,      // BM25 + 向量搜索
    "model": "llama3.2:1b",  // LLM模型
    "categories": ["cs.AI"]  // 可选过滤
}
```

## 性能

| 配置 | 响应时间 | 用例 |
|--------------|---------------|----------|
| `top_k=1, BM25` | 约2.4s | 快速答案 |
| `top_k=3, 混合` | 约15-20s | 平衡质量 |
| `top_k=5, 混合` | 约25-30s | 全面 |

**关键优化**：
- 移除冗余元数据（提示词减少80%）
- 共享代码架构（DRY原则）
- 300词响应限制以获得专注答案
- 自动来源去重

## 配置

```bash
# .env文件
OLLAMA_HOST=http://ollama:11434
OLLAMA__DEFAULT_MODEL=llama3.2:1b
JINA_API_KEY=your_key_here  # 用于嵌入
```

## 测试

### 运行笔记本
```bash
jupyter notebook notebooks/week5/week5_complete_rag_system.ipynb
```

### 测试流式
```bash
curl -X POST "http://localhost:8000/api/v1/stream" \
  -H "Content-Type: application/json" \
  -d '{"query": "解释注意力机制", "top_k": 2}' \
  --no-buffer
```

## 故障排除

| 问题 | 解决方案 |
|-------|----------|
| `/stream`返回404 | 重建API：`docker compose build api && docker compose restart api` |
| 响应缓慢 | 使用更小的模型：`llama3.2:1b`或减少`top_k` |
| 没有Gradio | 端口改为7861：`http://localhost:7861` |
| Ollama错误 | 检查服务：`docker exec rag-ollama ollama list` |

## 项目结构

```
src/
├── routers/
│   └── ask.py              # RAG端点
├── services/
│   └── ollama/
│       ├── client.py       # LLM客户端
│       └── prompts/        # 系统提示词
├── gradio_app.py           # Web界面
└── gradio_launcher.py      # 启动脚本

notebooks/week5/
├── README.md               # 本文件
└── week5_complete_rag_system.ipynb
```

## 下一步

- **增强**：添加对话记忆、反馈循环
- **优化**：实现缓存、微调模型
- **部署**：添加认证、监控、负载均衡

## 资源

- [笔记本教程](./week5_complete_rag_system.ipynb)
- [API文档](http://localhost:8000/docs)
- [Gradio界面](http://localhost:7861)
- [Ollama模型](https://ollama.ai/library)