- solidity 里的类型：
	- bool
	- 整型
		- 和很多底层语言一样，区分长度，比如 int，uint，uint256
	- address
		- 以太坊地址
		- 20 个字节
		- 有普通地址和 payable 地址
		- 有成员变量
	- 定长字节数组 bytes
		- byte，bytes8，bytes32
	- enum
		- 可以 uint 转换
	- 函数
		- ```javascript
		  function <function name> (<parameter types>) {internal|external|public|private} [pure|view|payable] [returns (<return types>)]
		  ```
		- {} 表示可见性
		- [] 表示功能权限
		  collapsed:: true
			- `pure` 和 `view` 与 gas fee 有关
				- 因为合约的状态存储在链上，如果不改链上状态就不用付 gas fee，包含这两个关键字的函数是不该写链上状态的，因此用户调用不用付 gas fee。
					- 在以太坊中，以下语句被视为修改链上状态：
						- https://www.wtf.academy/solidity-start/Function/#%E5%88%B0%E5%BA%95%E4%BB%80%E4%B9%88%E6%98%AFpure%E5%92%8Cview
				- 他们的区别
					- pure 只能是一些申明入参和出参的纯函数，它连读区合约状态都不行
						- ```javascript
						  function addPure(uint256 _number) external pure returns(uint256 new_number){
						    new_number = _number+1;
						  }
						  ```
					- view 可以读取合约状态但不能写
						- ```javascript
						  function addView() external view returns(uint256 new_number) {
						    new_number = number + 1;
						  }
						  ```
			- `internal` vs `external`
				- internal 函数外部无法抵用，部署之后 remix 上不会展示出来
					- ```javascript
					  function minus() internal {
					    number = number - 1;
					  }
					  ```
				- external 可以调 internal，这样 internal也暴露到了外部
				  collapsed:: true
					- ```javascript
					  function minusCall() external {
					    minus();
					  }
					  ```
			- `payable` 表示该函数可以给合约转入 ETH
				- ```javascript
				  function minusPayable() external payable returns(uint256 balance) {
				    minus();
				    balance = address(this).balance;
				  }
				  ```
				- this 引用合约本身
				- 在 remix 里可以在调用的时候设置支付金额
			-
		- 输出
			- `return` 与 `returns`
				- returns 在函数声明上，用于声明返回值类型及名称
				- return 在函数主题内，接着返回的变量
				- ```javascript
				  function returnMultiple() public pure returns(uint256, bool, uint256[3] memory){
				    return(1, true, [uint256(1),2,5]);
				  }
				  ```
			- 使用 returns 声明的变量名，只需要把对应的结果赋给它们，不需要手动 return
			- solidity 支持解构
				- ```javascript
				  	uint256 _number;
				      bool _bool;
				      uint256[3] memory _array;
				      (_number, _bool, _array) = returnNamed();
				  ```
		- 引用类型：
			- 有以下几种：
				- 数组
				- 结构体
				- 映射
			- 因为占用空间大，所以直接传地址。这些类型的变量需要在使用时申明数据的存储位置。
			- 位置分为三类 `storage`，`memory`，`calldata`。主要区别是 gas 成本不同，`storage` 存链上，消耗 gas 最多，`memory` 与 `calldata` 在临时内存里，gas 消耗低。
				- 合约状态默认都是 `storage`，在链上
				- 函数参数与局部变量一般用 `memory` 不上链
				- `calldata` 与 `memory` 类似，区别是 calldata 是 immutable 的，一般用于函数参数
			- 不同位置的变量互相赋值存在一些规则，决定是直接传递 reference 还是产生副本：
				- 合约 storage 变量赋予函数内 storage 变量，会创建引用
				- storage 给 memory，会创建副本
				- memory 传 memory 是引用
				- 其余 case 都是副本
		- solidity 中变量的作用域有三种：
			- state variable，local variable，global variable
			- state variable：合约的链上状态，所有合约都可访问，gas 消耗高，合约内函数外声明
			- local variable：很普通，就是函数的栈上变量
			- global variable：预留关键字，可以在函数内不申明直接用，比如：`msg.sender`, block.number和msg.data  [链接](https://learnblockchain.cn/docs/solidity/units-and-global-variables.html#special-variables-and-functions)
-
- 参考：
- https://github.com/AmazingAng/WTF-Solidity