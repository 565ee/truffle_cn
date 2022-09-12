• [建立本地区块链](#index1)  
• [部署智能合约](#index2)  
• [发送交易](#index3)  
• [查询状态](#index4)  
• [设置](#index5)  
• [获取合约实例](#index6)  
• [调用合约](#index7)  
• [发送交易](#index8)  
• [truffle Tutorials 教程](#index98)   
• [Contact 联系方式](#index99) 

# <span id='index1'>• 建立本地区块链</span>  
在开始之前，我们首先需要一个可以部署合约的环境。以太坊区块链（通常称为“主网”，表示“主网络”）需要花费真金白银才能使用它，以以太币（其本币）的形式。在尝试新想法或工具时，这使其成为一个糟糕的选择。

为了解决这个问题，存在许多“测试网络”（用于“测试网络”）：其中包括 Ropsten、Rinkeby、Kovan 和 Goerli 区块链。它们的工作方式与主网非常相似，但有一个区别：您可以免费获得这些网络的以太币，因此使用它们不会花费您一分钱。但是，您仍然需要处理私钥管理、5 到 20 秒范围内的阻塞时间，以及实际获得这个免费的 Ether。

在开发过程中，最好使用本地区块链。它在您的机器上运行，不需要互联网访问，为您提供所需的所有以太币，并立即挖掘区块。这些原因也使得本地区块链非常适合自动化测试。

最受欢迎的本地区块链是Ganache。要将其安装在您的项目中，请运行：
```
$ npm install --save-dev ganache-cli
```

启动时，Ganache 将创建一组随机的未锁定帐户并给他们以太币。为了获得将在本指南中使用的相同地址，您可以在确定性模式下启动 Ganache：
```
$ npx ganache-cli --deterministic
```

Ganache 将打印出可用帐户及其私钥的列表，以及一些区块链配置值。最重要的是，它将显示它的地址，我们将使用它来连接它。默认情况下，这将是127.0.0.1:8545.

请记住，每次运行 Ganache 时，它​​都会创建一个全新的本地区块链 -不会保留之前运行的状态。这对于短期实验来说很好，但这意味着您需要在这些指南期间打开一个运行 Ganache 的窗口。或者，您可以使用该选项运行 Ganache --db，提供一个目录来在两次运行之间存储其数据。

# <span id='index2'>• 部署智能合约</span>  
在开发智能合约指南中，我们设置了我们的开发环境。

如果您还没有此设置，请创建并设置项目，然后创建并编译我们的 Box 智能合约。

随着我们的项目设置完成，我们现在可以部署合约了。我们将从Box开发智能合约指南中进行部署。确保您有Box in的副本contracts/Box.sol。

Truffle 使用迁移来部署合约。迁移由 JavaScript 文件和一个特殊的迁移合约组成，用于跟踪链上的迁移。

我们将创建一个 JavaScript 迁移来部署我们的 Box 合约。我们将此文件另存为migrations/2_deploy.js.
```
// migrations/2_deploy.js
const Box = artifacts.require('Box');

module.exports = async function (deployer) {
  await deployer.deploy(Box);
};
```

在我们部署之前，我们需要配置到 ganache 的连接。我们需要为 localhost 和端口 8545 添加一个开发网络，这是我们本地区块链正在使用的。
```
// truffle-config.js
module.exports = {
...
  networks: {
...
    development: {
     host: "127.0.0.1",     // Localhost (default: none)
     port: 8545,            // Standard Ethereum port (default: none)
     network_id: "*",       // Any network (default: none)
    },
    ...
```

使用该migrate命令，我们可以将Box合约部署到development网络（Ganache）：
```
npx truffle migrate --network development

Compiling your contracts...
===========================
> Everything is up to date, there is nothing to compile.



Starting migrations...
======================
> Network name:    'development'
> Network id:      1619762548805
> Block gas limit: 6721975 (0x6691b7)


1_initial_migration.js
======================

   Deploying 'Migrations'
   ----------------------
...

2_deploy.js
===========

   Deploying 'Box'
   ---------------
   > transaction hash:    0x25b0a326bfc9aa64be13efb5a4fb3f784ffa845c36d049547eeb0f78e0a3108d
   > Blocks: 0            Seconds: 0
   > contract address:    0xCfEB869F69431e42cdB54A4F4f105C19C080A601
...
```

Truffle 将跟踪您部署的合约，但它也会在部署时显示它们的地址（在我们的示例中，0xCfEB869F69431e42cdB54A4F4f105C19C080A601）。当以编程方式与它们交互时，这些值将很有用。
全部完成！在真实的网络上，这个过程需要几秒钟，但在本地区块链上几乎是即时的。

请记住，本地区块链不会在多次运行中保持其状态！如果您关闭本地区块链流程，您将不得不重新部署您的合约。

# <span id='index3'>• 发送交易</span>  
Box的第一个函数store接收一个整数值并将其存储在合约存储中。因为这个函数修改了区块链状态，所以我们需要向合约发送一个交易来执行它。

我们将发送一个事务来调用store具有数值的函数：
```
truffle(development)> await box.store(42)
{ tx:
   '0x5d4cc78f5d5eac3650214740728192ac760978e261962736289b10da0ec0ea43',
...
       event: 'ValueChanged',
       args: [Result] } ] }
```

请注意交易收据如何也显示Box发出了ValueChanged事件。

# <span id='index4'>• 查询状态</span>  
Box的另一个函数被调用retrieve，它返回存储在合约中的整数值。这是区块链状态的查询，所以我们不需要发送交易：
```
truffle(development)> await box.retrieve()
<BN: 2a>
```
因为查询只读取状态而不发送交易，所以没有交易哈希报告。这也意味着使用查询不需要任何 Ether，并且可以在任何网络上免费使用。

我们的Box合约返回uint256的数字对于 JavaScript 来说太大了，因此我们返回了一个大数字对象。我们可以使用 将大数字显示为字符串(await box.retrieve()).toString()。
```
truffle(development)> (await box.retrieve()).toString()
'42'
```

# <span id='index5'>• 设置</span>  
让我们开始在一个新scripts/index.js文件中编码，我们将在其中编写 JavaScript 代码，从一些样板开始，包括编写异步代码。
```
// scripts/index.js
module.exports = async function main (callback) {
  try {
    // Our code will go here

    callback(0);
  } catch (error) {
    console.error(error);
    callback(1);
  }
};
```

我们可以通过询问本地节点来测试我们的设置，例如启用的帐户列表：
```
// Retrieve accounts from the local node
const accounts = await web3.eth.getAccounts();
console.log(accounts)
```

我们不会在每个片段上重复样板代码，但请确保始终在我们上面定义 的函数内进行编码！main
使用 运行上面的代码truffle exec，并检查您是否获得了可用帐户列表作为响应。
```
npx truffle exec --network development ./scripts/index.js
Using network 'development'.

[ '0x90F8bf6A479f320ead074411a4B0e7944Ea8c9C1',
  '0xFFcf8FDEE72ac11b5c542428B35EEF5769C409f0',
...
]
```

这些帐户应与您之前启动本地区块链时显示的帐户相匹配。现在我们有了第一个从区块链中获取数据的代码片段，让我们开始使用我们的合约。请记住，我们在上面定义的main函数中添加了代码。

# <span id='index6'>• 获取合约实例</span>  
为了与Box我们部署的合约进行交互，我们将使用 Truffle 合约抽象，这是一个 JavaScript 对象，代表我们在区块链上的合约。
```
// Set up a Truffle contract, representing our deployed Box instance
const Box = artifacts.require('Box');
const box = await Box.deployed();
```
我们现在可以使用这个 JavaScript 对象与我们的合约进行交互。

# <span id='index7'>• 调用合约</span>  
让我们从显示Box合约的当前价值开始。

我们需要调用retrieve()合约的公共方法，并等待响应：
```
// Call the retrieve() function of the deployed Box contract
const value = await box.retrieve();
console.log('Box value is', value.toString());
```
这个片段相当于我们之前从控制台运行的查询。现在，通过再次运行脚本并检查打印值来确保一切运行顺利：
```
$ npx truffle exec --network development ./scripts/index.js
Using network 'development'.

Box value is 42
```

# <span id='index8'>• 发送交易</span>  
现在，我们将向storeBox 中的新值发送交易。

让我们在 中存储一个值，然后使用23我们Box之前编写的代码来显示更新后的值：
```
// Send a transaction to store() a new value in the Box
await box.store(23);

// Call the retrieve() function of the deployed Box contract
const value = await box.retrieve();
console.log('Box value is', value.toString());
```

在现实世界的应用程序中，您可能想要估计交易的 gas，并检查gas 价格预言机以了解每笔交易使用的最佳值。
我们现在可以运行代码片段，并检查框的值是否已更新！
```
$ npx truffle exec --network development ./scripts/index.js
Using network 'development'.

Box value is 23
```

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
