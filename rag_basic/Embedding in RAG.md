在RAG中, 
- 我们会把每一段知识文档（文本段落、表格、图片）转换成**向量（embedding）**
    
- 用户提问也会被转换成**查询向量**
    
- 然后通过**向量相似度**（通常是余弦相似度）来找出与问题最相关的段落/图像/表格
    
- 这些相关内容组成“上下文”传给大模型进行生成（Answer）

过程简图如下: 
```bash
用户问题 → 向量 q
         ↓
知识库向量 [d1, d2, d3, ..., dn]
         ↓
q 与 di 计算相似度 → top-k 相关 chunk
         ↓
送给大模型生成回答
```


## 文本、表格、图像的向量表示方式

在项目中采用 **双通道嵌入策略**：

### 1. 文本嵌入（text embedding）

- 包括手册中的正文说明、操作说明、故障代码描述等
    
- 每一页（或每一段）作为一个文本 chunk
    
- 模型：`sentence-transformers/all-MiniLM-L6-v2`
    
- 向量维度：384
    
- 优点：小巧、高效、语义表达能力好
    
- 示例：

```python
from sentence_transformers import SentenceTransformer

model = SentenceTransformer("all-MiniLM-L6-v2")
embedding = model.encode("How to reset the device?")
```

### 2. 表格嵌入（表格 → markdown → 文本嵌入）

- 表格没有结构可直接嵌入，但我们将其转换为 **Markdown 表格**
    
- 然后当作普通文本送入与文本相同的嵌入模型
    
- 示例 Markdown 表格：

```bash
| 功能         | 参数值 |
|--------------|--------|
| 输入电压     | 24V    |
| 最大电流     | 2A     |
```

这实际上是让模型“读懂表格”，作为一句话的变体。

### 3. 图像嵌入（image embedding）

- 包括示意图、设备后面板布线图、按钮图标等
    
- 使用的是 **CLIP 模型（ViT + Text）**，可以支持：
    
    - 图像 → 向量（索引图像）
        
    - 文本查询 → 向量（检索图像）
        
- 模型：`sentence-transformers/clip-ViT-B-32`
    
- 向量维度：512
    
- 示例：

```python
from sentence_transformers import SentenceTransformer
from PIL import Image

clip_model = SentenceTransformer("clip-ViT-B-32")
img = Image.open("rear_panel.png")
embedding = clip_model.encode([img])
```

⚠️ CLIP 不需要分类标签，而是学到图文对齐的语义空间，因此你可以用 “rear panel layout” 来成功检索出一张图。

| 类型  | 内容来源        | 嵌入方式                      | 模型                 | 输出维度 |
| --- | ----------- | ------------------------- | ------------------ | ---- |
| 文本  | 正文段落、标题等    | 直接 encode 句子文本            | `all-MiniLM-L6-v2` | 384  |
| 表格  | markdown 表格 | 转换为 markdown 文本后一起 encode | `all-MiniLM-L6-v2` | 384  |
| 图像  | 提取 PNG 图像   | PIL.Image → CLIP encode   | `clip-ViT-B-32`    | 512  |



## 补充：为什么不对 OCR 文本单独嵌入？

如果启用了 OCR，那么提取的图片文字已经被加入到图片的 chunk 中（如 `chunk["content"]` 字段），因此可以选择：

- 一并作为图像 chunk 的文本处理（嵌入时附加）
- 或者仍由图像向量主导，OCR 仅作为补充信息显示（更推荐）



# 嵌入模型的选择
## ✅ 文本 & 表格嵌入模型：`all-MiniLM-L6-v2`

### 📌 选择理由

| 维度        | 理由说明                                                  |
| --------- | ----------------------------------------------------- |
| ✅ 性能-速度平衡 | MiniLM 是 transformer 蒸馏模型，参数少、推理快，适合 CPU / 本地运行       |
| ✅ 语义能力    | 在句子层面语义匹配任务上，效果优于 GloVe、BERT-base 等经典模型               |
| ✅ 社区验证    | HuggingFace + SentenceTransformers 社区大量检索系统都采用该模型作为默认 |
| ✅ 精度充足    | 在大部分 RAG QA / FAQ / 文档问答场景中，效果已非常可用（尤其是 top-k recall） |
| ✅ 通用性强    | 对短文本、中英文混排、技术术语都能保持稳健嵌入效果                             |
| ✅ 表格支持    | 表格转 markdown 后作为自然语言处理，MiniLM 处理 markdown 也没有问题       |

### 📌 具体效果支撑：

- 在 `MS MARCO Passage Retrieval`, `STSb`, `Quora Question Pairs` 等语义检索评测任务中，MiniLM 系列一直是中轻量级模型中的表现最佳者之一。
    
- 来自 SentenceTransformers 官方文档推荐，用于 production 向量索引场景。
    

### 📌 可选替代方案（适合追求更高语义质量）：
|模型|优势|代价|
|---|---|---|
|`multi-qa-MiniLM-L6-cos-v1`|微调于 QA 任务，更适合问答|稍大|
|`BAAI/bge-small-en`|中文英文都强，BGE 专为 RAG 优化|需要本地量化|
|`intfloat/e5-small-v2`|现代语义搜索 SOTA，任务泛化强|推理略慢|

## ✅ 图像嵌入模型：`clip-ViT-B-32`（via SentenceTransformer）

### 📌 选择理由
| 维度       | 理由说明                                        |
| -------- | ------------------------------------------- |
| ✅ 图文对齐   | CLIP 是专为图文匹配训练的模型：图像和文本嵌入在同一空间，可以交叉检索       |
| ✅ 文本检索图像 | 用户输入文本 `"rear panel layout"`，能直接检索出相应图片     |
| ✅ 部署简单   | SentenceTransformers 提供接口，输入 `PIL.Image` 即可 |
| ✅ 快速且准确  | ViT-B-32 是基础版本，平衡速度与精度，适合中型项目               |
| ✅ 文档适配强  | CLIP 对“技术图、设备草图、线缆标识”等低语义图像也能保持较好区分度        |
| ✅ 多语言支持  | 文本检索可以使用英文、德文、中文，均支持（训练语料较丰富）               |

### 📌 可选替代方案（如果希望强化图像语义理解）：
|模型|优势|代价|
|---|---|---|
|`openai/clip-vit-large-patch14`|更强特征能力|慢、显存占用高|
|`Salesforce/BLIP` + `BLIP-2`|图像问答能力强|不适合直接用于检索|
|`GritLM`|新一代多模态嵌入，学术领先|工程支持较弱，未集成入 SentenceTransformers|

## ✅ 为什么不选择大型嵌入模型（如 OpenAI Embedding 或 Cohere）？

| 原因        | 解释                                               |
| --------- | ------------------------------------------------ |
| 🧱 本地部署目标 | 你当前是基于 Ollama 和本地化部署，不依赖云服务                      |
| 💸 成本控制   | OpenAI embedding 模型要按 token 计费，价格随文档增大很快上升       |
| 🔁 可重复性   | 本地模型在不同机器上可离线运行，保证可重复实验                          |
| 🔌 兼容性强   | 使用 huggingface + sentence-transformers 框架，生态集成广泛 |








