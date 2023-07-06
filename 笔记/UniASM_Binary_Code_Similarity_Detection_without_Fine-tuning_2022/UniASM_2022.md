# UniASM: Binary Code Similarity Detection without Fine-tuning

> Gu Y, Shu H, Hu F. UniASM: Binary Code Similarity Detection without Fine-tuning[J]. arXiv preprint arXiv:2211.01144, 2022.

* 当前被引用数:13

## Summary

写完笔记之后最后填，概述文章的内容，以后查阅笔记的时候先看这一段。

> 注：写文章summary切记需要通过自己的思考，用自己的语言描述。忌讳直接Ctrl + C原文。

**开源：https://github.com/clm07/UniASM**
## Research Objective(s)

- BSCD
    - 如何选择backbone
    - 如何选择training task
    - 如何正确序列化汇编代码

## Background / Problem Statement

### 研究现状

#### 传统方法

- 动态方法
    1. 基于符号执行和深度五点分析：BinHunt [20] 和 iBinHunt [21]
        - 符号执行成本高，难以大规模运用
    2. 通过执行目标程序获取函数的I/O值：Blex[22]、BinGo[23]、BinGo-E[24]和Multi-MH[25]
        - I/O值不能完全代表函数语义
    3. 模拟执行：CACompare [6] 和 BinMatch [26]
    4. 运行时特征：IMF-sim [27] 和 BinSim
- 静态方法
    1. 统计特征：BinClone [31]、ILINE [32]、MutantX-S [33]、BinSign [34]和BinShape [35]
    2. 通过指令序列之间的编辑距离：Tracelet [36]和BinSequence [37]
    3. 通过树和图的编辑距离比较函数的CFG：TEDEM[38]、XMATCH[39]和Sæbjørns等人[9]
    4. 利用图同构：DiscovRe [40]、BinDiff [41]、Genius [42] 和 Kam1n0 [7]
#### 基于学习的方法

- 基于DNN（会丢失顺序信息）
    1. 二进制文件转化成图像，然后使用CNN：Marastoni等人[43]
    2. 从原始字节中使用CNN：αdiff [1] 
    3. 提取基本块的特征输入DNN：VulSeeker [44]

- 基于图（计算复杂度高）
    1. 基于ACFG：Gemini [11] 和 GraphEmb [45] 、BugGraph [10]（graph triplet-loss）、HBinSim[48]（分层注意力图嵌入） 
    2. 图匹配网络：GMNN [46]
    3. 图卷积：Bin2vec [47]
    4. 在语法树上使用Tree-LSTM: Asteria [49]
- 基于NLP
    1. word2vec: Asm2vec [12]、InnerEye [2]（还使用了LSTM）
    2. bert：PalmTree [17]、DeepSemantic [52] 和 BinShot [53] 
    3. transformer：MIRROR [54]、jTrans [19]

- 混合方法：
    1. word2vec+自注意力神经网络：SAFE [14]
    2. word2vec+DeepWalk：DeepBinDiff [55]
    3. CNN、LSTM 和常规全连接前馈神经网络:BinDNN [57] 
    4. NLP+图自动编码器模型： BEDetector [58]
    5. NLP+图表示学习模型： Codee [59]
    6. word2vec、BERT、MPNN [61] 和 CNN：OrderMatters [60]

## Method(s)
> 受到SimBERT [62] 和 UniLM [63]得启发

### 整体框架

![](./img/uniasm_overview.png)        

### function representation

#### 1. 指令规范化

- 替换地址、浮点指令、条件跳转指令
1. 间接寻址寄存器 eip/rip 替换为 PTR
2. 间接寻址寄存器 esp/rsp 替换为 SSP
3. 间接寻址寄存器 ebp/rbp 间接寻址替换为 SBP
4. 其他间接寻址替换为 MEM
5. 相关寻址替换为 REL
6. 立即数替换为 NUM
7. 浮点指令寄存器 xmm 的替换为 XMM
8. 条件跳转（例如 jnz）替换为 cjmp

#### 2. Assembly Tokenization
将整个指令视为一个token：mov rax, 0x10 -> mov_rax_NUM
> 将整个指令视为一个token，首先需要考虑的问题就是OOV问题，这也是上一步作者进行规范化的原因。但是作者在此也提出了通过消融实验，这种整个指令当作token的方法要优于更细粒度的方法

#### 3. 函数序列化

- 选择的是按照线性顺序（地址顺序）进行序列化
> 后续笑容实验表明，线性顺序的性能和随机游走、最长游走类似，但那时随机游走和嘴仗游走依赖于CFG

### Backbone network
> backbone为transformer             

![](./img/Backbone_network.png)         

#### 1. token embedding

- 用来生成函数token序列的输入向量
- 函数$F$表示为：$F=[x_1,x_2,...,x_n]$，$x_i$表示函数F的第i个token
- 输入向量$H^0$：

![](./img/inputvector.png)         

- $E(x_i)$:   
    ![](./img/exi.png)       
    - $E_{x_i}$：token embedding
    - $E_{p_i}$：position embedding
    - $E_{s_i}$：segment embedding

#### 2. Self-attention Layer

- 由多个transformer层相互堆叠组成
- 第一层的输入就是input vector：$H^0$

- Self-attention层公式     

    - ![](./img/Self-attention公式.png)    
    - $L$：transformer总共层数
    - $H^l$：第l层的输出，$H^l=transformer_l(H^{l-1})$

#### 3. Function Embedding Layer

> 通过token`CLS`的输出向量计算函数嵌入向量

![](./img/Function_Embedding.png)      

### training task

> ALG和SFP

#### 1.Assembly Language Generation      

![](./img/ALG.png)       

- 利用注意力掩码矩阵来定义双向注意力和单向注意力，输入序列中的第一个函数使用双向注意力，而第二个函数使用单向注意力。
- 输入是两个函数，第一个函数$F=[x_1,x_2,...,x_n]$和第二个函数$F'=[y_1,y_2,...,y_m]$，ALG的目标就是根据第一函数预测第二个函数
- 损失函数：交叉熵，p未softmax            

![](./img/ALG_loss.png)          

![](./img/softmax.png)          

#### 2. Similar Function Prediction 

> 每一次处理一批函数

![](./img/SFP.png)         

- 每一批次中的每个样本都是一对相似函数`[CLS] F [SEP] F' [SEP]`,`F`和`F'`是相似函数对1，交换两个函数后，得到一个新的样本：`[CLS] F' [SEP] F [SEP]`
- 同一批次中第k个函数的embedding为$v_k=[v_1,v_2,...,v_d]$，d是hidden size，然后对向量中每个元素进行归一化        
    ![](./img/归一化.png)      
    
    - 最终得到归一化函数embedding   

    ![](./img/归一化函数embedding.png)

    - 合并起来得到embedding矩阵,b为batch size                 

    ![](./img/合并矩阵.png)            

- 两个函数之间的相似度S：embedding矩阵与其转置矩阵的点积        

    ![](./img/相似度.png)        

    - S中每个值标书两个函数的相似度
        1. 单位向量的点积值等于cos(a)，a为两个向量的夹角，向量越相似，角度越小，cos越接近于一，不相同函数的向量，点积接近于-1
    - 由于矩阵S对角线上的值都等于1，为了避免对角线元素的影响，将所有对角线元素设置为`-∞`   

- 每个矩阵经过softmax层处理    

    ![](./img/softmax2.png)          

    - $s_{ij}$：第i个函数和第j个函数的相似度

- SFP的损失函数：交叉熵，p未softmax

    ![](./img/SFP_loss.png)       
       

- 整个框架的损失函数为两者之和        

![](./img/loss.png)
    

## Evaluation

### Dataset

- 训练集
    - 开源项目：7个
    - 优化选项：O0-O3+3个混淆（sub/fla/bcf），**使用“-fno-inline”编译的，以避免函数内联**
    - 编译器：GCC-7.5和clag-10
    - 二进制文件数：133
    - 反汇编工具：Radare2
    - 不重复函数：12,694
    - 相似函数对：500k
- 评估数据集
    - Dataset-1
        - 由五个开源项目生成，四个优化级别 (O0/O1/O2/O3) 的两个编译器（GCC 和 Clang）以及具有三个混淆（sub/fla/bcf）的 Ollvm14 编译
    - Dataset-2：消融研究。从Dataset1的每个项目中随机选择 200 个函数，得到 1000 个函数。
    - Dataset-3：用于评估现实世界漏洞搜索的性能

### 评估指标 

- 原函数池$F$和ground truth池$G$

    ![](./img/ground poll.png)      

    - 每个函数$fi ∈ F$在ground truth中都有对应的值     

- 评估指标：Recall@k

    ![](./img/Recall.png)      

### 超参数
1. 4 transformer layers
2. 12 attention heads
3. max sequence length of 256
4.  vocabulary size of 21000
5. intermediate size of 3072
6. train batch size of 8
7. learning rate is set to 5e-5 with the warmup of 4 steps

### 评估结果 

## Conclusion

作者给出了哪些结论？哪些是 `strong conclusions`, 哪些又是 `weak conclusions`（即作者并没有通过实验提供 `evidence`，只在 `discussion` 中提到；或实验的数据并没有给出充分的 `evidence`）?

## Notes(optional) 

不在以上列表中，但需要特别记录的笔记。例如英文书写模板，精美绘图所使用到的工具软件等。

## References(optional) 

- 通过BSCD进行补丁分析
> [9]

- Jtrans
> [19]

- 库函数的识别：IDA FLIRT [29] 和 UNSTRIP [30]


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

