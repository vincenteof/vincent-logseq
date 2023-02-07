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
	- ![https://ethereum.org/static/e8aca8381c7b3b40c44bf8882d4ab930/302a4/evm.png](https://ethereum.org/static/e8aca8381c7b3b40c44bf8882d4ab930/302a4/evm.png)
- 与 redux 类似
	- ```
	  Y(S, T)= S'
	  // S 老状态
	  // T 一堆 transaction
	  ```
- 就以太坊而言，它的状态是一堆堆账户，存储在[modified Merkle Patricia Trie](https://eth.wiki/en/fundamentals/patricia-tree)
- EVM 的模型是一种 stack machine，编译出来的智能合约的 bytecode 执行 EVM 的底层指令（opcode）。
- 所有的以太坊客户端都包含 EVM 的实现
	- [https://ethereum.org/en/developers/docs/evm/#evm-implementations](https://ethereum.org/en/developers/docs/evm/#evm-implementations)
	- ![https://ethereum.org/static/9628ab90bfd02f64cf873446cbdc6c70/302a4/gas.png](https://ethereum.org/static/9628ab90bfd02f64cf873446cbdc6c70/302a4/gas.png)
-
- 1 gwei = 10^-9 ETH
- 1 gwei = 1,000,000,000 wei
- wei 是 以太坊里的最小单位
- gas fee 在交易中的计算变更过
- **London update 之前：**
	- Gas units (limit) * Gas price per unit
	- gas 从转账发起者滑走，全部给矿工
- **London update 之后：**
	- Gas units (limit) * (Base fee + Tip)
	- 区别在于 base fee 会被直接销毁掉，矿工会获得剩下的作为奖励
- 用户还可以设定 max fee（maxFeePerGas），这样与实际 fee 的差额最终会返还交易发起者（本质上是为了解决啥问题？）
- base fee 的计算是基于之前的 block 的，与当前 block 无关，因此可以让交易手续费更为可预测
- 具体算法貌似与压缩有关？
	- [https://www.youtube.com/watch?v=MGemhK9t44Q](https://www.youtube.com/watch?v=MGemhK9t44Q)
-
- 以太坊网络实时
	- [https://etherscan.io/nodetracker](https://etherscan.io/nodetracker)
- **"Node" refers to a running piece of client software.**
- ![https://ethereum.org/static/aab39e6ce2a2a0ec074d19c4613281c1/ac25d/nodes.png](https://ethereum.org/static/aab39e6ce2a2a0ec074d19c4613281c1/ac25d/nodes.png)
- 以太坊世界欢迎用户去跑一个node，这样会让以太坊网络更去中心化。
- 除此之外对用户自身也有一些好处，主要是针对体量比较大的一些用户或者 dapp 的开发者
	- [https://ethereum.org/en/developers/docs/nodes-and-clients/#benefits-to-you](https://ethereum.org/en/developers/docs/nodes-and-clients/#benefits-to-you)
-
-
- 因为以太坊是一个协议，所以会有很多不同的独立网络，互相不会干扰。
- account 是互通的，但是 balance 和 history 是无法从主网上携带到其他网络的。
-
- 理论上来说，一旦有人能控制网络里的 51%，那么他就可以干扰共识机制。
- 不同的共识算法就是采取不同的方式让 51% 攻击更不容易发生。
- 针对与 POW 而言，攻击者需要掌握超过 51% 的算力。This would require such huge investments in equipment and energy; you're likely to spend more than you'd gain.
- POS 是由质押 ETH 的 validator 完成的，validator 被随机选取以产生新的 block。攻击 POS 需要达成掌握系统里超过 51% 的质押的 eth。
- POS 与 POW 只是一种便捷的叫法，实际上决定一个共识算法，即选择出最新的一个 block 的作者，的是 **Sybil resistance 和 chain selection**
- **Sybil resistance** 是指针对女巫攻击的防范能力，女巫攻击我简单的理解为少数人的决策伪造成多数人的决策。POS 与 POW 在解决这个问题上我觉得是类似的，它们本质上没有解决这个问题，只不过在这两种体系下，伪造身份需要资源，即算力或 token，想要伪造大量身份需要大量资源，投入产出比变得很低。
- C**hain selection** 说的是如何去决定一条链是一条正确的链，以太坊和比特币目前都基于最长链原则，即最长的链会被其他节点认为是合法的并与之交互。
- POW + 最长链 = 中本聪共识
-
- smart contract 最终会被编译到 evm bytecode。它们不仅是开源代码，更是不会宕机的 open api service。为了与以太坊网络交互，首先需要连接到以太坊节点。节点存储了全量的以太坊状态并且对完成对交易的共识后在以太坊上写入数据。web app 可以直接与节点交互，或者通过一个 server side 与节点交互。
	- [https://www.preethikasireddy.com/post/the-architecture-of-a-web-3-0-application](https://www.preethikasireddy.com/post/the-architecture-of-a-web-3-0-application)
	- [https://ethereum.org/en/developers/local-environment/](https://ethereum.org/en/developers/local-environment/)
- 只要付出 gas，任何人都可以在以太坊上部署智能合约。
- 智能合约有一个限制，那就是在智能合约内部是感知不到外部世界的事件的，因为无法发送 http 请求。这个目的主要是为了保证以太坊整个计算的纯的，类似 redux，为了保证所有的节点在拿到 transaction 以后最终都能共识到一个状态。这种情况需要靠 oracle 解决，其实就是 blockchain 与真实世界的 bridge。
	- [https://ethereum.org/en/developers/docs/oracles/](https://ethereum.org/en/developers/docs/oracles/)
- 另一个限制是目前 smart contract 最大不能超过 24KB。这个限制貌似可以靠 diamond pattern 解决。
	- [https://eips.ethereum.org/EIPS/eip-2535](https://eips.ethereum.org/EIPS/eip-2535)
-
- 有一种特殊的合约称之为多签合约，这种合约的执行需要多个 signature。好处是可以避免单点失败导致合约里持有大量token？（不是很理解啥意思）这种结构还可以保证单个私钥丢失导致的大量资产流失。多签合约通常用于简单的 DAO 治理，类似与投票，需要多个私钥里的大多数，比如 4/ 7。这样还可以保证即使 3 个都丢失，剩下的私钥也可以保持 contract 继续执行。
-
-
-
-
-
-