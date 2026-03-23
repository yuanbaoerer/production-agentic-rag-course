# Airflow配置

此目录包含arXiv论文管理器项目的Apache Airflow配置和DAG。

<p align="center">
  <img src="../static/week2_data_ingestion_flow.png" alt="第2周数据摄入架构" width="800">
</p>

## 当前设置（第2周）

### 生产就绪的DAG
- **hello_world_dag.py**：第1周的基本健康检查DAG
- **arxiv_paper_ingestion.py**：用于自动arXiv论文获取和处理的主生产DAG

### 生产管道特性
- **每日arXiv摄入**：自动获取CS.AI论文
- **PDF处理**：使用Docling下载和解析论文
- **数据库存储**：在PostgreSQL中存储完整的论文元数据和内容
- **错误处理**：全面的重试逻辑和错误报告
- **跨平台兼容性**：适用于macOS、Linux、WSL和Ubuntu

## 目录结构

```
airflow/
├── README.md                           # 本文件
├── Dockerfile                          # 具有依赖项的自定义Airflow容器
├── requirements-airflow.txt            # DAG的Python依赖项
└── dags/
    ├── hello_world_dag.py             # 第1周健康检查DAG
    ├── arxiv_paper_ingestion.py       # 第2周生产摄入DAG
    └── arxiv_ingestion/
        └── tasks.py                   # 具有异步处理的生产管道任务
```

## Docker配置

### 跨平台兼容性
Airflow容器配置为跨平台部署：
- **用户配置**：以`airflow`用户（50000:0）运行以避免权限问题
- **卷管理**：使用命名卷存储日志以防止绑定挂载冲突
- **数据库集成**：连接到共享PostgreSQL实例
- **服务依赖**：自动初始化和健康检查

### 容器特性
- **Python 3.12** 和 Apache Airflow 2.10.3
- **PostgreSQL支持** 通过 psycopg2
- **PDF处理** 使用Docling、Tesseract OCR和Poppler工具
- **速率限制** 和重试逻辑以符合arXiv API规范
- **异步处理** 实现最佳并发下载和解析性能

## 使用方法

### Web界面
- **URL**：http://localhost:8080
- **凭据**：容器初始化期间自动生成
- **功能**：DAG监控、任务日志、管道统计

### 生产DAG（`arxiv_paper_ingestion`）
1. **环境设置**：验证服务并初始化缓存
2. **每日论文获取**：检索前一天的论文（默认10篇）
3. **PDF处理**：使用Docling下载和解析PDF
4. **失败PDF重试**：处理任何处理失败
5. **数据库存储**：存储带有解析内容的完整论文数据
6. **OpenSearch占位符**：为第3周+搜索索引做准备
7. **每日报告**：生成全面的处理统计

### 管道性能
- **并发处理**：5个并行下载，1个解析操作（针对笔记本电脑优化）
- **速率限制**：遵守arXiv API指南（3秒延迟）
- **缓存**：PDF文件本地缓存以避免重复下载
- **错误恢复**：即使单个论文失败也继续处理

## 配置

### 环境变量
```bash
AIRFLOW__DATABASE__SQL_ALCHEMY_CONN=postgresql+psycopg2://rag_user:rag_password@postgres:5432/rag_db
AIRFLOW__CORE__EXECUTOR=LocalExecutor
POSTGRES_DATABASE_URL=postgresql+psycopg2://rag_user:rag_password@postgres:5432/rag_db
PYTHONPATH=/opt/airflow/src
```

### 服务依赖
- **PostgreSQL**：论文元数据和内容存储
- **源代码**：从`../src`挂载以访问服务
- **共享网络**：与API和数据库服务通信

## 第2周实现状态

### ✅ 已完成功能
- 具有所有依赖项的自定义Docker容器
- 具有全面错误处理的生产arXiv摄入DAG
- 具有并发控制的异步PDF处理管道
- 具有完整内容存储的PostgreSQL集成
- 跨平台兼容性（macOS、Linux、WSL、Ubuntu）
- 速率限制和重试逻辑以符合arXiv API规范
- 整个管道的详细日志记录和监控

### 🔄 第3周+路线图
- **OpenSearch集成**：真正的搜索索引（当前为占位符）
- **高级调度**：多种收集策略
- **监控和告警**：生产可观测性
- **规模优化**：更高的并发以应对生产工作负载