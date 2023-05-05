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
			- global variable：预留关键字，可以在函数内不申明直接用，比如：`msg.sender`, ` block.number` 和 `msg.data ` [链接](https://learnblockchain.cn/docs/solidity/units-and-global-variables.html#special-variables-and-functions)
		- solidity 中的 array
			- array 分为固定长度与可变长度两种：
				- 固定长度数组：在声明时指定数组的长度，如 `uint8[8] array1`
				- 不定长数组：声明时不指定长度，`uint[] array4`
				- `bytes` 比较特殊，是数组但是不需要 `[]`，也不能用 `byte[]` 来声明字节数组。可用 `bytes` 或者 `byte1[]`，在 gas 上 `bytes` 比`byte1[]` 数组便宜。
			- 创建数组的规则：
				- 对于用 `memory` 创建的动态数组，可以用 new 操作符来创建，但是必须声明长度。
					- ```javascript
					  bytes memory array9 = new bytes(9)
					  ```
				- 也可以用字面量创建数组，如果默认的 type 是最小单位，也可以通过声明字一个变量的类型来手动指定
					- ```javascript
					  [uint(1), 2, 3]
					  ```
				- 创建的动态数组需要一个一个赋值
					- ```javascript
					  uint[] memory x = new uint[](3);
					  x[0] = 1;
					  x[1] = 3;
					  x[2] = 4;
					  ```
					- 数组的属性方法：`length`，`push()`，`push(x)`，`pop()`
		- 结构体：
			- 定义
				- ```rust
				  struct Student {
				  	uint256 id;
				  	uint256 score;
				  }
				  
				  Student student;
				  ```
			- 给结构体赋值
				- ```javascript
				  function initStudent1() external {
				  	// 生成引用
				  	Student storage _student = student;
				  	_student.id = 11;
				  	_student.score = 100;
				  }
				  ```
			- 或者直接对合约状态变量赋值
				- ```javascript
				  function initStudent2() external {
				  	student.id = 1;
				  	student.score = 80;
				  }
				  ```
		- mapping 映射关系：
			- 声明：
				- `mapping(uint => address) public idToAddress;`
				  id:: 64525870-d893-49df-a547-a242d30b46d6
				- `mapping(address => address) public swapPair;`
			- 规则：
				- mapping 的 `_KeyType` 只能选择 solidity 的默认类型，比如 `uint` 和 `address` 等。而 `_ValueType` 可以使用自定义类型。
				- mapping 的存储位置必须是 `storage`，因此可用于合约的状态变量，函数中的 `storage` 变量，和 library 的参数 （[例子](https://github.com/ethereum/solidity/issues/4635)）。不能用于 public 函数的参数和返回结果。
				- 如果 mapping 声明为 `public`，那么 solidity 会自动给你创建一个 getter 函数，可以通过 key 来查询对应的 value。
				- 给 mapping 新增一个键值对
					- ```javascript
					  function writeMap (uint _Key, address _Value) public{
					  	idToAddress[_Key] = _Value;
					  }
					  ```
			- 原理：
				- mapping 不存储任何 key 的资讯，也没有 length 资讯
				- mapping 使用 `keccak256(key)` 当成 offset 存取 value
				- 以太坊会定义未使用的空间为 0。所以未赋值的 kv 是各个 type 的默认值
		- 常量：
			- solidity 中常量分为 `constant` 和 `immutable`
			- 可以用常量来节省 gas
			- `constant` 在声明的时候就必须给初始值，而 `immutable` 可以先声明后赋值
			- 只有数值类型能声明成 `constant` 或者 `immutable`；`string` 和 `bytes` 可以为 `constant` 但不能为 `immutable`。
		- 控制流：
			- 几乎与 JS 相同
		- constructor 与 modifier：
		  collapsed:: true
			- `constructor` 是一种特殊的函数，每个合约可以定义一个，在部署合约的时候自动执行一次。可以用它来初始化合约的一些参数。
				- ```javascript
				  address owner; // 定义owner变量
				  
				  // 构造函数
				  constructor() public {
				    owner = msg.sender; // 在部署合约的时候，将owner设置为部署者的地址
				  }
				  ```
			- `modifier` 类似于其他语言里的 `decorator`。主要运用场景是允许函数前的检查，例如地址，变量，余额等。
				- ```javascript
				  // 定义modifier
				  modifier onlyOwner {
				    require(msg.sender == owner); // 检查调用者是否为owner地址
				    _; // 如果是的话，继续运行函数主体；否则报错并revert交易
				  }
				  ```
				- 代有`onlyOwner`修饰符的函数只能被`owner`地址调用，比如下面这个例子：
				- ```javascript
				  function changeOwner(address _newOwner) external onlyOwner{
				        owner = _newOwner; // 只有owner地址运行这个函数，并改变owner
				  }
				  ```
				- oppenzeppelin 的标准实现
					- https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/access/Ownable.sol
			-
		- 事件：
			- Solidity 中的 event 是 EVM 上日志的抽象，它有两个特点：
				- 应用程序（ether.js）可以通过 RPC 接口
			-
-
- 参考：
- https://github.com/AmazingAng/WTF-Solidity