see also `abschluss_code/src/README.md`

```bash
rag_local/
│
├── rag/               # 核心模块代码包
│   ├── __init__.py
│   ├── config.py                 # 模型与系统配置
│   ├── ingest.py                 # 解析 PDF 提取文本、表格、图片
│   ├── embed.py                  # 向量化文本与图像（CLIP）
│   ├── index.py                  # 构建 FAISS 向量索引 + BM25
│   ├── retrieve.py               # 多通道检索逻辑 + RRF 融合
│   ├── generate.py               # 基于上下文通过 Ollama LLM 生成回答
│   └── cli.py                    # 命令行入口
│
├── workdir/                      # 存储处理结果（用户自定义）
│   ├── corpus.jsonl              # 所有 chunk 的元数据与内容
│   ├── images/                   # 提取出的图片
│   ├── text_embs.npy             # 文本向量
│   ├── image_embs.npy            # 图像向量
│   └── faiss_text.index          # 向量索引文件（文本）
│   └── faiss_image.index         # 向量索引文件（图像）
│
├── requirements.txt              # 所需依赖
└── README.md                     # 使用说明
```

| 模块            | 功能说明                                                    |
| ------------- | ------------------------------------------------------- |
| `ingest.py`   | 利用 `pdfplumber` 和 `PyMuPDF` 提取 PDF 中的文本、表格（markdown）与图片 |
| `embed.py`    | 使用 SentenceTransformer（MiniLM + CLIP）生成嵌入向量             |
| `index.py`    | 用 FAISS 构建向量索引，同时用 BM25 支持词面召回                          |
| `retrieve.py` | 实现多通道检索（文本 + 图像 + BM25）与融合排序                            |
| `generate.py` | 组织 prompt，将检索片段传给本地 Ollama 模型生成答案                       |
| `cli.py`      | 提供四个命令：`ingest`, `embed`, `index`, `ask`                |


# TODO list

## `ingest.py`

### extract texts and tables
![[Pasted image 20250824215743.png]]
公式应该如何解析?

页眉和页脚的文本也会被提取, 而这一部分的作用不大

### extract images
图片存在漏检的情况, 例如下面的图片是无法被解析的:
![[Pasted image 20250824214923.png]]


但是可以通过`extract_text_and_tables`函数将每一页存在的图片和表格的标题以text格式提取出来并保存到 `corpus.jsonl` 文件中。因此或许可以手动补全未被解析出来的图像, 或者使用正则表达式匹配 “Fig, ”

在手动补全缺失的图像时, 需要按顺序编号:
![[Pasted image 20250824230236.png]]
--> 此外还需要考虑是否需要手动添加图像说明, 并将图像说明添加到 content 字段, 从而替代OCR (这部分具体操作的依据要从论文中找)



## `embed.py`

see [[Embedding in RAG]] for details, 尤其是选择每一个embedding模型的理由, 需要按需检索相关文献

### 步骤 1：加载 PDF 内容切片（chunks）

我们首先加载在 `ingest.py` 中保存的 `corpus.jsonl` 文件。这是一个文本文件，每一行是一个 JSON 格式的 chunk，包括：

- 文本段落（type = "text"）
    
- markdown 格式表格（type = "table"）
    
- 图片（type = "image"，带有本地图片路径）
    

---

### 步骤 2：根据类型分类准备嵌入输入

我们将所有 chunk 分类：

- 文本和表格：从 `content` 字段读取字符串，组成一个列表
    
- 图片：从 `meta.image_path` 字段获取路径，检查文件是否存在，作为 `PIL.Image` 对象加载
    

---

### 步骤 3：加载两个模型分别嵌入

- 文本 + 表格一起送入 `SentenceTransformer("all-MiniLM-L6-v2")`，输出归一化后的 384 维向量
    
- 图像列表送入 `SentenceTransformer("clip-ViT-B-32")`，输出归一化后的 512 维图像嵌入向量
    

向量都使用 `convert_to_numpy=True` 返回为 NumPy 数组，并使用 `normalize_embeddings=True` 归一化，方便后续做余弦相似度检索。

---

### 步骤 4：保存结果文件

在 `workdir/` 文件夹下保存：

|文件名|内容|
|---|---|
|`text_embs.npy`|文本和表格的向量矩阵|
|`image_embs.npy`|图像的向量矩阵|
|`text_ids.json`|每个文本向量对应的 chunk ID|
|`image_ids.json`|每个图像向量对应的 chunk ID|
|`image_paths.json`|图像文件路径（用于渲染）|

这些文件将在后续构建向量检索索引时使用。

## `index.py`
see [[embedding & tokenization]] for more details

### FAISS 向量索引

- 文本和图像向量都使用 `IndexFlatIP` 类型构建索引，代表使用**内积相似度**
    
- 因为我们在 `embed.py` 中已做向量归一化，内积即为**余弦相似度**
    
- 索引支持快速 top-K 相似搜索
    

### BM25 构建

- 使用 `rank_bm25` 库构建语料库
    
- 每个 chunk 的内容都被分词，组成 token 列表
    
- 创建 `BM25Okapi` 索引模型（不需要训练）
    
- 保存分词结果与 ID，便于后续恢复


## `retrieve.py`

### 模块目标：多通道检索融合

1. **载入三类索引：**
    
    - 文本向量（from `faiss_text.index`）
    - 图像向量（from `faiss_image.index`）
    - BM25 索引（from 分词）
        
2. **用户输入 query 问题**
    
    - 用两种模型分别将 query 编码成向量（MiniLM、CLIP）
    - 同时分词（BM25）
        
3. **各通道进行检索：**
    
    - 向量检索（FAISS）
    - 关键词检索（BM25）
        
4. **结果融合（Reciprocal Rank Fusion）**
    
    - 将不同通道结果融合成一个统一排序的列表
        
5. **构建上下文供 LLM 使用（返回 top-k chunk）**

### 模块输出
--> 至此, 完成了PDF文件的上下文检索功能, 下一步实现拼接prompt并通过本地ollama调用gemma模型生成答案, 本模块运行结果如下:

```txt
❯ python retrieve.py
Using a slow image processor as `use_fast` is unset and a slow processor was saved with this model. `use_fast=True` will be the default behavior in v4.52, even if the model was saved with a slow processor. This will result in minor differences in outputs. You'll still be able to use a slow processor with `use_fast=False`.

[OK] Retrieved top 12 chunks:

--- Chunk 1 ---
Type: text, Page: 32
User's Manual
Control and Feeding Unit
Type RSE 400 / RSE 800
11.3.1. Screen "Test limits"
The screen [Test limits] can be used to enter limit values for a test; it can be called from the
main menu using the button [Test limits].
Fig. 9: Screen [Test limits]
[Maximum test voltage] When the test volt ...

--- Chunk 2 ---
Type: text, Page: 59
User's Manual
Control and Feeding Unit
Type RSE 400 / RSE 800
14. Test sequence
A test sequence can be controlled both fully automatically and manually. The operator may
additionally enable or disable automatic frequency search.
14.1. Fully automatic testing
Step 1 – Input of all test parameters
Pre ...

--- Chunk 3 ---
Type: text, Page: 66
User's Manual
Control and Feeding Unit
Type RSE 400 / RSE 800
18. Status and error messages
During a test or its preparation, events or errors can occur that lead either to cancellation of
the test or prevent its start. The error cause is displayed in the status bar of the screen [Test].
Errors occu ...

--- Chunk 4 ---
Type: table, Page: 67
| [Error: U ic difference too high] | The intermediate circuit voltage of a converter deviates
from the mean value of all converters by more than 150 V. |
| --- | --- |
| [Error: PT100 sensor for ambient
temperature defective] | The ambient temperature cannot be determined. |
| [Error: Buchholz rela ...

--- Chunk 5 ---
Type: text, Page: 67
User's Manual
Control and Feeding Unit
Type RSE 400 / RSE 800
The intermediate circuit voltage of a converter deviates
[Error: U ic difference too high]
from the mean value of all converters by more than 150 V.
[Error: PT100 sensor for ambient
The ambient temperature cannot be determined.
temperatur ...

--- Chunk 6 ---
Type: text, Page: 43
User's Manual
Control and Feeding Unit
Type RSE 400 / RSE 800
Screen [Summary of unit errors]
This screen provides a summarized overview of error information for the RSE 400 (800) in
respect of interconnected inverter units. It can be called from the screen [Errors] using the
button [Overview units] ...

--- Chunk 7 ---
Type: text, Page: 42
User's Manual
Control and Feeding Unit
Type RSE 400 / RSE 800
11.3.6.5. Screen "Errors"
The screen [Errors] provides a detailed overview of error information for the RSE 400 (800). It
can be called from the menu [Diagnosis & errors] using the button [Errors].
Fig. 23: Screen [Errors]
[PC communicati ...

--- Chunk 8 ---
Type: image, Page: 27
 ...

--- Chunk 9 ---
Type: image, Page: 27
 ...

--- Chunk 10 ---
Type: image, Page: 37
 ...

--- Chunk 11 ---
Type: text, Page: 71
User's Manual
Control and Feeding Unit
Type RSE 400 / RSE 800
1. Enter the resonance frequency to be expected as the start frequency.
2. Set the limit value for the test voltage to max. 50% of the future test voltage.
3. Set the setpoint test voltage to 10% of the future test voltage.
4. Set the fre ...

--- Chunk 12 ---
Type: image, Page: 27
 ...


[Assembled Context for LLM]:

[text] (Page 32)
User's Manual
Control and Feeding Unit
Type RSE 400 / RSE 800
11.3.1. Screen "Test limits"
The screen [Test limits] can be used to enter limit values for a test; it can be called from the
main menu using the button [Test limits].
Fig. 9: Screen [Test limits]
[Maximum test voltage] When the test voltage exceeds this value, the test is cancelled
automatically. The limit value may lie up to 5% above the system
nominal voltage to avoid shutting down by slight overshot during
a test with system nominal voltage.
[Maximum inverter If the output current of an inverter unit exceeds this value, the test
current per unit] is automatically cancelled immediately with the message [Error:
Overcurrent].
[Minimum test frequency] Limit value for the lowest test frequency
[Maximum test frequency] Limit value for the highest test frequency
[Maximum reactor If the oil or winding temperature of a resonant reactor exceeds
temperature] this value, the test is automatically cancelled immediate ...
```

> 必须进行文本预处理, 把每个chunk开头的文本通过 re 模块去掉, 同时还要考虑是否有必要去除换行符

## generate.py

### 模块目标：生成最终回答

1. 接收一个用户问题（query）
    
2. 调用 `retrieve.py` 中的检索函数，得到 top-k 最相关 chunk
    
3. 拼接这些 chunk 成为上下文（`context`），作为 prompt
    
4. 调用本地运行的 **Ollama 模型（gemma3:4b）**
    
5. 返回回答文本和引用内容


测试:
```python
if __name__ == "__main__":
    test_question = "What is the reason of Error: Overvoltage?" # table, page 66
    workdir = "workdir"
    generate_answer(test_question, workdir)
```

不调用大模型, 而是仅仅采用检索:
```
❯ python generate.py
[Query] What is the reason of Error: Overvoltage?
Using a slow image processor as `use_fast` is unset and a slow processor was saved with this model. `use_fast=True` will be the default behavior in v4.52, even if the model was saved with a slow processor. This will result in minor differences in outputs. You'll still be able to use a slow processor with `use_fast=False`.

[✓] 生成回答：

[Error: Overvoltage] The test voltage has exceeded its limit value.
```

证明索引是可以工作的




