---
string: '"business process automation" AND ("AI agent" OR "intelligent agent")'
time: 15.06.2025
since: "2022"
number: "79"
---
# 筛选原则: 
优先选取基于agent的知名期刊文章, 尤其是这个领域最新的研究成果。可以引用其中的理论框架来填充背景部分。甚至有些框架可以应用到智能体app的开发。
传统深度学习模型则不被纳入考虑范围, 因为我们只考虑应用层, 尤其是LLM的应用

# 快速上手 - [[德勤multi-agent报告.pdf]]
![[Pasted image 20250624113358.png]]

本文不是严肃的学术文章, 但是其中的各种流程图值得借鉴! 上图这种分步骤介绍agent系统的工作流程一定要写在论文里! 下图则是论文重点对比的两个对象: 
![[Pasted image 20250624113528.png]]




# Theories
##  1. [[Enhancing Trust in LLM-Based AI Automation Agents.pdf]]

我的想法: 强化人类用户和智能体之间的信任, 对于智能体系统是否真正能融入到价值创造过程很重要, 尤其是在智能体系统的核心功能之外的例如用户体验等方面。

文章列出了几个重要的评估准则, 并实际评估了

基于LLM的自动化智能体在商业使用环境中, 如何增加用户信任和信心, 文中提出了以下要点, 可以用作app构建的基础或者评估原则:
1. reliability
	- ![[Pasted image 20250624111903.png]]
2. Openness
- ![[Pasted image 20250624112220.png]]
3. Tangibility
	- ![[Pasted image 20250624112419.png]]
	- XAI???

3. Immediacy behaviors
	- ![[Pasted image 20250624112549.png]]

## 2. [[Empowering_Generative_AI_with_Knowledge_Base_4.0_Towards_Linking_Analytical_Cognitive_and_Generative_Intelligence.pdf]]

agent通过外接知识库的方式, 实现了knowledge, experience, creativity的组合: 
This accumulation of knowledge, experience, and creativity can enhance an individual’s ability to navigate complex systems and optimize outcomes

然后将系统中的智能体分类为3种: 
there are three primary types of AI systems based on these components: Analytical AI, Cognitive AI, and Generative AI, which correspond to the Intelligence components: Knowledge, Experience, and Creativity, respectively.
这三类ai分别扮演不同的角色, 执行不同的功能, 并且所需的实现技术也不同。

![[Pasted image 20250624121852.png]]

# Case studies
## 1. [[Generative_AI_Enabled_Robotic_Process_Automation_A_Practical_Case_Study.pdf]]
用C#开发了基于GAI的elearning平台评分系统的agent, 可以借鉴:
![[Pasted image 20250624105626.png]]
让不同agent扮演不同角色, 并赋予不同的功能, 类似于公司内部的多人协同工作! 
![[Pasted image 20250624105842.png]]


## 2. [[Rebalancing Worker Initiative and AI Initiative in Future Work.pdf]]

给出了4个评估任务的维度, 
Tasks may be analyzed along dimensions of retrospective vs. prospective [50], informative vs. actionable [101], reminding vs. being-reminded [75], visible vs. invisible [23, 94, 104], contentoriented vs. relationship-oriented [22, 75, 114], and holistic [6] vs. itemized [10, 25, 34, 51].

并采用了多个 case studies , 方法和结论可以借鉴

以下表格也能用, 在我描述任务的时候, 也要考虑这些维度:
![[Pasted image 20250624114812.png]]


分步骤定义任务:
![[Pasted image 20250624114911.png]]


lofi描述UI:
![[Pasted image 20250624114958.png]]



