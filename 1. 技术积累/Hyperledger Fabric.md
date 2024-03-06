[TOC]



# Hyperledger Fabric

## 简介以及链接

- Hyperledger Fabric是由 Linux 基金会支持的开源项目，提供了一种可扩展、可定制和安全的联盟链技术。
- 官网：[🔗](https://www.hyperledger.org/use/fabric)
- paper:[🔗](https://arxiv.org/pdf/1801.10228.pdf)
- 官方文档：[🔗](https://hyperledger-fabric.readthedocs.io/en/release-2.5/)
- 中文文档：[🔗](https://hyperledgercn.github.io/hyperledgerDocs/)
- 中文视频Tutorial：[🔗](https://wiki.hyperledger.org/display/TWGC/Fabric+Video+Tutorial)
- FAQ：[🔗](https://github.com/Hyperledger-TWGC/FAQ)
- 博客(Hyperledger Fabric 2.0系列)：[🔗](https://blog.csdn.net/qq_28540443/article/details/104265844)
- 电子书《区块链技术指南》:[🔗](https://github.com/yeasy/blockchain_guide)
- 电子书《Hyperledger 源码分析之Fabric》:[🔗](https://github.com/yeasy/hyperledger_code_fabric)

- **源码分析**
  - 《Hyperledger Fabric源代码分析与深入解读》
  - Hyperledger Fabric v2.x 最新资料汇总[🔗](https://hello2mao.github.io/2020/04/22/hyperledger-fabric-v2.x-info/)
  - Fabric2.2中的Raft共识模块源码分析[🔗](https://www.cnblogs.com/GarrettWale/p/16131853.html)

## 简易版 Getting Started

如果是初次使用，请跳转至[轻松上手 Getting Started](#轻松上手 Getting Started)，如果没有连外网，部分步骤会被墙出现无法连接的情况，请看[换源](#换源)

### 安装篇

1. 首先要获取`install-fabric.sh`，新建目录`go/src/github.com/user_name`，在该目录下下载

   ```bash
   # linux下的下载命令
   curl -sSLO https://raw.githubusercontent.com/hyperledger/fabric/main/scripts/install-fabric.sh
   # linux中没开VPN会被墙，可以在windows科学上网后下载文件再传至linux
   # windows下的下载命令
   Invoke-WebRequest -Uri "https://raw.githubusercontent.com/hyperledger/fabric/main/scripts/install-fabric.sh" -OutFile "install-fabric.sh"
   ```

2. 授予执行权限，执行`install-fabric.sh`脚本获取`fabric-samples`

   ```bash
   chmod +x install-fabric.sh
   # 执行这条命令时会出现下载失败的情况，如果VPN软件用的是Clash，需要先安装服务模式并启用，再打开TUN模式
   ./install-fabric.sh d s b  
   ```

### 使用篇

- 前言：在`fabric-samples/test-network`存在一个脚本文件`network.sh`，通过执行该脚本可以操作测试网络、通道、智能合约。

1. 测试网络

   ```bash
   cd fabric-samples/test-network
   sudo ./network.sh down
   sudo ./network.sh up
   ```

   

2. 通道

   ```bash
   # 创建channel 
   sudo ./network.sh createChannel [-c channel1]
   # 启动network同时创建channel
   sudo ./network.sh up createChannel -ca
   ```

3. 官方提供的fabric-samples中 有很多文件是没有权限访问的，但是在使用sdk连接测试网络并调用智能合约时会出现访问权限不足的情况，可以通过下面的方式修改相关要访问的文件的权限（这是个笨方法，每次重启网络都要改一遍权限很麻烦，也可能有其他方式）

   ```bash
   cd organizations/peerOrganizations/org1.example.com/
   sudo chmod 777 users/
   cd users/
   sudo chmod 777 User1@org1.example.com/
   cd User1@org1.example.com/
   sudo chmod 777 msp
   cd msp
   sudo chmod 777 keystore/
   cd keystore
   # 如果是priv_sk 就会submit失败
   sudo chmod 777 priv_sk
   sudo chmod 777 e3dd754a07cfcac3b10e4dffd141735c44282f401661b530d59a5bfeb6f32017_sk
   ```

   新版更新文件权限

   ```bash
   cd /home/zpwang/go/src/github.com/zipingw/fabric-samples/test-network
   
   sudo chmod 777 ./organizations/peerOrganizations/org1.example.com/peers/
   sudo chmod 777 ./organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/
   sudo chmod 777 ./organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls
   sudo chmod 777 ./organizations/peerOrganizations/org1.example.com/users/
   sudo chmod 777 ./organizations/peerOrganizations/org1.example.com/users/User1@org1.example.com/
   sudo chmod 777 ./organizations/peerOrganizations/org1.example.com/users/User1@org1.example.com/msp
   sudo chmod 777 ./organizations/peerOrganizations/org1.example.com/users/User1@org1.example.com/msp/keystore/
   cd ./organizations/peerOrganizations/org1.example.com/users/User1@org1.example.com/msp/keystore/
   sudo chmod 777 6ce117e00eafdc80206238f0ff81ec1a321b0b1c78ce0b2031edfe6eda37bbe2_sk
   ```

   

3. 智能合约部署

   ```bash
   sudo ./network.sh deployCC -ccn basic -ccp ../asset-transfer-basic/chaincode-go -ccl go
   sudo ./network.sh deployCC -ccn storage -ccp /home/zpwang/tiny-db/tiny-db-go/storage/chaincode-go -ccl go
   sudo ./network.sh deployCC -ccn storage2 -ccp /home/zpwang/tiny-db/tiny-db-go/storage2/chaincode-go -ccl go
   sudo ./network.sh deployCC -ccn operation -ccp /home/zpwang/tiny-db/tiny-db-go/operation/chaincode-go -ccl go
   sudo ./network.sh deployCC -ccn vTree -ccp /home/zpwang/go/src/timeChain/chaincode/chaincode-go -ccl go
   ```

### 智能合约篇

- Smart contract : implements the transactions that interact with the ledger

    ```bash
    # 创建链码文件夹
    mkdir newcc && cd newcc
    go mod init newcc
    mkdir chaincode-go && cd chaincode-go
    touch newcc.go
    # 编写完newcc.go之后
    go mod tidy
    go mod vendor
    # 一步安装 -ccn后的参数是chaincode name, ccp后面的参数表示要安装的链码在本地的路径
    ./network.sh deployCC -ccn basic -ccp ../ipfscc-basic/chaincode-go/ -ccl go
    ```
    
    ```bash
    # 使用Go一些通用性的相关配置
    # 1. 环境变量配置代理地址
    GOPROXY=https://goproxy.io,direct
    # 2. 打开mod
    sudo GO111MODULE=on
    go mod init # 创建go.mod文件管理依赖包
    go mod tidy # 在go.mod文件中移除不需要的依赖包，执行后生成go.sum文件（依赖下载条目）
    go mod verify # 检查当前模块依赖是否都被下载
    go mod vendor # 生成vendor文件夹，存储具体的依赖包，使go程序能够在无网络环境下运行
    ```





## 轻松上手 Getting Started

在Ubuntu18.04上的部署过程

### Prerequisites

```bash
sudo apt-get install git   # clone fabric project
sudo apt-get install curl  # ?
sudo apt-get -y install docker-compose # -y 表示自动回答 yes
```

```bash
# 验证docker已经安装
docker --version
docker-compose --version
```

```bash
sudo systemctl enable docker # 设置开机自动启动docker
sudo systemctl start docker # 开启Docker daemon 进程
# 现在只有root用户可以运行docker,需要将用户加入到docker用户组中并切换到docker用户组
sudo usermod -a -G docker <username>
newgrp docker # 当前用户同时属于2个用户组，默认是原用户组，需要切换用户组
```

- 如果出现一些下载失败的情况，如果用的是WSL中的Linux，需要在Winodws中配置代理软件 Clash 安装服务模式并打开TUN，使得WSL中也能够访问 外网，若还是出现访问一些网站超时的情况，先ping该网址，再下载一次即可。

```bash
# Install Go 如果需要write Go chaincode or SDK applications
 rm -rf /usr/local/go && tar -C /usr/local -xzf go1.21.1.linux-amd64.tar.gz
 export PATH=$PATH:/usr/local/go/bin # ~/.profile中添加该声明
 source ~/.profile # 使.profile中的声明生效
 go version 
```

```bash
# Install JQ 仅在与channel configuration transaction相关的tutorial中使用到
```

### Install Fabric and Fabric Samples

官方通过Docker compose创建了a simple Fabric test network，并通过一系列应用验证核心功能

官方预编译了`Fabric CLI tool binaries`和`Fabric Docker images`为我们使用

```bash
# 创建工作目录 
mkdir -p $HOME/go/src/github.com/<your_github_userid>
cd $HOME/go/src/github.com/<your_github_userid>
# cURL工具下载install-fabric.sh并授予执行权限
curl -sSLO https://raw.githubusercontent.com/hyperledger/fabric/main/scripts/install-fabric.sh && chmod +x install-fabric.sh
# original script : bootstrap.sh
curl -sSL https://raw.githubusercontent.com/hyperledger/fabric/main/scripts/bootstrap.sh| bash -s
#install-fabric.sh 相比 bootstrap.sh 优化了syntax,提供了积极的选择方式
```

接下来可以执行`install-fabric.sh`来安装fabric，但执行该命令时需要选择参数

```bash
# choose compnents and version
./install-fabric.sh docker samples binary
./install-fabric.sh --fabric-version 2.5.4 binary
```

此外如果要安装contributor版本，需要额外参考

[开发者模式]: https://hyperledger-fabric.readthedocs.io/en/latest/dev-setup/devenv.html

### Fabric Contract APIs and Application APIs

#### Fabric Contract APIs

- [Go contract API](https://github.com/hyperledger/fabric-contract-api-go) and [documentation](https://pkg.go.dev/github.com/hyperledger/fabric-contract-api-go).

下面根据Documents中的Tutorial一步一步开发智能合约并部署

##### Prerequisites

- 在安装Fabric时除了Go都已经安装
  - A clone of [fabric-samples](https://github.com/hyperledger/fabric-samples)
  - [Go 1.19.x](https://golang.org/doc/install)
  - [Docker](https://docs.docker.com/install/)
  - [Docker compose](https://docs.docker.com/compose/install/)

##### Housekeeping

```bash
mkdir fabric-samples/chaincode/contract-tutorial
cd fabric-samples/chaincode/contract-tutorial
go mod init github.com/hyperledger/fabric-samples/chaincode/contract-tutorial
go get -u github.com/hyperledger/fabric-contract-api-go
```

##### Declaring a contract

在`contract-tutorial`目录下创建`simple-contract.go`

所有需要在chaincode中使用的合约都需要implement the [contractapi.ContractInterface](https://godoc.org/github.com/hyperledger/fabric-contract-api-go/contractapi#ContractInterface) 实现该接口最简单的方式是将`contractapi.Contract`struct嵌入到合约中,示例如下:

```go
package main

import (
    "errors"
    "fmt"
    "github.com/hyperledger/fabric-contract-api-go/contractapi"
)

// SimpleContract contract for handling writing and reading from the world state
type SimpleContract struct {
    contractapi.Contract
}
```

##### Writing contract functions

合约中的public function需要满足一些规则,否则在chaincode创建时会报错,下面是实现一个contract fucntion的示例:



##### Using contracts in chaincode

同样在`simple-contract.go`目录下创建一个文件`main.go`

```go
package main

import (
    "github.com/hyperledger/fabric-contract-api-go/contractapi"
)

func main() {
	simpleContract := new(SimpleContract)
    cc, err := contractapi.NewChaincode(simpleContract)
    if err != nil {
        panic(err.Error())
    }
    if err := cc.Start(); err != nil {
        panic(err.Error())
    }
}
```

##### Testing your chaincode as a developer

打开一个terminal,进入到`fabric-samples`目录下的`chaincode-docker-devmode`中,该文件夹下提供了一个定义了一个 simple fabric network的docker compose文件,可以通过该文件运行chaincode

```bash
docker-compose -f docker-compose-simple.yaml.up # 启动fabric network
```

在`chaincode-docker-devmode`目录下打开一个新的terminal

```bash
docker exec -it chaincode sh # enter the chaincode docker container
cd contract-tutorial # 这一步不确定
```

接下来要保证fabric docker image版本在2.x.x,否则`go build`命令会fail , 并且要使用`chmod -R 766`命令使得`contract-tutorial` folder有创建文件的权限

```bash
go mod vendor
go build
# run the chaincode
CORE_CHAINCODE_ID_NAME=mycc:0 CORE_PEER_TLS_ENABLED=false ./contract-tutorial -peer.address peer:7052 
```

##### Interacting with the chaincode

新打开一个终端

```bash
docker exec -it cli sh
peer chaincode install -p chaincodedev/chaincode/contract-tutorial -n mycc -v 0
```



## 换源



```bash
# 换清华镜像源
sudo -s
cp /etc/apt/sources.list /etc/apt/sources.list.bak
# 打开文件
vi /etc/apt/sources.list
```

- 将下面内容粘贴到打开的文件中

```bash
# 默认注释了源码镜像以提高 apt update 速度，如有需要可自行取消注释
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-updates main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-updates main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-backports main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-backports main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-security main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-security main restricted universe multiverse

# 预发布软件源，不建议启用
# deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-proposed main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-proposed main restricted universe multiverse
```

- `:wq`保存文件并执行下列命令

```bash
sudo apt-get update # 更新apt-get

sudo apt-get install git
sudo apt-get install curl
sudo apt-get -y install docker-compose

sudo systemctl start docker
sudo systemctl enable docker
sudo usermod -aG docker <username> # 将当前用户加入到docker用户组中
newgrp docker # 这行是必要的，否则会出现当前用户没有docker的访问权限问题

sudo apt-get install openssh-server
sudo /etc/init.d/ssh start
```



## DeployCC命令的拆解

在测试网络中的任一节点上直接执行 `deployCC` 命令，可以直接安装智能合约，测试网络中共有三个节点，分别是属于 `Org1` 的 `peer1` 节点、属于 `Org2` 的 `peer2` 节点以及 `orderer` 节点，在linux环境中可以通过切换环境变量来控制当前操作哪一个节点（即在哪个节点上执行命令），基于此官网中还提供了另一种逐步安装智能合约的方式，下面为安装过程以及对命令相应的解释：

```bash
export PATH=${PWD}/../bin:$PATH
export FABRIC_CFG_PATH=$PWD/../config/
export CORE_PEER_TLS_ENABLED=true
export CORE_PEER_LOCALMSPID="Org1MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
export CORE_PEER_ADDRESS=localhost:7051
# 教程中的该命令有误 
peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile "${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem" -C mychannel -n basic --peerAddresses localhost:7051 --tlsRootCertFiles "${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt" --peerAddresses localhost:9051 --tlsRootCertFiles "${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt" -c '{"function":"InitLedger","Args":[]}'
```

```bash
# 报错信息
2023-10-16 14:37:09.465 CST 0001 ERRO [main] InitCmd -> Cannot run peer because error when setting up MSP of type bccsp from directory /home/zpwang/fabric/fabric-samples/test-network/organizations/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp: KeyMaterial not found in SigningIdentityInfo
```

这个地方得通过切换为root再在root里声明环境变量，再invoke链码，这个之后我以为下一条命令peer chaincode query可以直接在普通用户中执行，但是还是会出现一样的MSP KeyMaterial not found in SigningIdentityInfo错误。

解决方法跟上面一样，看来涉及到链码的operation都必须要在root用户下执行，应该是权限问题导致无法访问文件就会使得其误以为文件不存在，***直觉上认为可以通过修改org1.example.com/users/Admin@org1.example.com/msp这个文件夹的权限来解决该问题*** 该文件夹应该是由于涉及安全问题设置了较高权限



wsl2中不能用systemctl来启动docker服务，要用service ,service也会启动失败，在windows中docker desktop中的setting=>Resouces=>WSL integration 选中用的系统，再打开Ubuntu就可以用sudo service start docker启动docker

```bash
# 关闭之前的网络
sudo ./network.sh down
# 重新开启网络 并创建通道
sudo ./network.sh up createChannel
# chaincode 要先安装go并配置环境变量
sudo ./network.sh deployCC -ccn basic -ccp ../asset-transfer-basic/chaincode-go -ccl go
# 如果按照教程不切换为root用户直接export环境变量再执行peer chaincode invoke会报错MSP相关KeyMaterial not found in SigningIdentityInfo，如果加sudo执行会显示peer命令不存在或者仍然报错，这里需要先切换为root用户再export环境变量再执行peeer chaincode invoke就会成功了！！！（搞了一下午）
sudo bash
export PATH=${PWD}/../bin:$PATH
export FABRIC_CFG_PATH=$PWD/../config/
export CORE_PEER_TLS_ENABLED=true
export CORE_PEER_LOCALMSPID="Org1MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
export CORE_PEER_ADDRESS=localhost:7051

peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile "${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem" -C mychannel -n basic --peerAddresses localhost:7051 --tlsRootCertFiles "${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt" --peerAddresses localhost:9051 --tlsRootCertFiles "${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt" -c '{"function":"InitLedger","Args":[]}'

# 查看Assets
peer chaincode query -C mychannel -n basic -c '{"Args":["GetAllAssets"]}'
# 转移Assets
peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile "${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem" -C mychannel -n basic --peerAddresses localhost:7051 --tlsRootCertFiles "${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt" --peerAddresses localhost:9051 --tlsRootCertFiles "${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt" -c '{"function":"TransferAsset","Args":["asset6","Christopher"]}'

# 重新export环境变量
export CORE_PEER_TLS_ENABLED=true
export CORE_PEER_LOCALMSPID="Org2MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp
export CORE_PEER_ADDRESS=localhost:9051
# 查看org2上的assets
peer chaincode query -C mychannel -n basic -c '{"Args":["ReadAsset","asset6"]}'
```





```bash
# 下载go.mod中定义的运行代码需要的相关依赖到vendor文件夹中
# 使其在无网络环境下也可也运行
sudo GO111MODULE=on go mod vendor
cd ../../test-network
export PATH=${PWD}/../bin:$PATH
export FABRIC_CFG_PATH=$PWD/../config/
peer lifecycle chaincode package basic.tar.gz --path ../asset-transfer-basic/chaincode-go/ --lang golang --label basic_1.0
# 上述操作可以将链码打包成basic.tar.gz
# 接下来需要将打包后的链码部署到所有需要背书的组织相关的peer上
export PATH=${PWD}/../bin:$PATH
export FABRIC_CFG_PATH=$PWD/../config/
export CORE_PEER_TLS_ENABLED=true
export CORE_PEER_LOCALMSPID="Org1MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
export CORE_PEER_ADDRESS=localhost:7051
# 这里可以领悟到在测试网络中操作peer的方式是通过控制PATH环境变量来切换
peer lifecycle chaincode install basic.tar.gz
# 这里发现又出现了msp不存在的情况，但是无法用曾经的手段解决
# 发现这里的原因是需要重新启动网络 ./network.sh up 再建立一下channel(不清楚是否必要) 这样就可以成功执行 成功submitInstallProposal

#接下来在Org2上面安装chaincode
export CORE_PEER_LOCALMSPID="Org2MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp
export CORE_PEER_ADDRESS=localhost:9051
# 再次安装
peer lifecycle chaincode install basic.tar.gz

# 接下来通过packageID来控制在peer上install的chaincode
# 可以通过下面的命令查看在peer上安装的chaincode的packageID
peer lifecycle chaincode queryinstalled

# 先对Org2进行操作
export CC_PACKAGE_ID=basic_1.0:69b8a2e2397a9d80e59b51b2ec71cb523f064ea5b184336a413b5ea876a957c1
# 执行以下命令使Org2 approve chaincode , approve操作是以组织为单位的，Org2中的一个peer运行了该命令，其他peer也都会通过分布式系统的gossip approve这个chaincode
peer lifecycle chaincode approveformyorg -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --channelID mychannel --name basic --version 1.0 --package-id $CC_PACKAGE_ID --sequence 1 --tls --cafile "${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem"
# sequence 1 表示该chaincode被定义或被更新的次数总和
# approveformyorg 还可以加参数 --signature-policy , --channel-config-policy 
# endorsement policy可以更改这里的配置

# 之后切换PATH到Org1，再次执行 approveformyorg 
```



approve操作 返回了一个txid 这说明peer安装了chaincode并approve这个智能合约也会提交一个tx给Order，因为这样才能将某Org同意了该chaincode的信息传递给其他Org

```bash
# 可以在某个peer上运行该命令 checkcommitreadiness 来查看各Org 对 chaincode 的同意情况
peer lifecycle chaincode checkcommitreadiness --channelID mychannel --name basic --version 1.0 --sequence 1 --tls --cafile "${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem" --output json
```



```bash
# 同意数达到了defination的要求， 则任一Org中的某个Peer可以commit该chaincode to channel
peer lifecycle chaincode commit -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --channelID mychannel --name basic --version 1.0 --sequence 1 --tls --cafile "${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem" --peerAddresses localhost:7051 --tlsRootCertFiles "${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt" --peerAddresses localhost:9051 --tlsRootCertFiles "${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt"
```



```bash
# chaincode defination的上述背书，被channel members提交给Order，Order将该TX放进block分发给channel中的peers,交给peers进行Validation
# 上述执行的commit命令会等待Validation的结果，可以通过querycommitted命令查询Validation结果
peer lifecycle chaincode querycommitted --channelID mychannel --name basic --cafile "${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem"
```



```bash
# 现在chaincode等待被client applications invoke
peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile "${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem" -C mychannel -n basic --peerAddresses localhost:7051 --tlsRootCertFiles "${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt" --peerAddresses localhost:9051 --tlsRootCertFiles "${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt" -c '{"function":"InitLedger","Args":[]}'

# query Assets
peer chaincode query -C mychannel -n basic -c '{"Args":["GetAllAssets"]}'
```

***到这里相当于拆解了一开始 deployCC命令中打包起来的部署智能合约的具体过程***
