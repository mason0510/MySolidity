## solidity1
- 1.solidity remix的使用
  compile和deploy 
- 2.测试合约编译部署
    注释方式://
```测试合约
SPDX-License-Identifier: MIT
pragma solidity ^0.8.4;
contract HelloWeb3{
string public _string = "Hello Web3!";}
````



## solidity2 变量类型
- 值类型
  1.boolean 
  ` // 布尔值
  bool public _bool = true;`
  逻辑非 逻辑与 逻辑等于 逻辑不等于 逻辑大于 逻辑小于 逻辑大于等于 逻辑小于等于 逻辑异或 逻辑与非 逻辑或非 逻辑异或非
  2.整形
  int 正副值
  uint 正数
  _number 256位正整数
  3.addres类型
  - 定义
  20字节的地址,可以显式转账。地址有两种，普通地址和可以接受eth的地址。


````
    address public _address = 0x7A58c0Be72BE218B41C608b7Fe7C5bB630736C71;
    address payable public _address1 = payable(_address); // payable address，可以转账、查余额
    uint256 public balance = _address1.balance; // balance of address
       
    address x = 0x123;
    address myAddress = this;
    if (x.balance < 10 && myAddress.balance >= 10) x.transfer(10);
````

   - 转账
   transfer 如果gas费用耗光 合同终止
   send 返回false 合同不终止
   call
     当函数只有四个字节
     callcode
     delegatecall 使用另一份合同中的代码。（早起版本不能获取委托方的msg信息）
   注意：1.call调用可以手动调整gas数量和eth币的值  
    2.所有合约都继承了addres因此可以使用this.balance
    3.你在调用合约时将控制权交给了它，你必须为可能的改变做好应对。
    4.定长字节数组

    定长 数值类型
        byte bytes8 bytes32
    不定长 引用类型
````
    // 固定长度的字节数组
    bytes32 public _byte32 = "MiniSolidity"; 
    bytes1 public _byte = _byte32[0]; 
    MiniSolidity转换成hex为
    0x4d696e69536f6c69646974790000000000000000000000000000000000000000
    _byte存储0x4d

    补充：使用byte[]浪费空间(31个字节)
````

    5.枚举
    enum 可以指定一个数值的范围，比如0-3，0-9，0-255等等。
    用户定义的数据类型 为uint分配名称

  // 用enum将uint 0， 1， 2表示为Buy, Hold, Sell
    enum ActionSet { Buy, Hold, Sell }
    // 创建enum变量 action
    ActionSet action = ActionSet.Buy;
    ````
    6.有理数和整数字面敞亮
    在早期版本中，整数字面常数的除法也会截断，但在现在的版本中，会将结果转换成一个有理数。即 5 / 2 并不等于 2，而是等于 2.5。
    uint128 a = 1;
    uint128 b = 2.5 + a + 0.5; 
    这种会报错。 2.5+a 不见长类型
    
    7.字符串字面敞亮
    \xNN 表示一个 16 进制值
    \uNNNN 表示一个 Unicode 值 最终转成u8
    - 16禁止


- 函数类型
  数类型有两类：
- 内部（internal） 函数
      只能在当前上下文调用
      外部（external） 函数
  function (<parameter types>) {internal|external} [pure|constant|view|payable] [returns (<return types>)]
如果在函数以外使用 函数名编译成24位bytes24
     public 函数不限制调用，比较特殊可以返回selector返回abi
    目前不支持lambda表达式
````
//调用外部库进行计算
 contract Pyramid {
  using ArrayUtils for *;
  function pyramid(uint l) public pure returns (uint) {
    return ArrayUtils.range(l).map(square).reduce(sum);
  }
  function square(uint x) internal pure returns (uint) {
    return x * x;
  }
  function sum(uint x, uint y) internal pure returns (uint) {
    return x + y;
  }
}

查询 oracle
contract OracleUser {
  Oracle constant oracle = Oracle(0x1234567); // 已知的合约
  function buySomething() {
    oracle.query("USD", this.oracleResponse);
  }
  function oracleResponse(bytes response) public {
    require(msg.sender == address(oracle));
    // 使用数据
  }
}


````
  
-引用类型
    必须考虑是存储在内存还是storage,函数参数默认内存中。
    calldata 存储函数参数
````
pragma solidity ^0.4.0;

contract C {
    uint[] x; // x 的数据存储位置是 storage

    // memoryArray 的数据存储位置是 memory
    function f(uint[] memoryArray) public {
        x = memoryArray; // 将整个数组拷贝到 storage 中，可行
        var y = x;  // 分配一个指针（其中 y 的数据存储位置是 storage），可行
        y[7]; // 返回第 8 个元素，可行
        y.length = 2; // 通过 y 修改 x，可行
        delete x; // 清除数组，同时修改 y，可行
        // 下面的就不可行了；需要在 storage 中创建新的未命名的临时数组， /
        // 但 storage 是“静态”分配的：
        // y = memoryArray;
        // 下面这一行也不可行，因为这会“重置”指针，
        // 但并没有可以让它指向的合适的存储位置。
        // delete y;

        g(x); // 调用 g 函数，同时移交对 x 的引用
        h(x); // 调用 h 函数，同时在 memory 中创建一个独立的临时拷贝
    }

    function g(uint[] storage storageArray) internal {}
    function h(uint[] memoryArray) public {}
}
````
外部函数的参数（不包括返回参数）： calldata
状态变量： storage
内部函数的参数（包括返回参数）： memory
所有其它局部变量： storage
### 数组
    bytes 和 string 类型的变量是特殊的数组。
    如果想要访问以字节表示的字符串 s，请使用 bytes(s).length / bytes(s)[7] = 'x';

创建可变数组,例如
` uint[] memory a = new uint[](7);`
错误用法：抵偿不能赋值给变长
`uint[] x = [uint(1), 3, 4];`
### 成员
动态数组 ---> length
### push
变长的 存储storage 数组以及 bytes 类型 有一个push方法
｜在外部函数中目前还不能使用多维数组，并且不能通过合约返回动态内容。但是可使用大型的静态数组。
###结构体
Solidity 支持通过构造结构体的形式定义新的类型。
campaigns[campaignID].amount = 0//直接访问局部变量

- 映射类型
  _KeyType 可以是除了映射、变长数组、合约、枚举以及结构体以外的几乎所有类型
  _ValueType 可以是包括映射类型在内的任何类型。
  在映射中，实际上并不存储 key，而是存储它的 keccak256 哈希值，从而便于查询实际的值。
  
###涉及到LValues的值
delete a 无法删除映射，可以用于数组和变量 结构体

##值转换
只要值类型之间的转换在语义上行得通，而且转换的过程中没有信息丢，都可以转换。
###显式转换
如果某些情况下编译器不支持隐式转换
int8 y = -3;
uint x = uint(y);

uint32 a = 0x12345678;
uint16 b = uint16(a); 高位被舍弃
###类型推断
uint24 x = 0x123;
var y = x;





##安全
如果编译器警告了你什么事，你最好修改一下，即使你不认为这个特定的警告不会产生安全隐患，因为那也有可能埋藏着其他的问题。 我们给出的任何编译器警告，都可以通过轻微的修改来去掉。

将实现的函数文档化，这样别人看到代码的时候就可以理解你的意图，并判断代码是否按照正确的意图实现。

1.函数会首先做一些检查工作
2.会影响当前合约状态变量的那些处理
3.对已知合约的调用反过来也可能导致对未知合约的调用

增加检查函数。

如果自检查没有通过，合约就会自动切换到某种“故障安全”模式， 例如，关闭大部分功能，将控制权交给某个固定的可信第三方，或者将合约转换成一个简单的“退回我的钱”合约。


##错误类型
JSONError: JSON输入不符合所需格式，例如，输入不是JSON对象，不支持的语言等。
IOError: IO和导入处理错误，例如，在提供的源里包含无法解析的URL或哈希值不匹配。
ParserError: 源代码不符合语言规则。
DocstringParsingError: 注释块中的NatSpec标签无法解析。
SyntaxError: 语法错误，例如 continue 在 for 循环外部使用。
DeclarationError: 无效的，无法解析的或冲突的标识符名称 比如 Identifier not found。
TypeError: 类型系统内的错误，例如无效类型转换，无效赋值等。
UnimplementedFeatureError: 编译器当前不支持该功能，但预计将在未来的版本中支持。
InternalCompilerError: 在编译器中触发的内部错误——应将此报告为一个issue。
Exception: 编译期间的未知失败——应将此报告为一个issue。
CompilerError: 编译器堆栈的无效使用——应将此报告为一个issue。
FatalError: 未正确处理致命错误——应将此报告为一个issue。
Warning: 警告，不会停止编译，但应尽可能处理。

##函数分组
构造函数
fallback 函数（如果存在）
外部函数
公共函数
内部函数和变量
私有函数和变量

##语义规范
函数的可见性修饰符应该出现在任何自定义修饰符之前。
修饰符下沉
操作符应有空格
当在驼峰式命名中使用缩写时，应该将缩写中的所有字母都大写。 因此 HTTPServerError 比 HttpServerError 好。
枚举类型都大写
与保留关键字冲突保留下户线


原文链接
https://solidity-cn.readthedocs.io/zh/develop/types.html