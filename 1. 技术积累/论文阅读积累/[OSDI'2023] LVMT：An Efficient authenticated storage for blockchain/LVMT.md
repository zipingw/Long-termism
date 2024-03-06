# LVMT

[B站视频链接](https://www.bilibili.com/video/BV1sF4m1V7SP/?spm_id_from=333.999.0.0&vd_source=b31fa12489c4572ba8d42132c2a313cc)

## 基本概念

authenticated storage process ??? 将一个区块中的交易进行记录时需要授权

在MPT存储结构中 access会在磁盘IO操作中会被放大到O（logn）级别



proposed multi-Layer Versioned Multipoint Trie (LVMT) :使用了authenticated multipoint evaluation tree (AMT)

并在AMT基础上 使用多层设计 支持无限制的键值对和存储版本号 来替代value hashed ，来避免 椭圆曲线乘法运算 的复杂运算



authenticated的过程 light node 访问 full node ,state root



每修改一个leaf node，都需要对整棵Tree自下而上地进行更新，O（logn），每次交易至少改变两个leaf node，代价很大



两大工具： 1. AMT 和  2. append-only Merkle tree

AMT: cryptographic vector commitment scheme



AMT会导致一些问题：  

1. 椭圆曲线乘积运算  直接引入AMT会导致复杂运算  反而更慢了，尽管理论上减少了更新Tree的时间复杂度,  LVMT采用了一种新的设计： key-versioned-value  , 由此LVMT 使用AMT来维护key-version pairs并用Merkle Tree维护一个key-version-value triples的append-only认证列表

2. 如果采用AMT结构，那键值是区块链中的地址，论文中所说的blockchain state keys,通常这个键值的bit-depth即位宽是比较长的，具有k位密钥空间的AMT需要2^k个椭圆曲线点的公共参数，32bit需要256GB预先计算的元数据，而一般blockchian ledger keys是256bit的，为了解决这个问题LVMT采用多级多插槽结构，每个AMT再这个结构中都是16位密钥空间，这样每棵AMT只需要64KB的预先计算参数，并且这个结构允许自动生成sub-AMT在下一层，以此来适应keys-version pair

3. AMT的引入使得更新root hash只需要常数时间，但是maintaining the proof generation metadata还是需要O（logn）级别的时间复杂度，引起和MPT一样的放大程度，LVMT使用proof sharding technique来解决这个问题，将proof generation metadata分散到多个节点上，LVMT中每个full node只需要保存a shard of the blockchain state的proof generation metadate。  作者发现通常在production blockchain中由数千个full nodes，让所有full node都保存total state的proof generation是没有必要的。 即使 sharded之后，对于part of the state，也会有足够的node为light clients提供proof generation requests的服务。 在现在的Ethereum生态中，大部分的light nodes通过specialized providers访问full nodes，如Infura就采用几个full nodes来平衡query workload，



作者将LVMT整合进了Conflux中，具有智能合约支持的开源高性能区块链产品，我们对比了LVMT和OpenEthereum中的MPT的实现、RainBlock's MPT 、LMPTs，并且单独对读写的workload以及端到端的blockchain processing tasks进行了评估 ， 结果表明LVMT在随机的state read/write 操作中达到了10倍的throughput，当与高性能区块链端到端集成时，simple payment transaction的throughput上提升了2.7倍，在ERC20 token transfer transactions中提升了2.1倍。       这些提升都来源于disk I/O amplification的显著减少，就I/O amplification而言，LVMT相比MPT 在read上提升了4.1倍，在write上提升了8.2倍



### Authenticcated Storage in Blockchain

在permission-less blockchain system中，有两种nodes,full node 同步并执行所有的transaction 而light node只同步block headers。  

1. 当 full node提出一个new block 这个block中的所有transaction都需要被执行，并且将所有交易执行后的ledger state的commitment合并到blockheader中。     这个node在执行交易时 要维护一个write-back cache写回寄存器， 执行完毕后将所有修改提交到存储。  执行引擎中需要两个interfaces 为authenticated storage提供服务：
   - Get(k) -> v
   - Set({(k, v)i},  e) -> comm  将一系列键值对存进某个块号为e的block中，并返回新的账本状态的commit

2. 当light node需要某一个key对应的value时，将会query full node，希望得到一个带有proof的value，随后轻节点将会检查是否该commitment是否存在于已经被验证为合法的承诺集合中，再检查proof是否合法，所以authenticated storage需要为轻节点提供两个interface:
   - Respond (k) -> (v ，Π， comm)
   - Verify (k, v, Π ，comm) -> true / false



### Elliptic Curve Group

### KZG

### Authenticated Multipoint Evaluation Tree

KZG commitment使得做到了常数时间的更新，但还是需要O（n）来构建一个指定位置的proof或者是维护所有位置的proof，在一个区块链系统中，被提交的vector经常改变，KZG commitment不能很有效地生成proofs

为了解决这个问题，Alin等人提出了AMT commitment protocol，这个AMT维护一个大小为O（nlogn）的辅助信息空间，并且可以在O（logn）时间下生成proof，



n = 8    input vector a有8个elements    AMT计算其Lagrange interpolation f（x）使得 *f*(*i*) = ai for 1 *≤* *i* *≤* 8.



## Overview

交易执行时间的主要开销在于 改变区块链世界状态  在ERC-20智能合约中， 读写操作占据67%的时间，

proposed system是基于 AMT 的 ， 因为AMT有一个理想的时间复杂度，即以常数时间更新commitment，

几个问题：

1. constant ratio 是很大的， 在i9-10900K CPU机器上进行的micro-benchmark 如下 ， 展示了基本密码学操作的时间消耗 ， 椭圆曲线相乘运算用了约0.1ms甚至比MPT的更新操作还要慢

2. 为了支持最大n个实体，AMT需要预先计算O(nlogn)级别的参数量，并维持一个大小为O（nlog^2 n)的辅助信息空间 因此AMT无法直接支持固定长度字符串的键值对。  因为当区块链账本状态持续增长时，AMT不是一种可扩展的方案
3. 区块链系统必须考虑最慢的节点，即使大多数矿工不需要去维护提供proof的辅助信息，但是认证存储机制必须要保证应答节点必须跟上节奏

为此我们提出了下面几种技术手段：

1.  设计了一个versioned database , 只存储AMT中key的版本号，从而避免了椭圆曲线计算，但是这个设计同样支持确定长度的values,因为values并不保存在AMT中
2. 扩展了AMT 变成多层AMT 为了适应无限制的keys的版本号，使得AMT的size相对比较小。 为了支持确定的key的长度并且最小化多级结构中deep updates ，我们利用了key hasher来为版本号分配slots，
3. proof sharding 来减少单个节点存储辅助信息的cost

### Versioned Key-value Database







AMT如何提供 常数时间的 proof 
