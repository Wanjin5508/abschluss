# day 1
## 概述
![[Pasted image 20250625224459.png]]

pytorch, tf 属于开发框架

langchain 兼容宝宝脸上的模型

agent借鉴了RL中的概念

langchain开发类似乐高积木, 使用多个组件构造不同的应用

![[Pasted image 20250625224900.png]]
LC的官方包在core, 但是第三方包更强大

graph强调图的概念, 尤其是多角色

![[Pasted image 20250625225044.png]]


![[Pasted image 20250625225110.png]]


![[Pasted image 20250625225202.png]]

![[Pasted image 20250625225240.png]]
--> 典型应用, LLM用于检索内部和外部资源库

最基本的用法: 
![[Pasted image 20250625225527.png]]
 
![[Pasted image 20250625225702.png]]

![[Pasted image 20250625225825.png]]
LC 目前属于前端开发, 代码的复用率很高。

AI 在公司中肯定要和业务结合, 所以有利于深入了解实际业务
`pip install langchain=0.3.0, pip install langchain-openai=0.2.9`
目前工业界最广泛应用的版本。

langchain调用的大模型可以使用ollama部署的模型。


## LangChain 初探
### 计算tokens
![[Pasted image 20250625231147.png]]
--> 我在hiwi工作中用过这个分词器, 在我微调gpt2模型的时候
最大的好处就是速度快

![[Pasted image 20250625231310.png]]

langchain 虽然可以兼容ollama的模型, 但是效果最好的还是原生 Open Ai 的api



# day 2 LangChain API &  Function Call























 