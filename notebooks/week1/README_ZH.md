# 第1周：基础设施设置和验证

此文件夹包含arXiv论文管理器项目第1周的材料，重点在于设置和验证完整的基础设施栈。

## 内容

### `week1_setup.ipynb`
一个全面的Jupyter笔记本，指导学生完成：

1. **系统要求和设置**
   - 了解每个技术组件及其用途
   - 跨平台安装说明（Windows、macOS、Linux）
   - 具有自动检查的先决条件验证

2. **基础设施架构**
   - 多服务架构的完整概述
   - 了解Docker容器如何通信
   - 数据持久化和卷管理概念

测试git提交

<p align="center">
  <img src="../../static/week1_infra_setup.png" alt="第1周基础设施设置" width="700">
</p>

**架构概览：**
- **FastAPI**（端口8000）：具有异步支持和自动文档的REST API
- **PostgreSQL 16**（端口5432）：论文元数据和内容存储的主数据库
- **OpenSearch 2.19**（端口9200、5601）：具有管理仪表板的混合搜索引擎
- **Apache Airflow 3.0**（端口8080）：具有DAG和PostgreSQL后端的工作流编排
- **Ollama**（端口11434）：用于未来RAG实现的本地LLM服务器
- **Docker网络**：所有服务通过`rag-network`通信，具有持久化卷

3. **逐服务设置**
   - 用于论文元数据存储的PostgreSQL数据库
   - 用于全文搜索功能的OpenSearch
   - 用于工作流自动化的Apache Airflow
   - 用于本地LLM推理的Ollama
   - 用于REST API端点的FastAPI

4. **验证和测试**
   - 所有服务的自动健康检查
   - 逐步验证程序
   - 模块化Ollama测试（4个专注的测试单元）
   - 常见故障排除场景和解决方案

## 学习目标

完成本周材料后，学生将：

- 理解容器化和Docker Compose编排
- 学习如何设置生产级基础设施栈
- 获得数据库设计和API开发经验
- 掌握多服务应用程序的故障排除技术
- 学习直接HTTP API测试与服务抽象层
- 建立使用专业开发工具的信心

## Ollama测试（第1周简化版）

笔记本包含分解为专注单元的模块化Ollama测试：

- **测试3A**：检查可用模型
- **测试3B**：简单模型测试（如果已安装模型）
- **测试3C**：性能分析
- **测试3D**：学习笔记和设置命令

### 简易模型安装（第1周可选）

```bash
# 使用Makefile（推荐）
make ollama-pull MODEL=llama3.2:1b
make ollama-test MODEL=llama3.2:1b

# 用于学习的直接HTTP调用
curl -X POST http://localhost:11434/api/pull -d '{"name":"llama3.2:1b"}'
curl -X POST http://localhost:11434/api/generate -d '{"model":"llama3.2:1b","prompt":"Hello","stream":false}'
```

### 课程推荐模型

- **llama3.2:1b**（1.2GB）- 快速，适合测试
- **llama3.2:3b**（2.0GB）- 速度/质量平衡
- **llama3.1:8b**（4.7GB）- 更好质量，较慢

**注意**：第1周不需要任何模型 - 没有它们服务健康检查也能工作。

## 目标受众

本材料适用于：
- **初学者**，想学习现代软件基础设施
- **学生**，想了解现实世界应用程序是如何构建的
- **专业人士**，正在转型到软件开发或DevOps
- **任何人**，对构建自己的AI驱动研究工具感兴趣

## 时间投入

- **设置**：2-3小时（包括软件安装和下载）
- **笔记本完成**：1小时
- **总计**：2-4小时

## 📖 额外资源

**第1周博客文章：** [驱动RAG系统的基础设施](https://jamwithai.substack.com/p/the-infrastructure-that-powers-rag)
- 深入了解每个基础设施组件
- 生产部署注意事项
- 架构决策说明

## 支持资源

如果遇到问题：
1. 查看笔记本中的故障排除章节
2. 查看常见问题和解决方案
3. 确保所有先决条件已正确安装
4. 遵循逐步验证程序
5. 在Jam With AI substack聊天频道提问

## 下一步

完成第1周后，你将准备好：
- 理解每个服务如何为整体系统做出贡献
- 根据需要修改和扩展基础设施
- 进入第2周：arXiv集成和PDF处理
- 建立在专业开发环境中工作的信心