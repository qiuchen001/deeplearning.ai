# L4: 多模态向量存储检索 (Multimodal Retrieval from Vector Stores)

## 课程目标

本课程介绍如何使用向量存储进行多模态检索，包括:
1. 配置和使用LanceDB向量数据库
2. 处理和存储多模态数据
3. 使用Langchain进行检索
4. 实现和优化检索效果

## 一、环境配置

### 1.1 依赖库
```python
from mm_rag.embeddings.bridgetower_embeddings import BridgeTowerEmbeddings
from mm_rag.vectorstores.multimodal_lancedb import MultimodalLanceDB
import lancedb
import json
import os
from PIL import Image
```

### 1.2 LanceDB配置
```python
# 配置LanceDB
LANCEDB_HOST_FILE = "./shared_data/.lancedb"
TBL_NAME = "test_tbl"
db = lancedb.connect(LANCEDB_HOST_FILE)
```

## 二、数据准备与处理

### 2.1 加载数据
```python
# 加载元数据
vid1_metadata = load_json_file(vid1_metadata_path)
vid2_metadata = load_json_file(vid2_metadata_path)

# 提取文本和图像路径
vid1_trans = [vid['transcript'] for vid in vid1_metadata]
vid1_img_path = [vid['extracted_frame_path'] for vid in vid1_metadata]
```

### 2.2 文本增强
- 处理片段化的转录文本
- 合并相邻帧的文本
- 创建有意义的文本描述

```python
# 使用滑动窗口合并文本
n = 7  # 窗口大小
updated_vid1_trans = [
    ' '.join(vid1_trans[i-int(n/2) : i+int(n/2)]) 
    for i in range(len(vid1_trans))
]
```

## 三、向量存储实现

### 3.1 初始化嵌入模型
```python
embedder = BridgeTowerEmbeddings()
```

### 3.2 数据导入
```python
vectorstore = MultimodalLanceDB.from_text_image_pairs(
    texts=updated_vid1_trans+vid2_trans,
    image_paths=vid1_img_path+vid2_img_path,
    embedding=embedder,
    metadatas=vid1_metadata+vid2_metadata,
    connection=db,
    table_name=TBL_NAME,
    mode="overwrite", 
)
```

## 四、检索实现

### 4.1 创建检索器
```python
vectorstore = MultimodalLanceDB(
    uri=LANCEDB_HOST_FILE, 
    embedding=embedder, 
    table_name=TBL_NAME
)

retriever = vectorstore.as_retriever(
    search_type='similarity', 
    search_kwargs={"k": 1}
)
```

### 4.2 检索示例
```python
# 示例查询
query1 = "a toddler and an adult"
results = retriever.invoke(query1)

# 返回多个结果
retriever = vectorstore.as_retriever(
    search_type='similarity', 
    search_kwargs={"k": 3}
)
```

## 五、检索优化

### 5.1 文本处理优化
- 合并相邻帧文本提高语义完整性
- 避免过长或过短的文本片段
- 保持文本的上下文连贯性

### 5.2 检索参数调优
- 调整返回结果数量(k值)
- 选择合适的相似度计算方法
- 优化检索阈值

## 六、注意事项

1. 数据质量:
   - 确保文本描述的质量
   - 处理不完整或噪声文本
   - 维护图文对的对应关系

2. 性能考虑:
   - 向量存储的规模
   - 检索速度优化
   - 内存使用管理

3. 检索效果:
   - 结果相关性
   - 召回率和精确度
   - 排序质量

## 七、应用场景

1. 视频内容检索:
   - 场景搜索
   - 内容推荐
   - 相似片段查找

2. 交互式应用:
   - 多模态问答
   - 内容分析
   - 视频摘要

## 八、总结

本课程展示了:
1. 如何构建多模态向量存储系统
2. 实现高效的多模态检索
3. 优化检索效果和性能

这些技术为构建高质量的多模态RAG系统提供了重要支持。

## 九、扩展阅读

1. 向量数据库:
   - LanceDB文档
   - 其他向量数据库对比

2. 检索技术:
   - 向量检索算法
   - 多模态检索优化
   - 相似度计算方法 