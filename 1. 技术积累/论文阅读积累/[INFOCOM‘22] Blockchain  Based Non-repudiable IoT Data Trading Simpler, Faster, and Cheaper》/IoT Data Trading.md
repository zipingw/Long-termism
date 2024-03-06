# 01 notes

### target :

 connect and share data at any time among loT smart devices

### problem: 

traditional centralized data sharing/trading machanism  :

1. lacks trust guarantee 
2. cannot satisfy the real-time requirement

### solution:

distributed system, especially blockchain

resolve two problem:

1. credibility
2. real-time limits

#### the proposed scheme(two parts)

- trading scheme:(running in two-round manner)
  - ***divide-and-conquer*** method
  - ***two commitment*** methods

- arbitration scheme
  - ***smart contract***  : solve disputes on-chain in real time
  - off-line arbitration to make a final resolution of on-chain arbitration dissatisfaction

### Why trust matters?

during the proceedings of data trading , both of data buyers and data owner have motivation to cheat others, and the third part cannot assure the trading keep safe.

the current legal system for data trading is not perfect, disputes are hard to resolve, so non-repudiation mechanism is needed.

#### traditional non-repudiation mechanisms

- TTP-based: trusted third party based approaches
  - assumes a trusted third part
  - leads to single point of failure
- probabilistic approaches
  - multiple communication rounds
  - inefficient

#### blockchain-based mechanism

- heavy cryptographic computations and blockchain customization requirements
- critical concern : efficiency 



blockchain records data sending and the corresponding receipt proof, smart contract achieve automatic arbitration using the previously recorded proof



### Technical Aspects: Three Questions

- ensure the non- repudiation 
  - on-chain： stored Tag(S) can be recomputed using digest(S1) and S2, buyer is malicious,
  -   疑问：如何验证数据是buyer需要的数据？如https://sslvpn.zju.edu.cn解决？即数据真实性问题，off-chain
  - off-chain:  submits the trading data to the arbitrator 这里所谓的arbitrator是指发布智能合约的第三方，所谓的background information 又是什么
  - 这里的off-chain arbitration是有问题的，当data buyer提出off-chain arbitration时，需要data owner upload data to the arbitrator off-line
- guarantee the efficiency of the proposed scheme
  - digest(*S*1) = Hash(*S*1) 
  - Tag(*S*) = Hash(*S*1) *⊕* Hash(*S*2)
- minimize the possible disputes
  - incentive mechanism ，actually , is penalized mechanism 