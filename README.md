# 智扫通机器人智能客服

基于 LangChain 和 ChromaDB 的智能扫地机器人客服 Agent 系统，支持 RAG 知识检索、工具调用和多轮对话，提供 Web 界面和命令行两种交互方式。

## 功能特性

- **智能问答**：基于 RAG（检索增强生成）的扫地机器人知识库检索，支持故障排除、维护保养、选购指南等专业问答
- **个性化报告生成**：自动获取用户数据并生成个性化使用报告，包含清洁效率、耗材状态、使用对比等维度
- **多工具协同**：
  - RAG 知识检索（rag_summarize）
  - 天气查询（get_weather）
  - 用户信息获取（get_user_id, get_user_location, get_current_month）
  - 外部数据检索（fetch_external_data）
  - 上下文注入（fill_context_for_report）
- **流式对话**：支持 Streamlit Web 界面实时流式响应
- **日志监控**：完整的工具调用监控和日志记录
- **提示词切换**：支持动态提示词切换的中间件机制

## 技术栈

| 类别 | 技术 |
|------|------|
| **核心框架** | LangChain, LangGraph |
| **向量数据库** | ChromaDB |
| **大模型** | 通义千问（qwen3-max） |
| **Embedding** | 通义千问（text-embedding-v4） |
| **Web 界面** | Streamlit |
| **文档处理** | PyPDF, LangChain Document Loaders |
| **配置管理** | YAML |
| **日志系统** | Python logging |

## 项目结构

```
agent-project/
├── agent/                          # Agent 核心模块
│   ├── react_agent.py              # ReAct Agent 实现（思考-行动-观察循环）
│   └── tools/                      # 工具定义和中间件
│       ├── agent_tools.py          # 工具函数实现
│       └── middleware.py           # 中间件（监控、日志、提示词切换）
│
├── rag/                            # RAG（检索增强生成）模块
│   ├── rag_service.py              # RAG 服务（检索、总结、生成）
│   └── vector_store.py             # 向量存储管理（文档加载、分片、存储）
│
├── model/                          # 模型工厂
│   └── factory.py                  # 聊天模型和 Embedding 模型工厂
│
├── config/                         # 配置文件
│   ├── agent.yml                   # Agent 配置（外部数据路径等）
│   ├── chroma.yml                  # ChromaDB 配置（集合名、分片参数等）
│   ├── rag.yml                     # RAG 配置（模型名称等）
│   └── prompts.yml                 # 提示词路径配置
│
├── prompts/                        # 提示词模板
│   ├── main_prompt.txt             # 主提示词（ReAct Agent 指令）
│   ├── rag_summarize.txt           # RAG 总结提示词
│   └── report_prompt.txt           # 报告生成提示词
│
├── data/                           # 知识库数据
│   ├── external/                   # 外部数据（用户使用记录）
│   │   └── records.csv             # 用户月度使用记录
│   ├── 扫地机器人100问.pdf          # 知识库文档
│   ├── 扫地机器人100问2.txt         # 知识库文档
│   ├── 扫拖一体机器人100问.txt       # 知识库文档
│   ├── 故障排除.txt                 # 知识库文档
│   ├── 维护保养.txt                 # 知识库文档
│   └── 选购指南.txt                 # 知识库文档
│
── utils/                          # 工具函数
│   ├── config_handler.py           # 配置文件加载器
│   ├── file_handler.py             # 文件处理（PDF、TXT 加载器）
│   ├── logger_handler.py           # 日志处理器
│   ├── path_tool.py                # 路径工具（绝对路径转换）
│   └── prompt_loader.py            # 提示词加载器
│
├── logs/                           # 日志文件（按日期记录）
│   ├── agent_20260605.log
│   ── agent_20260606.log
│
├── md5.txt                         # 知识库文件 MD5 去重记录
├── app.py                          # Streamlit Web 应用
└── .gitignore                      # Git 忽略文件配置
```

## 快速开始

### 环境要求

- Python >= 3.10
- pip >= 21.0

### 安装依赖

```bash
pip install -r requirements.txt
```

如果没有 requirements.txt，请安装以下核心依赖：

```bash
pip install langchain langchain-chroma langchain-community langgraph \
            streamlit chromadb pypdf pyyaml dashscope
```

### 配置模型

在 `config/rag.yml` 中配置通义千问 API 密钥（环境变量）：

```bash
# Windows PowerShell
$env:DASHSCOPE_API_KEY="your-api-key-here"

# Linux/macOS
export DASHSCOPE_API_KEY="your-api-key-here"
```

### 初始化知识库

首次使用需要加载知识库文档到向量数据库：

```bash
python rag/vector_store.py
```

这会将 `data/` 目录下的所有 `.txt` 和 `.pdf` 文件进行：
1. 文档加载（支持 PDF 和 TXT）
2. MD5 去重检查
3. 文本分片（chunk_size=200, chunk_overlap=20）
4. Embedding 向量化
5. 存储到 ChromaDB

### 运行方式

#### 方式一：Web 界面（推荐）

```bash
streamlit run app.py
```

访问 http://localhost:8501 即可使用智能客服界面。

#### 方式二：命令行测试

```bash
# 测试 ReAct Agent
python agent/react_agent.py

# 测试 RAG 服务
python rag/rag_service.py

# 测试向量库
python rag/vector_store.py
```

### 使用示例

#### 1. 专业问答

```
用户：小户型适合哪些扫地机器人？
AI：[基于 RAG 检索知识库，返回专业选购建议]
```

#### 2. 天气适配建议

```
用户：扫地机器人在我所在的地区的气温下如何保养？
AI：[自动获取用户位置 → 查询天气 → 基于环境给出保养建议]
```

#### 3. 生成使用报告

```
用户：生成我的使用报告
AI：[调用 fill_context_for_report → 获取用户ID → 获取月份 → 查询使用记录 → 生成个性化报告]
```

报告包含以下维度：
- 用户基本情况（居住环境、宠物、地毯类型）
- 使用效率分析（毛发清理效率、地毯增压模式使用）
- 耗材状态（胶刷寿命、尘盒清理频率）
- 专业建议（保养方法、清洁优化、宠物家庭建议）

## 配置说明

### ChromaDB 配置（config/chroma.yml）

```yaml
collection_name: agent              # 集合名称
persist_directory: rag/chroma_db    # 向量库持久化路径
k: 3                                # 检索返回的最相似文档数量
data_path: data                     # 知识库文档路径
md5_hex_store: md5.txt              # MD5 去重记录文件
allow_knowledge_file_type: ["txt","pdf"]  # 允许的文件类型

chunk_size: 200                     # 文本分片大小
chunk_overlap: 20                   # 分片重叠大小
separators: ["\n\n","。",".","?","？","!"," ",""]  # 分片分隔符
```

### RAG 配置（config/rag.yml）

```yaml
chat_model_name: qwen3-max           # 聊天模型名称
embedding_model_name: text-embedding-v4  # Embedding 模型名称
```

### Agent 配置（config/agent.yml）

```yaml
external_data_path: data/external/records.csv  # 外部用户数据路径
```

## 核心模块说明

### 1. ReAct Agent（agent/react_agent.py）

实现思考-行动-观察循环机制：

```python
思考 → 调用工具 → 观察结果 → 再思考 → 最终回答
```

**中间件**：
- `monitor_tool`: 监控工具调用（工具名、参数、结果）
- `log_before_model`: 记录每次模型调用前的消息数量
- `report_prompt_switch`: 报告生成场景的提示词切换

### 2. RAG 服务（rag/rag_service.py）

提供基于向量检索的智能问答：

1. **检索**：根据用户查询从 ChromaDB 检索最相关的 k 个文档
2. **构建上下文**：将检索结果格式化为参考资料
3. **生成**：结合用户问题和参考资料，使用 LLM 生成回答

### 3. 工具集（agent/tools/agent_tools.py）

| 工具名 | 功能 | 入参 | 出参 |
|--------|------|------|------|
| rag_summarize | RAG 知识检索 | query: str | str |
| get_weather | 天气查询 | city: str | str |
| get_user_location | 获取用户城市 | 无 | str |
| get_user_id | 获取用户ID | 无 | str |
| get_current_month | 获取当前月份 | 无 | str |
| fetch_external_data | 查询使用记录 | user_id, month | str |
| fill_context_for_report | 注入报告上下文 | 无 | str |

### 4. 向量存储（rag/vector_store.py）

管理知识库文档的生命周期：

- **load_document()**: 加载文档到向量库
  - 读取 `data/` 目录下的文件
  - MD5 去重（避免重复加载）
  - 文本分片（RecursiveCharacterTextSplitter）
  - Embedding 向量化
  - 存储到 ChromaDB
- **get_retriever()**: 获取检索器（用于 RAG 服务）

## 数据格式

### 外部数据（data/external/records.csv）

用户月度使用记录，格式示例：

```csv
user_id,feature,efficiency,consumables,comparison,month
1001,"90㎡|1狗|短毛地毯","毛发清理:95%\n地毯增压使用:20次/月","胶刷寿命:剩余30天\n尘盒清理:每2天","毛发处理效率前5%",2025-12
```

字段说明：
- `user_id`: 用户ID
- `feature`: 特征（面积、宠物、地毯类型）
- `efficiency`: 效率（清理效率、模式使用频次）
- `consumables`: 耗材状态（寿命、清理频率）
- `comparison`: 对比数据（排名、百分比）
- `month`: 月份（YYYY-MM 格式）

### 知识库文档（data/*.txt, data/*.pdf）

支持以下类型文档：
- 产品 FAQ（扫地机器人100问、扫拖一体机器人100问）
- 故障排除指南
- 维护保养手册
- 选购指南

## 常见问题

### 1. RAG 检索无结果

**原因**：知识库未加载或查询内容不在知识范围内

**解决**：
```bash
# 1. 确认 data/ 目录下有相关文档
# 2. 执行知识库加载
python rag/vector_store.py
# 3. 检查日志是否有加载成功记录
```

### 2. ChromaDB 路径问题

**症状**：不同模块检索结果不一致

**原因**：`persist_directory` 使用了相对路径

**解决**：确保 `vector_store.py` 中使用 `get_abs_path()` 转换路径

### 3. 模型调用失败

**原因**：未配置 DASHSCOPE_API_KEY 环境变量

**解决**：
```bash
# Windows
$env:DASHSCOPE_API_KEY="your-api-key"

# Linux/macOS
export DASHSCOPE_API_KEY="your-api-key"
```

### 4. PDF 加载失败

**原因**：PyPDFLoader 不支持加密 PDF

**解决**：移除 PDF 密码或使用其他加载器

### 5. RagSummarizeService 调用错误

**症状**：`TypeError: rag_summarize() missing 1 required positional argument: 'query'`

**原因**：直接使用类名调用而非实例

**解决**：
```python
# ❌ 错误
RagSummarizeService.rag_summarize(query)

# ✅ 正确
rag = RagSummarizeService()
rag.rag_summarize(query)
```

## 日志说明

日志文件位于 `logs/` 目录，按日期命名（如 `agent_20260606.log`）

**日志级别**：
- `INFO`: 工具调用、模型调用、知识库加载
- `WARNING`: 检索无结果、数据缺失
- `ERROR`: 加载失败、异常堆栈

**日志示例**：
```
2026-06-06 15:44:11,123 - agent - INFO - [monitor_tool]执行工具: rag_summarize
2026-06-06 15:44:11,123 - agent - INFO - [monitor_tool]参入参数: {'query': '毛发清理'}
2026-06-06 15:44:14,526 - agent - INFO - [monitor_tool]工具rag_summarize调用成功
```

## 开发指南

### 添加新工具

1. 在 `agent/tools/agent_tools.py` 中定义工具函数：

```python
@tool(description="工具描述")
def new_tool(param1: str) -> str:
    """工具实现"""
    return f"结果: {param1}"
```

2. 在 `agent/react_agent.py` 中注册工具：

```python
from agent.tools.agent_tools import new_tool

self.agent = create_agent(
    ...
    tools=[..., new_tool],
    ...
)
```

3. 在 `prompts/main_prompt.txt` 中添加工具使用说明

### 添加知识库文档

1. 将文档放入 `data/` 目录（支持 .txt 和 .pdf）
2. 重新加载知识库：
```bash
python rag/vector_store.py
```

### 修改提示词

提示词文件位于 `prompts/` 目录：
- `main_prompt.txt`: ReAct Agent 主提示词
- `rag_summarize.txt`: RAG 总结提示词
- `report_prompt.txt`: 报告生成提示词

修改后无需重启服务，系统自动加载。

## 部署建议

### 开发环境

```bash
streamlit run app.py
```

### 生产环境

1. **使用 gunicorn 运行 Streamlit**（不推荐，Streamlit 有内置服务器）
2. **使用 Docker 容器化部署**
3. **配置环境变量**：
   - `DASHSCOPE_API_KEY`: 通义千问 API 密钥
   - `STREAMLIT_SERVER_PORT`: 服务端口
   - `STREAMLIT_SERVER_ADDRESS`: 服务地址

### 性能优化

1. **知识库预加载**：启动时预先加载向量库
2. **缓存机制**：缓存频繁查询的结果
3. **异步处理**：使用异步模型调用
4. **向量库优化**：调整 `chunk_size` 和 `k` 参数

## 许可证

MIT License

## 联系方式

如有问题或建议，欢迎提交 Issue 或 Pull Request。

---

**项目版本**: v1.0.0  
**最后更新**: 2026-06-06  
**维护者**: qcyn6
