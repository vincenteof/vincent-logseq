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
				- returns 在
	-
-
-
-
- 参考：
- https://github.com/AmazingAng/WTF-Solidity