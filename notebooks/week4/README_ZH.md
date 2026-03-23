# 第4周：文档分块和混合搜索

## 概述

第4周实现了一个**生产级混合搜索系统**，结合了BM25关键词搜索的精确性和向量嵌入的语义理解。该系统通过智能地将文档分解为可搜索的块并启用多种搜索模式，为检索增强生成（RAG）提供基础。

## 我们构建了什么

### 🧩 **基于章节的文档分块**
- **智能分割**：利用文档结构（解析的章节）作为自然的块边界
- **上下文保留**：块之间100词重叠保持语义连续性
- **自适应处理**：处理结构化（有章节）和非结构化（基于段落）文档
- **最佳大小**：目标600词块，最小100词阈值

### 🔍 **统一混合搜索系统**
- **单索引架构**：一个OpenSearch索引（`arxiv-papers-chunks`）支持所有搜索模式
- **多种搜索类型**：
  - **BM25关键词搜索**：快速（约50ms）传统文本匹配
  - **向量相似度搜索**：使用1024维嵌入的语义搜索
  - **混合搜索**：结合两种方法的RRF（倒数排名融合）
- **生产API**：具有全面验证的RESTful端点`/api/v1/hybrid-search/`

### 🤖 **真实嵌入集成**
- **Jina AI嵌入**：针对检索优化的生产级1024维向量
- **自动生成**：FastAPI端点自动生成查询嵌入
- **回退策略**：当嵌入不可用时优雅降级到BM25
- **性能优化**：高效的嵌入生成和存储

## 架构

### 系统概览

<p align="center">
  <img src="../../static/week4_hybrid_opensearch.png" alt="第4周混合搜索架构" width="800">
  <br>
  <em>完整的第4周架构，展示具有分块、嵌入和RRF融合的混合搜索</em>
</p>

### 数据流
```
原始论文 → PDF解析 → 章节提取 → 分块 → 嵌入 → 索引 → 搜索
```

上图说明了完整的第4周实现：
- **数据处理管道**：arXiv论文通过分块和嵌入生成流转
- **统一OpenSearch索引**：支持BM25、向量和混合搜索模式的单一索引
- **混合检索管道**：结合关键词精确性和语义理解的RRF融合
- **生产API层**：具有自动嵌入生成的FastAPI端点

### 系统组件

#### **1. 文档处理管道**
```python
# 位于：src/services/indexing/text_chunker.py
TextChunker.chunk_paper(
    title="论文标题",
    abstract="摘要文本",
    full_text="完整论文内容",
    sections=parsed_sections_dict,
    target_words=600,
    overlap_words=100
)
```

#### **2. 嵌入服务**
```python
# 位于：src/services/embeddings/factory.py
embeddings_service = make_embeddings_service()
vectors = await embeddings_service.embed_query(["查询文本"])
```

#### **3. 统一搜索客户端**
```python
# 位于：src/services/opensearch/client.py
results = opensearch_client.search_unified(
    query="机器学习",
    query_embedding=vector,
    use_hybrid=True,
    size=10
)
```

#### **4. 生产API**
```python
# 位于：src/routers/hybrid_search.py
POST /api/v1/hybrid-search/
{
  "query": "神经网络",
  "use_hybrid": true,
  "size": 5
}
```

## 关键功能

### **混合搜索模式**

| 模式 | 速度 | 召回率 | 精确率 | 用例 |
|------|--------|--------|-----------|----------|
| **仅BM25** | 约50ms | 高 | 中 | 精确关键词匹配 |
| **仅向量** | 约100ms | 中 | 高 | 语义相似性 |
| **混合（RRF）** | 约2-4s | 高 | 高 | 最佳整体相关性 |

### **RRF（倒数排名融合）**
- **算法**：使用倒数排名融合结合BM25和向量搜索排名
- **实现**：手动融合算法（OpenSearch 2.19兼容性）
- **加权**：可配置的关键词和语义相关性平衡
- **回退**：如果向量搜索失败自动回退到BM25

### **基于章节的分块策略**

```python
# 分块参数（通过测试优化）
CHUNK_SIZE = 600        # 每块目标词数
OVERLAP_SIZE = 100      # 块之间重叠词数
MIN_CHUNK_SIZE = 100    # 最小可行块大小
SECTION_BASED = True    # 可用时使用文档结构
```

**好处**：
- **语义连贯性**：块遵守自然文档边界
- **上下文保留**：重叠防止边界信息丢失
- **检索准确性**：更好地将用户查询匹配到相关内容
- **可扩展性**：处理1,000到100,000+词的文档

## 实现细节

### **OpenSearch索引配置**

**索引名称**：`arxiv-papers-chunks`

**关键字段**：
```json
{
  "arxiv_id": "2508.18563v1",
  "title": "论文标题",
  "chunk_text": "块内容...",
  "chunk_id": "唯一块标识符",
  "section_name": "引言",
  "embedding": [0.123, 0.456, ...],  // 1024维
  "paper_categories": ["cs.AI", "cs.LG"],
  "published_date": "2025-08-25T23:43:33"
}
```

### **搜索查询结构**

**BM25查询**（关键词匹配）：
```json
{
  "query": {
    "bool": {
      "should": [
        {"match": {"chunk_text": {"query": "机器学习", "fuzziness": "AUTO"}}},
        {"match": {"title": {"query": "机器学习", "boost": 2.0}}},
        {"match": {"abstract": {"query": "机器学习", "boost": 1.5}}}
      ]
    }
  }
}
```

**混合查询**（RRF手动融合）：
1. 执行BM25查询 → 获取排名结果
2. 执行向量查询 → 获取排名结果
3. 应用RRF融合算法 → 合并排名
4. 返回带有混合分数的合并结果

### **环境配置**

**必需变量**：
```bash
# 核心服务
POSTGRES_DATABASE_URL=postgresql+psycopg2://rag_user:rag_password@postgres:5432/rag_db
OPENSEARCH__HOST=http://opensearch:9200

# 嵌入（混合搜索必需）
JINA_API_KEY=jina_your_api_key_here

# 分块配置
CHUNKING__CHUNK_SIZE=600
CHUNKING__OVERLAP_SIZE=100
CHUNKING__MIN_CHUNK_SIZE=100
CHUNKING__SECTION_BASED=true

# OpenSearch配置
OPENSEARCH__INDEX_NAME=arxiv-papers
OPENSEARCH__CHUNK_INDEX_SUFFIX=chunks
OPENSEARCH__VECTOR_DIMENSION=1024
```

## API参考

### **混合搜索端点**

**端点**：`POST /api/v1/hybrid-search/`

**请求体**：
```json
{
  "query": "transformer神经网络",
  "use_hybrid": true,
  "size": 10,
  "from": 0,
  "categories": ["cs.AI", "cs.LG"],
  "latest_papers": false,
  "min_score": 0.0
}
```

**响应**：
```json
{
  "query": "transformer神经网络",
  "total": 15,
  "hits": [
    {
      "arxiv_id": "2508.18563v1",
      "title": "论文标题",
      "authors": "作者姓名",
      "abstract": "论文摘要...",
      "score": 0.8542,
      "chunk_text": "相关块内容...",
      "chunk_id": "chunk_uuid",
      "section_name": "相关工作"
    }
  ],
  "size": 10,
  "from": 0,
  "search_mode": "hybrid"
}
```


## 性能基准

**测试环境**：3篇论文，81个块，单节点OpenSearch

| 搜索类型 | 平均响应时间 | 吞吐量 | Recall@10 | Precision@10 |
|-------------|-------------------|------------|-----------|--------------|
| 仅BM25 | 52ms | 约200 req/s | 0.78 | 0.65 |
| 仅向量 | 105ms | 约95 req/s | 0.82 | 0.71 |
| 混合（RRF） | 2.4s | 约25 req/s | 0.89 | 0.84 |

**关键洞察**：
- **混合搜索**以响应时间为代价提供最佳相关性
- **BM25**非常适合高吞吐量关键词匹配
- **向量搜索**具有良好的语义理解能力和适中的速度
- **嵌入生成**约占混合搜索时间的~2s

## 生产部署

### **扩展考量**

**OpenSearch集群**：
```yaml
# 生产推荐最低配置
opensearch:
  image: opensearchproject/opensearch:2.19.0
  environment:
    - cluster.name=rag-cluster
    - node.name=rag-node-1
    - discovery.type=single-node
    - OPENSEARCH_JAVA_OPTS=-Xms1g -Xmx1g
  deploy:
    resources:
      limits:
        memory: 2G
      reservations:
        memory: 1G
```

**嵌入服务优化**：
- **批量处理**：每次API调用处理多个嵌入
- **缓存**：缓存频繁查询的嵌入
- **速率限制**：遵守Jina AI API限制（1000请求/分钟）
- **回退策略**：嵌入不可用时使用仅BM25模式

### **监控和可观测性**

**关键指标**：
- 搜索请求延迟（p50、p95、p99）
- 嵌入生成成功率
- 索引文档数量和大小
- 搜索模式使用分布（BM25 vs 混合）

**健康检查**：
- OpenSearch集群健康
- 嵌入服务可用性
- 索引文档数量验证
- 示例查询执行

## 故障排除

### **常见问题**

**1. 混合搜索返回BM25模式**
```bash
# 检查嵌入服务
curl -X POST "http://localhost:8000/api/v1/hybrid-search/" \
  -H "Content-Type: application/json" \
  -d '{"query": "test", "use_hybrid": true}'

# 查看嵌入错误日志
docker compose logs api | grep -i embedding
```

**2. 空搜索结果**
```bash
# 验证索引存在并有文档
curl "http://localhost:9200/arxiv-papers-chunks/_count"

# 检查索引映射
curl "http://localhost:9200/arxiv-papers-chunks/_mapping"
```

**3. 嵌入生成缓慢**
```bash
# 检查Jina API密钥配置
docker compose exec api env | grep JINA

# 直接测试嵌入服务
curl -X POST "https://api.jina.ai/v1/embeddings" \
  -H "Authorization: Bearer $JINA_API_KEY" \
  -d '{"model": "jina-embeddings-v3", "input": ["test"]}'
```

## 测试

### **运行第4周笔记本**

1. **启动服务**：
```bash
docker compose up --build -d
```

2. **打开笔记本**：
```bash
cd notebooks/week4
jupyter notebook week4_hybrid_search.ipynb
```

3. **执行所有单元格**：笔记本包括：
   - 环境设置和健康检查
   - 基于章节的分块演示
   - 使用Jina AI的真实嵌入生成
   - 所有搜索模式（BM25、向量、混合）
   - 生产API端点测试
   - 性能比较

### **手动测试**

**测试BM25搜索**：
```bash
curl -X POST "http://localhost:8000/api/v1/hybrid-search/" \
  -H "Content-Type: application/json" \
  -d '{"query": "机器学习", "use_hybrid": false, "size": 3}'
```

**测试混合搜索**：
```bash
curl -X POST "http://localhost:8000/api/v1/hybrid-search/" \
  -H "Content-Type: application/json" \
  -d '{"query": "神经网络", "use_hybrid": true, "size": 3}'
```

## 下一步（第5周）

第4周为第5周的LLM集成提供搜索基础：

1. **LLM集成**：连接Ollama进行答案生成
2. **RAG管道**：查询 → 搜索 → 上下文 → 生成 → 响应
3. **上下文管理**：优化检索到的块用于LLM输入
4. **答案质量**：实现引用和来源归属
5. **对话记忆**：支持多轮对话

混合搜索系统**生产就绪**，提供高质量RAG应用所需的检索准确性。

## 文件结构

```
src/
├── routers/
│   └── hybrid_search.py          # FastAPI端点
├── services/
│   ├── opensearch/
│   │   ├── client.py              # 统一搜索客户端
│   │   ├── factory.py             # 客户端工厂
│   │   └── index_config_hybrid.py # 索引配置
│   ├── indexing/
│   │   ├── text_chunker.py        # 基于章节的分块
│   │   ├── hybrid_indexer.py      # 文档索引
│   │   └── factory.py             # 索引服务工厂
│   └── embeddings/
│       ├── jina_client.py         # Jina AI客户端
│       └── factory.py             # 嵌入服务工厂
├── schemas/
│   └── api/
│       └── search.py              # 请求/响应模型
└── config.py                      # 配置管理

notebooks/week4/
├── README.md                      # 本文档
├── week4_hybrid_search.ipynb      # 交互式教程
└── data/                          # 示例数据目录
```

## 资源

- **OpenSearch文档**：https://opensearch.org/docs/
- **Jina AI嵌入**：https://jina.ai/embeddings/
- **FastAPI文档**：https://fastapi.tiangolo.com/
- **倒数排名融合论文**：https://plg.uwaterloo.ca/~gvcormac/cormacksigir09-rrf.pdf
- **第4周笔记本**：[week4_hybrid_search.ipynb](./week4_hybrid_search.ipynb)