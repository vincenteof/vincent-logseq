- a smart contract like a sort of vending machine
-
- dapp 的优势我大概知道：
	- 几乎不可能宕机，有无数节点
	- 隐私性与去中心化
	- 匿名
	- 数据透明性，服务完全开源，可以搞清楚每项目服务到底干了点啥
- 但是它也有一些缺陷：
	- 开发者很难修改部署的代码和数据（实际如何更新版本？）
	- 性能很难优化，因为区块链本身的性质
	- 要想作出 user-friendly 的产品，势必要引入中心化特性
-
- 以太坊里的两种账户类型：
	- Externally-owned，即有账户私钥 的人控制行为
	- Contract，行为完全由代码控制
- 一些区别：
	- https://ethereum.org/en/developers/docs/accounts/#key-differences
-
- 交易会让 eth world state 变更
- world state(t) → transaction → world state(t + 1)
- 一次交易大概有如下的一些字段：
- ```json
  {
    from: "0xEA674fdDe714fd979de3EdF0F56AA9716B898ec8",
    to: "0xac03bb73b6a9e108530aff4df5077c2b3d481e5a",
    gasLimit: "21000",
    maxFeePerGas: "300",
    maxPriorityFeePerGas: "10",
    nonce: "0",
    value: "10000000000"
  }
  ```
- 这里面好几个字段都和 gas 有关，gas 是以太坊世界里仍和计算所需要的（具体的 fee 计算方法？）
- 为了能被成功验证和相应，这个 transaction data 还要被用用户的 private key 签名（具体 case？）
-
- 当用户与合约，或者合约与合约交互时，需要 abi。abi 与 api 类似，不过是在二进制角度的。因为部署在以太坊网络里就是一串 byte code，调用者视角而言就是一串地址，当需要与这串地址交互并获取返回数据时，abi 就派上用场了。
	- [https://www.quicknode.com/guides/solidity/what-is-an-abi](https://www.quicknode.com/guides/solidity/what-is-an-abi)
-
- 一次交易的大致过程：
	- 交易数据已生成签名，并由 clinet 发出
	- 交易在网络上被广播，被与期待待打包的交易一起放在一个池子里
	- miner 打包交易到 block
	- 数个区块确认后，你的交易被认为是成功的
- block 就是一批交易，里面还存储着之前 block 的 hash
- ![https://ethereum.org/static/85d784391401f89209d3bcc51e0ea677/302a4/tx-block.png](https://ethereum.org/static/85d784391401f89209d3bcc51e0ea677/302a4/tx-block.png){:height 444, :width 776}
- 一旦某个 block 被 miner 挖到，它会把这个消息广播到网络的其他地方，其他节点会也把这个 block 加到它们自己的 chain 上，来达成共识。（POW）
- block 里含有下面的信息
	- [https://ethereum.org/en/developers/docs/blocks/#block-anatomy](https://ethereum.org/en/developers/docs/blocks/#block-anatomy)
-
- 如果说比特币实现的是分布式账本，那么以太坊实现的是分布式状态机。
-
-
-
-
-
-
-