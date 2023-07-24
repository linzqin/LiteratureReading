# Revisiting binary code similarity analysis using interpretable feature engineering and lessons learned
使用可解释的特征工程和经验教训重新审视二进制代码相似性分析

> Kim D, Kim E, Cha S K, et al. Revisiting binary code similarity analysis using interpretable feature engineering and lessons learned[J]. IEEE Transactions on Software Engineering, 2022, 49(4): 1661-1682.


* 检索收录情况：SCI
* 中科院 JCR 分区:Q1
* 当前被引用数:33

## Summary

写完笔记之后最后填，概述文章的内容，以后查阅笔记的时候先看这一段。

> 注：写文章summary切记需要通过自己的思考，用自己的语言描述。忌讳直接Ctrl + C原文。

## Research Objective(s)

1. 从新审视BSCD这一块的研究难点，并进行总结合回顾BSCD这一块的采用的技术，使用精确定义术语并对之前文献中使用的特征进行分类
2. 为BSCD构建一个全面可重复的基准
3. 设计一个可解释的特征工程模型

## Background / Problem Statement

- BSCD的研究难点
    1. 大部分人采用的是带有不可解释性的机器学习技术
    2. 每一篇论文都有自己的基准去进行评估，所以方法之间无法进行相互比较
    3. 研究人员的术语不统一，导致无法正确引用以前的文献

## Method(s)
### BINARY CODE SIMILARITY ANALYSIS（BSCA,二进制代码相似性分析）
1. 句法分析
    - 获取二进制代码的中间表示（IR）：反汇编/抽象语法树
2. 结构分析
    - 例如恢复CFG、CG等
3. 语义分析
    - 可以通过对二进制文件进行数据流分析或者符号分析等，找出底层语义
4. 向量化和比较
    - 产生一个0-1之间的相似分数

## Evaluation

作者如何评估自己的方法？实验的setup是什么样的？感兴趣实验数据和结果有哪些？有没有问题或者可以借鉴的地方？

## Conclusion

作者给出了哪些结论？哪些是 `strong conclusions`, 哪些又是 `weak conclusions`（即作者并没有通过实验提供 `evidence`，只在 `discussion` 中提到；或实验的数据并没有给出充分的 `evidence`）?

## Notes(optional) 

不在以上列表中，但需要特别记录的笔记。例如英文书写模板，精美绘图所使用到的工具软件等。

## References(optional) 

列出相关性高的文献，以便之后可以继续 track 下去。

## Origin

给出指向你个人论文仓库的本篇论文阅读笔记原文链接。

## Tags

逗号分隔本文的所有标签，标签使用规范参见以下 `GitLab Issue 标签使用规范` 。

2021, SCI, SCI-1, CyberRange, Playbook, Ansible, Scenario, CTF

------ 以下内容仅为解释说明，请在提交时删除 ------

### GitLab Issue 标签使用规范

* 在不影响语义理解的前提下，标签关键词要尽可能短
* 优先选择已有标签，确实没有的情况下再 `新建标签`

#### 建议的标签列表

* 文献发表年份。例如 2021
* 检索收录情况：EI, SCI 
* 中科院 JCR 分区（针对 SCI 收录文献才需要标记）：SCI-1, SCI-2, SCI-3, SCI-4
* 关联实验室内项目简称（最近更新 2021-05-22）：Fuzz, CyberRange, osint4sn, SoftFP, MTD
    * 物联网漏洞挖掘：Fuzz
    * 靶场：CyberRange
    * 开源社区情报分析：osint4sn
    * SoftFP：软件指纹
    * 欺骗式防御：MTD
* 研究对象（不超过3个）。例如：IoT, 源代码 等
* 研究方法（不超过3个）。例如：综述, 动态分析, 静态分析, CNN 等
* 数据集（不超过3个）。例如： NB-15，MalwareZoo, VulDeePecker, LAVA-M 等

