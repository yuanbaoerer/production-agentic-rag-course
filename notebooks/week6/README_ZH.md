# 第6周：使用Langfuse和Redis进行生产监控和缓存

## 概述

第6周为我们的RAG系统添加生产级监控和智能缓存。我们集成Langfuse实现完整的管道可观测性，集成Redis实现高性能响应缓存。

## 我们构建了什么

- **Langfuse集成**：端到端RAG管道追踪和分析
- **Redis缓存**：重复查询响应速度提升150-400倍
- **性能监控**：实时指标和系统健康
- **生产就绪**：企业级可观测性和优化

## 架构

<p align="center">
  <img src="../../static/week6_monitoring_and_caching.png" alt="第6周监控和缓存架构" width="900">
  <br>
  <em>第6周架构，展示Langfuse追踪和Redis缓存集成</em>
</p>

### 数据流
```
查询 → 缓存检查 → [命中：约100ms] | [未命中：完整管道约15s] → 缓存存储 → Langfuse追踪
```

## 关键功能

### **Langfuse可观测性**
- 具有性能分解的完整RAG管道追踪
- 用户分析、查询模式和成功率追踪
- 具有成本和使用指标的实时监控仪表板
- 具有答案相关性和来源归属的质量洞察

### **Redis智能缓存**
- **精确匹配策略**：参数感知的缓存键实现精确匹配
- **性能**：重复查询响应速度提升150-400倍（约100ms vs 15-20s）
- **TTL管理**：默认24小时过期，可配置设置
- **未来增强**：可升级为语义相似度缓存实现模糊匹配

## 快速开始

### 环境设置
```bash
# 必需的环境变量
LANGFUSE__SECRET_KEY=sk_lf_your_secret_key
LANGFUSE__PUBLIC_KEY=pk_lf_your_public_key
REDIS__HOST=redis
REDIS__TTL_HOURS=24
```

### 启动服务
```bash
docker compose up --build -d
```

### 测试缓存性能
```bash
# 第一次请求（缓存未命中约15-20s）
curl -X POST "http://localhost:8000/api/v1/ask" \
  -H "Content-Type: application/json" \
  -d '{"query": "什么是transformers？", "top_k": 3}'

# 第二次相同请求（缓存命中约100ms）
curl -X POST "http://localhost:8000/api/v1/ask" \
  -H "Content-Type: application/json" \
  -d '{"query": "什么是transformers？", "top_k": 3}'
```

## 性能基准

| 场景 | 响应时间 | 改善 |
|----------|---------------|-------------|
| **缓存未命中** | 15-20秒 | 基线 |
| **缓存命中** | 50-100ms | **快150-400倍** |
| **监控开销** | <2% | 可忽略影响 |

## 测试

### 运行笔记本
```bash
jupyter notebook notebooks/week6/week6_cache_testing.ipynb
```

### 监控系统健康
```bash
# 检查Redis连接
redis-cli ping

# 查看缓存统计
curl "http://localhost:8000/api/v1/health"

# 访问Langfuse仪表板
# 访问：https://cloud.langfuse.com（或你的自托管实例）
```

## 故障排除

| 问题 | 解决方案 |
|-------|----------|
| **缓存不工作** | 检查Redis：`redis-cli ping` |
| **没有Langfuse追踪** | 验证环境变量：`LANGFUSE__*` |
| **响应缓慢** | 监控缓存命中率和系统资源 |

## 下一步

- **增强缓存**：升级为语义相似度缓存实现模糊匹配
- **高级分析**：自定义仪表板和A/B测试框架
- **生产扩展**：分布式缓存和自动监控
- **质量优化**：用户反馈集成和答案评分

## 资源

- **笔记本**：[week6_cache_testing.ipynb](./week6_cache_testing.ipynb)
- **Langfuse仪表板**：https://cloud.langfuse.com
- **Redis文档**：https://redis.io/docs

---

第6周将你的RAG系统转变为生产级服务，性能提升150-400倍并具有全面的可观测性。