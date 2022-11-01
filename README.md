# GethPrivateNode
云服务器搭建以太坊私链，用于合约开发测试


**cloudPATH:**
> ubuntu 20.02
nodjs 18.12
go-1.18.7
geth-1.10.16


about：
> chainName: myPrivate
> RpcUrl: http://47.99.55.27:8500
> chainId: 84537


### 搭建流程

1. make geth

安装 Go 后，可以通过以下方式将 Geth 下载到GOPATH工作区：
>go get -d github.com/ethereum/go-ethereum

或者从官方下载go-etherem 源码
> git clone https://github.com/ethereum/go-ethereum

构建 Geth
>cd go-ethereum
make geth


2. 创世区块创建
配置文件参考：./genesis.json
配置文件主要参数解析：
> config项是定义链配置，会影响共识协议，虽然链配置对创世影响不大，但新区块的出块规则均依赖链配置。
创世区块头信息配置
nonce：随机数，对应创世区块 Nonce 字段。
timestamp：UTC时间戳，对应创世区块 Time字段。
extraData：额外数据，对应创世区块 Extra 字段。
gasLimit：必填，燃料上限，对应创世区块 GasLimit 字段。
difficulty：必填，难度系数，对应创世区块 Difficulty 字段。搭建私有链时，需要根据情况选择合适的难度值，以便调整出块。
minHash：一个哈希值，对应创世区块的MixDigest字段。和 nonce 值一起证明在区块上已经进行了足够的计算。
coinbase：一个地址，对应创世区块的Coinbase字段。
初始账户资产配置
alloc 项是创世中初始账户资产配置。在生成创世区块时，将此数据集中的账户资产写入区块中，相当于预挖矿。这对开发测试和私有链非常好用，不需要挖矿就可以直接为任意多个账户分配资产。

初始化创世区块：
> geth --datadir $HOME/deepeth init genesis.json

执行成功后会在目录下生成相应的两个文件：
> private_chain
├── geth
├── keystore 保存用户信息
└── genesis.json


3. 启动节点
>  geth --datadir ./privateNode --networkid 84537 --mine --nodiscover --http --http.addr "0.0.0.0" --http.port 8500 --http.api personal,eth,net,web3 --allow-insecure-unlock console

参数说明：
>--nodiscover：关闭节点的可发现性，可以防止使用了相同network id和创世块的节点连接到你的区块链网络中（只能通过手动来添加节点）
--maxpeers 0：指定网络中的最多节点数
--http：启用RPC-http服务
--http.api "db,eth,net,web3"：指定启用的RPC API
--http.port "8500"：指定RPC的端口
--http.addr：指定哪些URL可以连接到你的节点,'0.0.0.0'不限制访问地址
--datadir：以太坊区块链的数据目录
--networkid：启动的链ID
--identity "FirstNode"：指定节点名称
console：启动geth控制台程序


4. 导入metamask账户，设置为矿工地址
>1. 导出私钥，保存到linux路径
>2. geth account import $privateKeyPath 生成用户文件
>3. geth account list 查看生成的用户文件 复制到 keystore文件下
>4. 进入geth命令后台查看 eth.accounts
>5. 设置矿工地址 miner.setEtherbase(eth.accounts[0])


5. 启动挖矿
> miner.start(1) 指定线程数启动
或者可以直接命令行启动 geth --mine --miner.threads=4 --miner.etherbase 'address'


6. 设置开机后台启动：
> 后台启动： nohup 'command' & 
查看进程 ps -aux | grep "geth"
> 1.编辑rc.local文件 vi /etc/rc.local
> 2.修改rc.local文件，在 exit 0 前面加入启动命令。保存并退出。


7. 登入节点
> 1. dir 方式
geth attach --datadir ./data
> 2. ipc方式:
geth attach ipc:$datadir/geth,ipc
> 3. http-RPC方式：
geth attach http:localhost:8545


8. 添加Peer节点
注意：同步节点配置一定要一样，
启动另外一个节点（ETH-SlaveNode）时, 可以重新定义节点的identity
> geth --identity "ETH-SlaveNode" --rpc --rpcport "6060" --rpccorsdomain "*" --datadir "/home/donaldhan/Documents/data" --port "30303" --maxpeers 5 --rpcapi "admin,db,eth,debug,miner,net,shh,txpool,personal,web3" --networkid 3131 console

当前我们的有连个节点，我们现在两个节点添加到同一链上。并且进行同步
1. 在其中ETH-MainNode节点的geth控制台中执行：
> admin.nodeInfo

可以获得节点信息
> 
 
> admin.nodeInfo
{
  enode: "enode://f78e7061fbc93ebb03ca7cdb36f2bda17b726cd034ef7d46423af64f8e8040d6fe5379885aa2f40052194421c123627ab4cf311b82d6696856be6b68fe5cb337@127.0.0.1:30303?discport=0",
  enr: "enr:-JC4QE0BLBE3R6dJ1YYMdbWQTUdca46zJiQleYY8yfTSUafjRp2sH-DIvmrFn8MZr0w1rYAsoFtBq54RUEH1lCNyGisBg2V0aMfGhO9Ad0-AgmlkgnY0gmlwhH8AAAGJc2VjcDI1NmsxoQP3jnBh-8k-uwPKfNs28r2he3Js0DTvfUZCOvZPjoBA1oN0Y3CCdl8",
  id: "6d0015752dd6c66aa4b89981f627dc4aa35b82d40df2735c85c55d8e46f09c32",
  ip: "127.0.0.1",
  listenAddr: "0.0.0.0:30303",
  name: "Geth/ETH-MainNode/v1.9.12-unstable/linux-amd64/go1.14",
  ports: {
    discovery: 0,
    listener: 30303
  },
  protocols: {
    eth: {
      config: {
        chainId: 3131,
        eip150Block: 0,
        eip150Hash: "0x0000000000000000000000000000000000000000000000000000000000000000",
        eip155Block: 0,
        eip158Block: 0,
        homesteadBlock: 0
      },
      difficulty: 10000,
      genesis: "0x67c66380b3275fb6fd3e534ef51ca097fc9c5fbb66aed255ec29869bfe85d298",
      head: "0x67c66380b3275fb6fd3e534ef51ca097fc9c5fbb66aed255ec29869bfe85d298",
      network: 3131
    }
  }
}
> 
从上面，我们可以得到enode信息：
> enode: "enode://f78e7061fbc93ebb03ca7cdb36f2bda17b726cd034ef7d46423af64f8e8040d6fe5379885aa2f40052194421c123627ab4cf311b82d6696856be6b68fe5cb337@192.168.230.128:30303?discport=0"

我们需要在另一台ETH-SlaveNode节点上添加ETH-MainNode节点，使用命令：
> admin.addPeer("enode://f78e7061fbc93ebb03ca7cdb36f2bda17b726cd034ef7d46423af64f8e8040d6fe5379885aa2f40052194421c123627ab4cf311b82d6696856be6b68fe5cb337@192.168.230.128:30303")
true

我们可以使用如下命令查看是否添加peer
> > net.peerCount
1
> admin.peers
[{
    caps: ["eth/63", "eth/64", "eth/65"],
    enode: "enode://f78e7061fbc93ebb03ca7cdb36f2bda17b726cd034ef7d46423af64f8e8040d6fe5379885aa2f40052194421c123627ab4cf311b82d6696856be6b68fe5cb337@192.168.230.128:30303",
    id: "6d0015752dd6c66aa4b89981f627dc4aa35b82d40df2735c85c55d8e46f09c32",
    name: "Geth/ETH-MainNode/v1.9.12-unstable/linux-amd64/go1.14",
    network: {
      inbound: false,
      localAddress: "192.168.230.129:34018",
      remoteAddress: "192.168.230.128:30303",
      static: true,
      trusted: false
    },
    protocols: {
      eth: {
        difficulty: 10000,
        head: "0x67c66380b3275fb6fd3e534ef51ca097fc9c5fbb66aed255ec29869bfe85d298",
        version: 65
      }
    }
}]
> 
当net.peerCount大于0,这可以确认节点添加成功，同时我们可以使用admin.peers，查看对应的节点状态；
测试节点同步：
1. 对两个节点进行挖矿
2. 进行一笔两个链的账户交易测试 
3. 查看交易hash上的信息是否同步，余额是否同步 
> eth.getTransaction('blockHash')

信息与MainNode节点信息，一致，表明在同一个链，同时数据一致。



