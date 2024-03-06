# IPFS的理解

DHT 检索数据存储位置

BIT 进行数据交换

GIT 数据内容版本管理 文件，目录，更新       结合DHT对象是基于内容寻址的  对象之间的连接形成Merkle DAG  大量使用指针和引用，创建和释放对象搞笑，更新只需更新引用或者增加对象，在分布式系统中，版本更新只需传输对象或者更新远程引用

SFS 证明节点的存在，作为数据存储，交换，读取，更改的前提  公私钥验证，平等地共享命名空间



整合上述四个部分 得到IPFS

可分为7部分子协议

身份 管理节点身份生成与验证（如果设置权限应该加在这里）

网络 管理节点之间的通信，（用到的TCP、UDP等协议？）

路由 维护数据寻址的结构 如维护swappable DHT

交换 BitSwap新的块交换协议

对象 MerkleDAG 内容寻址

文件 inspired by Git

命名 自认证可变更

## Identities

- NodeId 用户可以在每次启动时（启动是指什么？）实例化一个新的节点（这不被鼓励，损失了网络应得的利益）

- 当节点首次连接时，其他节点将与它交换公钥，并相互验证`hash(other.PublicKey) equals other.NodeId.`验证不通过则连接终止。

- 上面是Go语言，PKI全称为*Public Key Infrastructure*， Go语言内置的一个包实现各种函数，这里用到了生成一对公私钥的函数，并对公钥取hash得到NodeId，仅当NodeId取hash得到的值满足指定难度时(0的个数足够，类似于比特币挖矿)，此次随机生成的公私钥可以被作为一个有效节点Node
- IPFS中使用的cryptographic hash function 不采用具体的一系列函数，而是支持自描述值（应该可以理解为可以自己选择使用哪些hash function）Hash digest最终以如下形式存储，称之为multihash,由三部分组成:`<function code><digest length><digest bytes>`



## Network

IPFS中的nodes与全世界中的nodes相互连接，相当于在已有的TCP/IP架构的应用层，但是并不一定用了TCP/IP结构

- Transport(传输层)：可以使用任何协议，最推荐WebRTC DataChannels）（Web Real Time Communication）或者uTP(基于UDP协议的改进)（这两种协议在论文中被置于传输层，但实则应该属于TCP/IP模型中的应用层，传输层负责端到端的通信而不在乎传输的具体内容，此处的小标题传输层，应该指的是IPFS协议中负责nodes之间数据传输的协议，有别于TCP/IP模型中的传输层）
- Reliability(可靠性)：即使底层网络不可靠，（应该就是指传输层使用UDP协议时的情况？）IPFS此时也可以使用uTP协议或者SCTP（Stream Control Transmission Protocol）协议(与TCP、UDP协议类似)保证应用层的可靠性（其实IPFS协议也算是一种应用层的协议？只是应用层协议可以有更多层的嵌套，如七层结构中包含了3层：会话层、表示层、应用层）
- Connectivity(连接性)：IPFS使用了ICE NAT穿透技术，使得两个node分别通过两个NAT时也能连接
- Integrity(完整性)：可以选择通过hash checksum检验消息完整性（通过使用哈希校验和，可以快速有效地验证数据在传输或存储过程中是否保持完整性）
- Authenticity（权限性）：可以选择使用HMCA技术结合发送者的公钥检查发送者是否有发送该消息的权限，HMAC主要目的是防止数据在传输过程中被纂改或伪造

### *Note on Peer Addressing*

- 这里作者对IPFS中用到的address这个概念进行了补充解释

- 原文及翻译如下：

  `IPFS can use any network; it does not rely on or assume access to IP. This allows IPFS to be used in overlay networks.IPFS stores addresses as multiaddr formatted byte strings for the underlying network to use. multiaddr provides a way to express addresses and their protocols, including support for encapsulation.for example:
  \# an SCTP/IPv4 connection
  /ip4/10.20.30.40/sctp/1234/
  \# an SCTP/IPv4 connection proxied over TCP/IPv4
  /ip4/5.6.7.8/tcp/5678/ip4/1.2.3.4/sctp/1234/ `
  
  IPFS可以使用任何网络；它不依赖于或假设对IP的访问。这使得IPFS可以在覆盖网络中使用。IPFS将地址存储为多地址（multiaddr）格式的字节字符串，供底层网络使用。多地址提供了一种表示地址及其协议的方式，包括对封装的支持。

- 这段话很难理解，经过一番与chatGPT的问答，谈谈我的理解。首先IPFS可以使用任何网络，这里的网络应该指IP网络（通常指使用IP协议，即Internet Protocol协议的网络）以及其他不直接使用IP协议的网络，对于这里的不直接使用IP协议的网络，chatGPT给出了下面的例子：
  1. **局域网（LAN）**：局域网是一种在相对较小的地理范围内组建的网络，通常位于同一建筑物或办公区域。局域网通常使用Ethernet或Wi-Fi等协议，而不直接使用全球范围内的IP协议。
  2. **CAN网络（Controller Area Network）**：CAN网络是一种广泛用于汽车和工业控制系统的网络。CAN网络采用特定的物理层和数据链路层协议，并不直接使用IP协议。
  3. **蓝牙网络**：蓝牙是一种用于短距离无线通信的技术，它用于连接蓝牙设备，如手机、耳机、键盘等。蓝牙网络不使用IP协议，而是使用蓝牙协议栈。
  4. **CANet（Content Addressable Network）**：CANet是一种特殊的分布式网络，用于实现内容寻址，不依赖于IP地址。

​		这样就能理解不依赖或假设对IP网络的访问的含义，而后面的覆盖网络，chatGPT给出的解释如下：

> ***Overlay networks（覆盖网络）是计算机网络中的一种网络架构，它建立在底层物理网络之上，通过逻辑上的连接将多个计算机或网络设备组合成一个虚拟网络。在覆盖网络中，网络节点之间的通信不依赖于底层物理网络的拓扑结构，而是依赖于覆盖网络定义的逻辑连接。IPFS可以在覆盖网络中工作，这意味着它可以在不同类型的网络环境中实现通信和数据传输，而不仅限于传统的IP网络。***

​		基于这个解释，上面一些非IP网络应该都属于***Overlay network***的范畴，进一步科普：

> 覆盖网络的构建可以通过软件定义网络（SDN）技术实现，也可以通过应用层的协议和中间件实现。它在计算机网络中有广泛的应用，用于解决一些特定的问题或提供特定的服务。覆盖网络的一些常见用途和特点包括：
>
> 1. **虚拟专用网络（Virtual Private Networks - VPN）**：VPN是覆盖网络的一种常见用途。通过VPN，远程用户或分支机构可以通过公共互联网安全地访问公司的内部网络，实现远程办公和资源共享。
> 2. **容器网络（Container Networking）**：在容器化应用中，不同容器之间的通信可以通过覆盖网络实现，使得容器之间的连接独立于底层主机网络。
> 3. **内容分发网络（Content Delivery Network - CDN）**：CDN通过在全球范围内部署服务器节点，提供高效的内容传输和分发服务。这些节点可以形成覆盖网络，从而将内容快速传递给最终用户。
> 4. **区块链网络**：区块链网络中的节点可以通过覆盖网络相互连接，实现分布式账本的共识和同步。
>
> 覆盖网络的优势在于它可以在现有的物理网络基础上构建更灵活、安全和高效的网络架构。通过逻辑连接和隔离，覆盖网络可以提供更多的网络功能和服务，并且可以适应不同的应用需求。

- 随后是对IPFS中***address***的理解，对于作者所说的多地址，作者举例对其进行了说明：第一个为单地址，指明了传输层使用SCTP协议，网络层使用IPv4协议，再加上一个端口号；第二个为多地址，只是叠加了一个单地址，并且新增的单地址中可以使用不同的传输层协议TCP。这种“***multiaddress***”的定义，，提供了一种通用的方式来表示网络地址和协议，并支持不同网络协议的封装，使得该地址可以再不同类型网络工作，而不仅限于传统的IP网络。 因此，称IPFS不依赖于或假设对IP的访问（实际上大部分情况下还是会在IP网络上运行，只是也支持覆盖网络），并且能够适应不同的网络拓扑和通信需求。这使得IPFS在覆盖网络中也能够发挥作用，实现分布式数据存储和传输。

## Routing(DHT)

有了node Identity标识节点，有了network连接节点，下一步是节点间如何找到对方的位置呢，routing就是解决这个问题，首先这里要明确的问题是，当IPFS运行在IP网络之上时，ip网络中还是用到ip路由表的，ip路由表解决的是点对点之间的寻址问题，而此处IPFS协议中的路由解决的是应用层层面端到端中node间的寻址问题，端到端本身的寻址问题本来不存在，是由网络层向传输层提供服务的，但是这里要解决查找存放数据的node在哪个ip上，故需要一个应用层层面的路由表。

- routing system that can find
  1. other peer's network address 寻找节点的网络地址
  2. peers who can serve particular objects 寻找可以提供所需数据的节点

​		即two-stage 先找节点，再找节点网络地址，最后就能通过network实现传输

- DSHT based on S/Kademlia and Coral

- IPFS采用对不同size的metadata使用不同的存储方法，≤1KB直接存在DHT上，否则存储数据的引用在DHT上

- ```go
  type IPFSRouting interface {
      FindPeer(node NodeId)
      // gets a particular peer’s network address
      SetValue(key []bytes, value []bytes)
      // stores a small metadata value in DHT
      GetValue(key []bytes)
      // retrieves small metadata value from DHT
      ProvideValue(key Multihash)
      // announces this node can serve a large value
      FindValuePeers(key Multihash, min int)
      // gets a number of peers serving a large value
  }
  ```



## Block Exchange - BitSwap Protocol

上面三节已经能体会到作者bottom up的介绍顺序，下面一层协议是数据块交换协议，解决具体发生数据交换时的问题

数据交换的论文：《FairSwap:How to fairly exchange digital goods》；《Blockchain Based Non-repudiable IoT Data Trading Simpler Faster and Cheaper》

### *BitSwap Credit*

- 一个类信用系统
  - 节点间寻求平衡
  - 节点向欠债节点发送数据块的概率与债务量成反比
- 一个节点可以向另一个节点发送`ignore_cooldown`持续地忽略一个节点一定时间，以避免消耗计算概率所用资源

### *BitSwap Strategy*

- BitTorrent 采用tit-for-tat策略
- BitSwap应支持多种协议（不论善意或恶意）aim to
  - 最大化交易性能 对于一个节点or对于整个网络
  - 防止freeloaders利用并降低交易质量
  - 对其他未知策略有效且具有抵抗力（chatGPT翻译，不太理解，be effective with and resistant to other, unknown strategies）
  - 对可信节点宽容
- 具体的策略将在未来的工作中探索
- 一种在实际应用中表现良好的是sigmoid
  - <img src="C:\Users\zipin\AppData\Roaming\Typora\typora-user-images\image-20230728001830483.png" alt="image-20230728001830483" style="zoom: 67%;" />
  - <img src="C:\Users\zipin\AppData\Roaming\Typora\typora-user-images\image-20230728001843390.png" alt="image-20230728001843390" style="zoom: 67%;" />

### *BitSwap Ledger*

- 维护账本来追踪历史避免纂改（但这个账本似乎与区块链的分布式账本不太一样）

- ```go
  type Ledger struct {
      owner NodeId
      partner NodeId
      bytes_sent int
      bytes_recv int
      timestamp Timestamp
  }
  ```

  

### *BitSwap Specification*

```go
// Additional state kept
type BitSwap struct {
    ledgers map[NodeId]Ledger
    // Ledgers known to this node, inc inactive
    active map[NodeId]Peer
    // currently open connections to other nodes
    need_list []Multihash
    // checksums of blocks this node needs
    have_list []Multihash
    // checksums of blocks this node has
}
type Peer struct {
    nodeid NodeId
    ledger Ledger
    // Ledger between the node and this peer
    last_seen Timestamp
    // timestamp of last received message
    want_list []Multihash
    // checksums of all blocks wanted by peer
    // includes blocks wanted by peer’s peers
}
// Protocol interface:
interface Peer {
    open (nodeid :NodeId, ledger :Ledger);
    send_want_list (want_list :WantList);
    send_block (block :Block) -> (complete :Bool);
    close (final :Bool);
}
```

对Interface实现的解释和理解

- ***Peer.open(NodeId, Ledger)***
  - 发送方初始化一个连接并发送带有Ledger的Open messeage给接收方，接收方根据Open message中携带的Ledger判断发送方是否值得信任（根据帐本中的交易量和负债信息），并可以选择建立连接或者忽视请求（以ignore_cooldown）
  - 如果连接被激活，接收方需要初始化一个Peer对象（包含本地的Ledger,并设置last_seen参数），之后接收方将对比二者的Ledger，如果匹配，连接才会被打开，否则接收方的Peer创建一个空白Ledger并发送（这个行为我不太理解？）
- ***Peer.send_want_list(WantList)***
  - 何时Peer会发送该请求？每次发送会向所有连接的Peers同时发送
    - 当打开某个连接
    - 在一段随机时间后
    - 当want_list发生变化时
    - 收到一个新的数据块时
- ***Peer.send_block(Block)***
  - 数据块发送很直白，接收数据方壶计算数据块的Multihash检验数据块是否为自己想要的，再返回确认
  - 收到数据块时，将该block从need_list中转入have_list中，双方都会更新Ledgers
  - 如果数据块的验证结果为否，接收方可以认为发送方故障或恶意攻击并拒绝后续交易
- ***Peer.close(Bool)***
  - close 的最后一个参数表明发送者是否有意断开连接，如果不是，接收方可以立刻选择重新连接，避免意外的连接断开
  - 连接应在满足以下两个条件中任意之一时断开
    - silence_wait 持续30秒（BitSwap默认时间）未收到任何message，节点发出Peer.close(false)
    - 节点存在但是BitSwap被关闭？ 节点发出Peer.close(true)
  - 当发出close message后，双方都会关闭连接，清楚存储的状态信息，但是Ledger将被存储



## Object Merkle DAG

DHT和BitSwap使得IPFS构建了巨大的P2P系统以存储和分发数据块，在此基础上，IPFS搭建了Merkle DAG，这使得IPFS有如下有用的属性：

- 内容寻址
- 纂改抵制性
- 去重性

IPFS定义如下：

```go
type IPFSLink struct {
    Name string
    // name or alias of this link
    Hash Multihash
    // cryptographic hash of target
    Size int
    // total size of target
    }

type IPFSObject struct {
    links []IPFSLink
    // array of links
    data []byte
    // opaque content data
}
```

一个IPFS对象包含了一个连接数组和存储数据的字节数组，连接数组元素为IPFSLink类型，IPFSLink类型存储了连接名，多重哈希值和大小， IPFS还有一个重要概念CID

IPFS Merkle DAG提供了一种非常灵活的方式存储数据，唯一的需求是对象引用需要为内容寻址，且采用上述格式进行编码（对象引用为内容寻址含义是对象通过其内容的哈希值来唯一标识，而不是对象名称或位置。

对象内部的链接表使得IPFS可以实现下面这些功能：

- 列出本对象链接表中的所有对象引用 即本对象中记录的其他对象的哈希值等信息

  - ```go
    <object multihash> <object size> <link name> 
    ```

- 解析字符串路径进行查找 如foo/bar/baz
- 递归地解析所有被引用的对象

谈谈自己的理解，首先这里有两个概念要分清楚，DHT和Merkle DAG。1.DHT是指对一个数据块取hash得到哈希值，根据该hash值去找到对应的存储该数据块的节点，节点以CID来标识，故DHT实现的路由是根据数据块内容定位节点。2.Merkle DAG实现的是IPFSObject之间的连接，这里的Object根据我的理解从论文给出的go语言代码定义来看，并不是节点，而是一系列的link和数据，其中的数据可以是多种形式的，link就是其它数据块的名称、哈希值和大小，在Files小节中，作者说明了IPFS采用了类似于Git的四种Object类型：block,list,tree,commit

### *Paths*

Merkle DAG使得Object之间建立了连接（逻辑上的连接，类似数组链表，而不是真的用指针连接），这使得IPFS能够实现字符串路径解析：

```go
# format
/ipfs/<hash-of-object>/<name-path-to-object>
# example
/ipfs/XLYkgq61DYaQ8NhkcqyU7rLcnSa7dSHQ16x/foo.txt
```

上面是路径格式，/ipfs 可以将数据挂载到现有系统中的一个不会产生冲突的标准挂载点，挂载点的名称可以配置,第二个路径组成 <hash-of-object> 即对象的哈希值。 IPFS中没有全局根节点，分布式环境中如果存在根节点，不可能解决数百万个对象的一致性问题。 作者说”故我们采用内容寻址来模拟根节点“，我的理解是将对象的哈希值作为了一个类似于根路径的地位，并且我们是可以通过DHT由对象的哈希值来找到对象存储的位置的，故所有对象都可以通过哈希值进行访问。第三个路径组成就是具体要访问的数据块了（这里是我自己的理解，更具体地将，比如一个foo.txt文件应该被分成很多个数据块，故并不会只存储在一个节点上，作者在这并未讨论此问题）

```go
/ipfs/<hash-of-foo>/bar/baz
/ipfs/<hash-of-bar>/baz
/ipfs/<hash-of-baz>
```

上面三种路径下，在第一个路径种，给定三个对象，最后一个对象baz对所有人都是可访问的

### *Local Objects*

IPFS客户端根据节点的用途和需求，使用不同类型的本地存储和外部系统来管理和存储IPFS网络中的数据对象，从而实现高效的数据共享和传输。IPFS中的所有数据块都存储在某个节点的本地存储中。当用户请求对象时，这些对象会被找到、下载并临时存储在本地。这样可以在接下来的一段时间内快速查找这些数据。

Local一词是重点，一个是节点存储数据的方式，一个是请求数据的节点会对数据对一个临时性的备份（类似于操作系统中CPU访问内存会在cache中做备份）

### *Object Pinning*

Pinning的含义在于，当一个节点想要确保某些数据保存在本地时，可以通过Pinning操作执行，并且该操作可以递归执行，即PIN的对象的所有后代对象也会被一并固定。（个人理解，这无疑大大增加了本地存储的负担）

### *Publishing Objects*

IPFS是一个全球分布式的网络系统，可以实现数据的全球范围共享和传输， IPFS使用DHT（分布式哈希表）和内容哈希寻址来支持对象的发布和查找。DHT允许以公平、安全和完全分布式的方式在网络中发布对象，其中对象的标识是通过内容哈希值进行寻址的，确保了对象的唯一性和完整性。任何人都可以通过对象的键添加到DHT中来发布一个对象，将自己作为对等节点加入网络，然后向其他用户提供对象的路径。这样，其他用户就可以根据路径访问和获取对象。

值得注意的是，类似于Git中的Commit，如果对象发生变化，将生成一个全新的对象，其哈希值不同于之前的对象，这意味着IPFS不直接处理版本控制，而是增加额外的版本控制对象跟踪和管理对象的不同版本（代价是增加了存储和管理开销）

### *Object-level Cryptography*

对象层级的加密，这指的是加密的单位不是文件不是节点，而是包含数据和连接的对象，对象和数据块的概念是不一样的，这在下一节Files中很容易知道，对对象进行加密操作会使得对象的哈希值随之改变，相当于定义了一个不同的对象，在解密之前，该对象无法链接到加密前的链接对象。

IPFS允许加密和签名操作，并支持灵活的加密方案

![image-20230731140344190](C:\Users\zipin\AppData\Roaming\Typora\typora-user-images\image-20230731140344190.png)

## Files

在Merkle DAG之上，定义了版本化的文件系统，这种文件系统与Git类似，共有四种对象类型，依次为：

1. block:可变大小的数据块
2. list:数据块或其他list的合集
3. tree:数据块，list,或其他tree的合集
4. commit:tree的版本历史的快照

作者表示原本想用与Git中完全一致的object格式，但是最后还是引入了一些他认为在分布式系统中有用的特征：

1. fast size lookups
2. large file dedupliacation
3. embedding commits into trees

这些特征是在Git的object之上增加的，可以很方便地将Git objects转化为IPFS file objects，并且不损失任何信息

### *File Object: blob*

blob对象包含了一个可寻址的数据单元，并表示一个文件，IPFS files既可以用list表示，也可以用blob表示，blob没有links

```json
{
    "data": "some data here",
    // blobs have no links
}
```

### *File Object: list*

list对象表示一个由数个blob或list对象构成的大文件或去重文件。从某种意义上说，IPFS列表的功能类似于具有间接块的文件系统文件。列表可以包含其他列表，因此可以构建包含链接列表和平衡树的拓扑结构。通过在文件中使用同一节点的多个位置，可以实现文件内去重。当然，由于哈希寻址的限制，循环是不可能的。

嵌套与链接使得能够构建拓扑结构，进而能够实现去重与寻址。

```json
{
    "data": ["blob", "list", "blob"],
    // lists have an array of object types as data
    "links": [
    { "hash": "XLYkgq61DYaQ8NhkcqyU7rLcnSa7dSHQ16x",
    "size": 189458 },
    { "hash": "XLHBNmRQ5sJJrdMPuu48pzeyTtRo39tNDR5",
    "size": 19441 },
    { "hash": "XLWVQDqxo9Km9zLyquoC9gAP8CL1gWnHZ7z",
    "size": 5286 }
    // lists have no names in links
	]
}
```

### *File Object: tree

tree对象表示目录，和Git一样，它表示了名称到哈希值的映射，这些哈希值可以指向blob,list,tree,commit ,传统的路径命名已经通过Merkle DAG实现。

```json
{
    "data": ["blob", "list", "blob"],
    // trees have an array of object types as data
    "links": [
    { "hash": "XLYkgq61DYaQ8NhkcqyU7rLcnSa7dSHQ16x",
    "name": "less", "size": 189458 },
    { "hash": "XLHBNmRQ5sJJrdMPuu48pzeyTtRo39tNDR5",
    "name": "script", "size": 19441 },
    { "hash": "XLWVQDqxo9Km9zLyquoC9gAP8CL1gWnHZ7z",
    "name": "template", "size": 5286 }
    // trees do have names
    ]
}
```

### *File Object: commit*

commit对象表示任何object在版本历史中的快照

```json
{
    "data": {
    "type": "tree",
    "date": "2014-09-20 12:44:06Z",
    "message": "This is a commit message."
    },
    "links": [
    { "hash": "XLa1qMBKiSEEDhojb9FFZ4tEvLf7FEQdhdU",
    "name": "parent", "size": 25309 },
    { "hash": "XLGw74KAy9junbh28x7ccWov9inu1Vo7pnX",
    "name": "object", "size": 5198 },
    { "hash": "XLF2ipQ4jD3UdeX5xp1KBgeHRhemUtaA8Vm",
    "name": "author", "size": 109 }
    ]
}
```

### *Version control*

提交对象表示对象版本历史中的特定快照。比较两个不同提交的对象（和子对象）可以揭示文件系统两个版本之间的差异。只要可以访问单个提交及其引用的所有子对象，就可以检索所有之前的版本，并访问文件系统更改的完整历史记录。这是Merkle DAG对象模型的自然结果。

Git版本控制工具的所有功能都可以在IPFS中使用，IPFS与Git的对象模型是兼容的，尽管并不完全相同。可以通过以下方式实现：(a) 构建修改后使用IPFS对象图的Git工具版本，(b) 构建一个挂载的FUSE文件系统，将IPFS树挂载为Git仓库，将Git文件系统的读写操作转换为IPFS格式。

### *Filesystem Paths*

在Merkle DAG小节中，我们已经看到IPFS对象可以通过字符串路径API追溯到，IPFS文件对象的设计旨在简化将IPFS挂载到UNIX文件系统上。它们限制了树对象不包含实际数据，以便将其表示为目录。而提交对象可以被表示为目录，也可以完全隐藏在文件系统中。

### *Splitting Files into Lists and Blob*

版本控制和分发大型文件的主要挑战之一是找到将它们拆分为独立块的正确方法。IPFS不假设可以对每种类型的文件都做出正确的决策，而提供以下几种选择： (a) 使用Rabin指纹[?]，例如在LBFS[?]中，来选择适当的块边界。 (b) 使用rsync[?]滚动校验算法，以检测在版本之间发生了变化的块。 (c) 允许用户指定高度针对特定文件优化的块拆分函数。

- (a) 使用Rabin指纹：Rabin指纹是一种用于快速切分文件的方法，例如在LBFS中使用。它可以帮助选择适当的块边界，以实现高效的文件拆分。
- (b) 使用rsync滚动校验算法：rsync算法用于检测文件在不同版本之间发生了哪些变化。它可以帮助识别已更改的块，以便在进行版本控制时只传输更改的块。
- (c) 允许用户指定块拆分函数：IPFS允许用户自定义块拆分函数，特别针对某些特定类型的文件进行优化。这样用户可以根据文件的特性，选择最适合的拆分方式。

### *Path Lookup Performance*

基于路径的访问需要遍历对象图。检索每个对象需要在DHT中查找其键值，连接到对等节点，并检索其数据块。这会带来相当大的开销，特别是在查找具有许多组件的路径时。为了减轻这种开销，采取了以下措施：

- **树缓存**：由于所有对象都是通过哈希地址定位的，它们可以被无限期地缓存。此外，树对象的大小通常较小，因此IPFS优先将它们缓存而不是数据块（blobs）。
  - **优先缓存树对象：** 树对象在IPFS中通常表示目录结构，它们往往比数据块（数据对象）的大小要小。由于树对象的大小相对较小且不容易发生变化，IPFS优先将树对象缓存起来，而不是数据块。这样做的目的是在资源有限的情况下，优先缓存对整个文件系统结构起着重要作用的树对象，以提高整体性能和访问速度。
  - 综合而言，IPFS采用树缓存机制，可以将对象缓存并长期保存，从而提高文件系统的性能和访问效率。通过优先缓存树对象，IPFS可以更有效地管理资源，确保重要的文件结构在缓存中得到优先处理，提供更快速的数据访问体验。这种缓存策略使得IPFS在处理大型文件和目录时更加高效和可靠。

- **展开的树**：对于任何给定的树，可以构建一个特殊的展开树来列出所有从该树可达的对象。展开树中的名称实际上是从原始树开始的路径，带有斜杠分隔。



## **IPNS: Naming and Mutable State**

到目前为止，IPFS协议栈形成了一个点对点的块交换网络，构建了一个内容寻址的对象有向无环图（DAG）。它用于发布和检索不可变的对象，并且甚至可以跟踪这些对象的版本历史。然而，有一个关键组件缺失：可变命名。如果没有可变命名，所有新内容的通信都必须在外部进行，发送IPFS链接。因此，需要一种方法在同一路径上检索可变状态。

这是值得阐明的原因——为什么我们在最初努力构建一个不可变的Merkle DAG（有向无环图）。考虑一下IPFS的特性，这些特性都是由Merkle DAG带来的：对象可以通过它们的哈希值进行（a）检索，（b）进行完整性检查，（c）链接到其他对象，并且（d）可以被无限期地缓存。从某种意义上说：对象是永久的。

IPFS中最初选择构建一个不可变的Merkle DAG，由此带来的一些重要特性和优势：

1. **不可变性和哈希检索：** 在IPFS中，对象的哈希值是它们唯一的标识符，通过哈希值可以高效地定位和检索对象。由于Merkle DAG中的对象是不可变的，一旦对象被创建后，它们的内容和哈希值是固定的，不会发生改变。这确保了IPFS中的对象可以被准确地通过哈希值进行检索，而无需担心对象被篡改或更改。
2. **完整性检查：** 由于对象的哈希值是根据其内容计算得出的，因此哈希值可以用于验证对象的完整性。接收方可以通过重新计算对象的哈希值并与收到的哈希值进行比较，来验证对象是否在传输过程中被修改过。
3. **链接到其他对象：** IPFS中的对象可以通过链接到其他对象来建立关联。由于Merkle DAG的特性，这些链接也是通过哈希值实现的，这样可以确保链接的目标对象是确定的，而且不会发生改变。这种链接能力使得IPFS可以构建复杂的数据结构，实现高效的数据共享和组织。
4. **无限期缓存：** IPFS中的对象是可被缓存的，因为它们通过哈希值进行标识。这意味着一旦对象被检索，它们可以被缓存在本地节点中，以便在未来的访问中可以直接从缓存中获取，而无需再次下载。由于对象的不可变性，缓存可以被无限期地保留，提供更快速的访问体验。

这些特性使得IPFS中的对象是永久的，可以被高效地定位、验证和共享，为分布式存储和内容传输提供了强大的基础和保障。

Merkle DAG、不可变的内容寻址对象和命名，以及可变的指向Merkle DAG的指针，形成了许多成功的分布式系统中普遍存在的二分法。其中包括Git版本控制系统，它具有不可变的对象和可变的引用；以及Plan9 [?]-UNIX的分布式后继系统，它具有可变的Fossil[?]和不可变的Venti[?]文件系统。LBFS [?] 也使用可变的索引和不可变的数据块。

1. **Merkle DAG和不可变对象：** 在IPFS中，Merkle DAG是一个有向无环图，用于构建内容寻址的不可变对象。每个对象都通过其内容的哈希值进行标识，一旦对象被创建，其内容和哈希值就是固定的，不可更改。这种不可变性确保了对象在整个网络中的唯一性和数据完整性。
2. **Naming（命名）和可变指针：** 为了实现可变状态和动态数据交互，IPFS引入了Naming机制，例如IPNS。Naming允许在Merkle DAG中使用可变的标识符与特定路径上的内容进行关联。通过这种方式，可以在同一路径上检索到最新的可变状态。
3. **类似于其他分布式系统：** 这种二分法（不可变对象和可变指针）在许多其他成功的分布式系统中也存在。例如，Git版本控制系统中，对象是不可变的，而分支和标签等是可变的，用于跟踪版本历史和标记重要的提交。Plan9是UNIX的分布式后继系统，其中也存在不可变的Venti文件系统和可变的Fossil文件系统。LBFS也使用了类似的机制，其中索引是可变的，而块（数据对象）是不可变的。

### *Self-Certified Names*



###  *Human Friendly Names*



- *Peer Links.*
- *DNS TXT IPNS Records.*
- *Proquint Pronounceable Identifiers.*
- *Name Shortening Services.*



## *Using IPFS*

