• [介绍](#index1)  
• [关于测试](#index2)  
• [设置测试环境](#index3)  
• [编写单元测试](#index4)  
• [执行复杂的断言](#index5)  
• [truffle Tutorials 教程](#index98)   
• [Contact 联系方式](#index99) 

# <span id='index1'>• 介绍</span>  
在区块链环境中，一个错误可能会花费您所有的资金——甚至更糟的是，您的用户的资金！本指南将通过编写自动化测试来验证您的应用程序的行为是否完全符合您的预期，从而帮助您开发强大的应用程序。

# <span id='index2'>• 关于测试</span>  
有各种各样的测试技术，从简单的手动验证到复杂的端到端设置，所有这些技术都以自己的方式有用。

然而，在智能合约开发方面，实践表明合约单元测试非常值得。这些测试编写简单，运行迅速，让您可以放心地添加功能并修复代码中的错误。

智能合约单元测试由多个小型的、有针对性的测试组成，每个测试都检查合约的一小部分是否正确。它们通常可以用构成规范的单个句子来表达，例如“管理员能够暂停合同”、“转移代币会发出事件”或“非管理员不能铸造新代币”。

# <span id='index3'>• 设置测试环境</span>  
您可能想知道我们将如何运行这些测试，因为智能合约是在区块链中执行的。使用实际的以太坊网络会非常昂贵，虽然测试网是免费的，但它们也很慢（阻塞时间在 5 到 20 秒之间）。如果我们打算在对代码进行更改时运行数百个测试，我们需要更好的东西。

我们将使用的称为本地区块链：真实事物的精简版，与互联网断开连接，在您的机器上运行。这将大大简化事情：你不需要获得以太币，新块将立即被挖掘。

# <span id='index4'>• 编写单元测试</span>  
我们将使用Chai断言进行单元测试。
```
$ npm install --save-dev chai
```
我们将把我们的测试文件保存在一个test目录中。最好通过镜像contracts目录来构建测试：为那里的每个.sol文件创建一个相应的测试文件。

是时候编写我们的第一个测试了！这些将测试之前指南Box中合同的属性：一个简单的合同，让您获得之前所有者的价值d。retrievestore

我们将测试保存为test/Box.test.js. 每个.test.js文件都应该有一个合同的测试，并以它命名。
```
// test/Box.test.js
// Load dependencies
const { expect } = require('chai');

// Load compiled artifacts
const Box = artifacts.require('Box');

// Start test block
contract('Box', function () {
  beforeEach(async function () {
    // Deploy a new Box contract for each test
    this.box = await Box.new();
  });

  // Test case
  it('retrieve returns a value previously stored', async function () {
    // Store a value
    await this.box.store(42);

    // Test if the returned value is the same one
    // Note that we need to use strings to compare the 256 bit integers
    expect((await this.box.retrieve()).toString()).to.equal('42');
  });
});
```

已经写了很多关于如何构建单元测试的书籍。查看Moloch 测试指南，了解为测试 Solidity 智能合约而设计的一组原则。
我们现在准备好运行我们的测试了！

运行npx truffle test将执行test目录中的所有测试，检查您的合约是否按照您的预期工作。
```
npx truffle test
Using network 'development'.


Compiling your contracts...
===========================
> Everything is up to date, there is nothing to compile.



  Contract: Box
    ✓ retrieve returns a value previously stored (38ms)


  1 passing (117ms)
```

此时设置一个持续集成服务（例如CircleCI）来让您的测试在每次将代码提交到 GitHub 时自动运行也是一个非常好的主意。

# <span id='index5'>• 执行复杂的断言</span>  
您的合约的许多有趣属性可能难以捕捉，例如：
验证合同是否因错误而恢复
衡量一个账户的以太币余额变化了多少
检查是否发出了正确的事件

OpenZeppelin Test Helpers是一个旨在帮助您测试所有这些属性的库。它还将简化模拟在区块链上传递的时间和处理非常大的数字的任务。

OpenZeppelin Test Helpers 是基于 web3.js 的，因此 Hardhat 用户应该使用 Truffle 插件以获得兼容性，或者使用Waffle和 ethers.js，它提供了类似的功能。

要安装 OpenZeppelin 测试助手，请运行：
```
$ npm install --save-dev @openzeppelin/test-helpers
```
然后我们可以更新我们的测试以使用 OpenZeppelin 测试助手来支持非常大的数量，检查正在发出的事件并检查事务是否恢复。
```
// test/Box.test.js
// Load dependencies
const { expect } = require('chai');

// Import utilities from Test Helpers
const { BN, expectEvent, expectRevert } = require('@openzeppelin/test-helpers');

// Load compiled artifacts
const Box = artifacts.require('Box');

// Start test block
contract('Box', function ([ owner, other ]) {
  // Use large integers ('big numbers')
  const value = new BN('42');

  beforeEach(async function () {
    this.box = await Box.new({ from: owner });
  });

  it('retrieve returns a value previously stored', async function () {
    await this.box.store(value, { from: owner });

    // Use large integer comparisons
    expect(await this.box.retrieve()).to.be.bignumber.equal(value);
  });

  it('store emits an event', async function () {
    const receipt = await this.box.store(value, { from: owner });

    // Test that a ValueChanged event was emitted with the new value
    expectEvent(receipt, 'ValueChanged', { value: value });
  });

  it('non owner cannot store a value', async function () {
    // Test a transaction reverts
    await expectRevert(
      this.box.store(value, { from: other }),
      'Ownable: caller is not the owner',
    );
  });
});
```

这些将测试之前指南Ownable Box中合同的属性：一个简单的合同，让您获得之前所有者的价值d。retrievestore

再次运行测试以查看测试助手的运行情况：
```
npx truffle test
...
  Contract: Box
    ✓ retrieve returns a value previously stored
    ✓ store emits an event
    ✓ non owner cannot store a value (588ms)


  3 passing (753ms)
```

测试助手将让您编写强大的断言，而不必担心底层以太坊库的低级细节。要了解有关您可以使用它们做什么的更多信息，请访问他们的API 参考。

# <span id='index98'>• truffle Tutorials 教程</span>  
CN 中文 Github  [truffle 教程 : github.com/565ee/truffle_CN](https://github.com/565ee/truffle_CN)  
CN 中文 CSDN    [truffle 教程 : blog.csdn.net/wx468116118](https://blog.csdn.net/wx468116118/category_12007659.html)  
EN 英文 Github  [truffle Tutorials : github.com/565ee/truffle_EN](https://github.com/565ee/truffle_EN)  

# <span id='index99'>• Contact 联系方式</span>  
Homepage : [565.ee](https://565.ee)  
微信公众号 : wx468116118  
微信 QQ   : 468116118  
GitHub   : [github.com/565ee](https://github.com/565ee)   
CSDN     : [blog.csdn.net/wx468116118](https://blog.csdn.net/wx468116118)  
Email    : 468116118@qq.com
