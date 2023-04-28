**对Blockchain Large Language Models的摘要_98112a83408d75f830703b43255f27b9c9ffa038**

1⃣️

1、2.

20180430-20220431 32.4亿美元 lost

IDS：Real-time Intrusion Detection System (IDS)

SOTA ：State of the Art: reward based / pattern based

基于奖励：专注于识别和利用产生重大利润的交易；基于模式：图7

以上：仅仅对预定义规则、模式或有利可图的漏洞的依赖

3.假设训练一个模型来学习交易的执行痕迹，而无需事先知道漏洞（重入、价格操纵），假阳性可能会比较多；

4.研究对象是四年内的攻击，数据集一共68M个交易，指定时间段内124个合约产生的所有交易，这124个合约都是遭受过攻击；txrank的性能给予这些交易进行测试验证；

5.两个指标：

a percentage ranking alarm threshold;

an absolute ranking alarm threshold;

在124次攻击中，排名49位的是与相应受害者合约交互的top3异常的交易；

6.10000次交易中 警报阈值设置为0.01% 的defi程序中（每10000次交易取前1个报警）：

txrank监测到24%的攻击；（这1个报警有24%的几率是真实攻击）

0.097%的假阳性率？见figure4

100笔交易/天 * 10天 *0.1%FPR（false positive rate） =  1

也就是说这十天只生成了一次报警。

7.实时性：2284±289交易/秒     0.16±0.3s/交易

提供智能合约暂停触发，作者调查的被攻击合约中有50%的已经部署了这样的暂停触发机制。

1. 对交易分析领域的贡献：
   1. 首次将无监督/自监督学习应用于智能合约交易中的异常检测，
      1. employing custom data encoding,
      2. domain-specifictokenization,
      3. and a tree encoding method tailored for EVMtrace tree representation,
      4. capturing calls, function names,parameters, and storage modifications
   2. 124 attacked contracts\ 68M transaction\1523 days \ block5470817-15000000
   3. 一项闪电贷攻击案例指示：不仅可以有效识别异常交易，还可以识别不同类型的恶意活动。

## 2⃣️Background

1. Blockchain and DeFi
   1. Blockchain
      1. 7-9全面参考
   2. SmartContract
      1. 交易费来防止拒绝服务攻击
      2. 区块链既不存储solidity 也不存储ABI，只存储编译后的字节码；
      3. 字节码的执行由交易触发，在EVM中执行；类似于Java语言，SmartContract的执行可以通过日志来跟踪状态转换。完整的区块链客户端存储整个历史块，由于存储需求过大，会丢弃间歇性状态和日志；存档节点的存在会提供只嗯那个合约执行的索引数据库。
   3. DeFi
      1. 250B USD
2. Natural Language Processing (NLP)
   1. NLP and Bytecode
      1. 一种方法是将代码或汇编视为自然语言文本，并将其输入到自然语言模型中。
      2. 代码结构化表示（AST abstract syntax tree）, AST as input of NLP model, 模型会更好的理解代码结构和含义；
      3. 字节码的NLP处理，将之转化为更高级的语言比如汇编之后输入模型，另一种 treat the **bytecode** as a **sequence of tokens** and input it into a sequence-to-sequence NLP model;
   2. Embeddings in NLP
      1. embedding 是 word、token（记号）的密集、连续向量值表示。目的：以数字形式表示单词或者记号，并且可以作为NLP model的输入。Embeddings used in NLP to represent  words and tokens in a way that **captures the semantic meaning and relationships** between words.
      2. Why embeddings are useful in NLP?
         1. 模型为每个单词获得单独的权重，这样处理大量词汇更有效率；
         2. embeddings 可以捕捉单词之间的关系：同义词、类比 which can be useful for tasks such as language generation or translation.
         3. 提高模型的泛化性能，embeddings 允许模型处理词汇表以外的单词或训练中没有看到的单词。

## 3⃣️：TXRANK OVERVIEW

1. System Model
   1. Transaction and State Transition：

      $$
      S' = tx(S)
      $$
   2. Blockchain Nodes的任务：交易排序、确定区块内交易的顺序、产生区块、数据验证、数据传播；

      1. Sequencer nodes(序列化节点)：pow中的miner、pos中的validator、PBS中的block builder;Sequencer可以插入、省略、重排transaction（in the blocks they create within the boundaries set by the protocol）
      2. 普通节点ordinary node：仅处理区块链数据传播、也可能执行数据验证。
   3. 交易传播：将transaction从 transaction generator 传播到 sequencer node的两种方法；

      1. Public propagation：transaction 可以在P2P网络中 通过transaction generator 传递给sequencer nodes
      2. Private propagation：FaaS（Front-running as a Service）允许defi交易者不通过P2P网络直接提交交易给Sequence nodes。
   4. transaction execution trace：记录处理事务产生的操作序列和状态更改，transaction trace的多种表现形式：比如低级OP code 和 高级DeFi操作。
2. Threat Model：设置一个计算受限的adversary（对手方），记作A。其实就是攻击方，A可能会利用一些漏洞来试图使Defi协议发生非预期的状态转换。
   1. 可观测的对手方：无法向txrank隐藏他们的pending transaction，
      1. txrank可以在P2P网络上进行广播，任何人都可以观察到对手方的交易；
      2. 如果sequencer nodes 使用txrank，即使对手方使用private propagation（FaaS），sequencer依然可以观察到交易；
   2. 隐藏对手方：可以通过FaaS或者类似的方式隐藏他们的交易，直到交易最终确认。
3. TxRank Overview
