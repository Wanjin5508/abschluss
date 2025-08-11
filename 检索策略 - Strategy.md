# 一、检索目标与研究问题

- **总体目标**：系统性整合“Retrieval-Augmented Generation”与“LLM-Based Agent”领域的核心理论、架构、平台及最佳实践。
    
- **具体研究问题**
    
    1. RAG 技术的发展脉络、核心组件及主流架构有哪些？
        
    2. 智能 Agent 的定义、分类与主要实现框架有哪些？
        
    3. RAG 与 Agent 在原理与技术上有何异同？如何协同？
        
    4. 企业中 RAG-Agent 的典型应用场景、价值和挑战是什么？

# 二、检索数据库与工具
覆盖计算机科学、人工智能与信息系统的核心平台：

1. Google Scholar
2. **IEEE Xplore**
3. **ACM Digital Library**
4. ~~Scopus 或 Web of Science~~
5. **arXiv**


# 三、检索词与布尔组合
- 先限定 **Title/Abstract**，年份设 **2020–2025（优先近两年）**；命中不足再放宽到全文。
- 行业词（high voltage / partial discharge / HFCT）建议作为**可选附加**，先不加以免过窄。
- 如果噪声多，可在英文库里加：`NOT (healthcare OR education OR robot)`

### **S1｜单文档手册的 RAG 文档问答（含版面/表格/图片要素）**  

`(retrieval-augmented generation OR RAG ) AND ("document question answering" OR "manual question answering" OR "user guide QA") AND (PDF OR "manual" OR "user guide" OR datasheet)`

(检索增强生成 OR RAG OR 开卷问答) AND (文档问答 OR 手册问答 OR 用户指南 问答) AND (PDF OR 技术手册 OR 用户指南 OR 规格书) AND (版面 OR 版面感知 OR 文档解析 OR 表格抽取 OR 图题 OR OCR) NOT (医疗 OR 教育 OR 机器人)


### **S2｜多模态文档理解（图片/图示/表格）+ 检索增强（仍以文本问答为输入）**  

`(("document VQA" OR DocVQA OR "visual document understanding" OR "diagram understanding" OR "chart question answering") AND (PDF OR manual OR "user guide" OR datasheet) AND (figure OR image OR diagram OR chart OR table)) AND (retrieval OR RAG OR "hybrid search")`  


`((文档VQA OR DocVQA OR 可视文档理解 OR 图示理解 OR 图表问答) AND (PDF OR 手册 OR 用户指南 OR 规格书) AND (图片 OR 图示 OR 图表 OR 表格)) AND (检索 OR RAG OR 混合检索)`

### **S3｜轻量 Agent 路由/工具调用 与 RAG 协同（单文档、小型流水线/演示）**  

`(("tool use" OR "function calling" OR routing OR ReAct OR planner) AND ("document QA" OR "manual QA")) AND ("retrieval-augmented generation" OR RAG)`  


`(工具调用 OR 函数调用 OR 路由 OR ReAct OR 策划) AND (文档问答 OR 手册问答) AND (检索增强生成 OR RAG) AND (流程 OR 流水线 OR 单文档 OR 演示 OR 小规模)`


###  (optional) **S4｜文档/手册问答的域适配与轻量微调（LoRA/指令微调/继续预训练）**

`(("retrieval-augmented generation" OR RAG) AND ("document QA" OR "manual QA") AND (PDF OR "user guide" OR datasheet OR specification)) AND ("instruction tuning" OR "domain adaptation" OR "continued pretraining" OR LoRA OR "adapter tuning") AND (layout OR table OR diagram OR figure)`  

`(检索增强生成 OR RAG) AND (文档问答 OR 手册问答) AND (PDF OR 用户指南 OR 规格书) AND (指令微调 OR 领域自适应 OR 连续预训练 OR LoRA OR 适配器微调) AND (版面 OR 表格 OR 图示)`





*技术相关的文献, 重点关注pdf, 产品手册, 问答系统。尤其是涉及多模态、图像等*, 
- **优先保证解析质量**：表格尽量转 Markdown、图像抓图注+OCR；这些信息进入索引后，表格/图片问答才靠谱。
- 先跑**文本-only**流程；若图片信息很关键，再加 OCR 和（可选）图像检索。
- 双机跑更顺：Windows 负责解析/嵌入/检索，M1 负责生成。

### LoRA 微调 + 小模型

不是所有 RAG 应用都必须微调，但针对这种***专业性较强***的电气领域文档，做一个 LoRA 微调会是一个不错的选择

在 RAG 系统里，预训练大模型的基础性能确实很重要，但在这种专业性很强的领域里，微调后的“小而精”模型有时确实能胜过没微调过的大模型。
很多时候，一个经过 LoRA 微调的小型模型在特定领域的 RAG 应用中确实能比一个没微调的大模型表现更好。因为 RAG 的核心是检索和生成的结合，在你的场景里，生成部分能更好地理解专业语境和专有词汇，会让整个系统的回答更精准。


时间范围: 2021年以后

# 四、筛选与质量评估

1. **初筛（Title/Abstract）**
    
    - 与 RAG 或 Agent 核心概念高度相关
    - 涉及企业场景或架构分析
        
2. **复筛（Full Text）**
    
    - 方法部分详述 RAG/Agent 组件或协同实现
    - 应用/案例明确指出企业流程或评估指标
        
3. **质量评估标准**
    
    - **影响力**：被引频次、出版渠道（顶会/期刊）
    - **新颖度**：发表时间、是否首次提出关键架构
    - **可信度**：实验/案例规模、评估方法的严谨性

# 五、数据提取与归纳

为每篇入选文献填写提取表，主要字段包括：

1. **文献基本信息**：作者、年份、来源
2. **研究主题**：RAG 架构、Agent 平台、协同方式、应用场景
3. **技术细节**：检索器模型、生成器模型、系统流程图
4. **应用价值**：目标业务、达成效果、评估指标／结果
5. **挑战与未来方向**：性能瓶颈、数据依赖、扩展性问题



# 六、结果汇总与可视化

- ***趋势图***：年度论文数量、主要会议分布
- **主题聚类**：RAG-only、Agent-only、RAG-Agent 三大群组
- **架构对比表**：主流开源/商用框架特性一览
- **应用场景矩阵**：场景 vs. 技术能力 vs. 业务收益







