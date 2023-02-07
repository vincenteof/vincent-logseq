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
- ```
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
-