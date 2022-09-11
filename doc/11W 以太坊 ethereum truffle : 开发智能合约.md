• [设置项目](#index1)  
• [第一份合约](#index2)  
• [编译 Solidity](#index3)  
• [使用 OpenZeppelin 合约](#index4)  
• [导入 OpenZeppelin 合约](#index5)  
• [truffle Tutorials 教程](#index98)   
• [Contact 联系方式](#index99) 

# <span id='index1'>• 设置项目</span>  
创建项目后的第一步是安装开发工具。

以太坊最流行的开发框架是Hardhat，我们用ethers.js介绍了它最常见的用途。下一个最受欢迎的是使用web3.js的Truffle。每个人都有自己的长处，舒适地使用它们是很有用的。

在这些指南中，我们将展示如何使用 Truffle 和 Hardhat 开发、测试和部署智能合约。

说明适用于 Truffle 和 Hardhat。使用此切换选择您的偏好！

要开始使用 Truffle，我们将把它安装在我们的项目目录中。
```
$ npm install --save-dev truffle
```

安装后，我们可以初始化 Truffle。这将在我们的项目目录中创建一个空的 Truffle 项目。
```
npx truffle init

Starting init...
================

> Copying project files to /home/openzeppelin/learn

Init successful, sweet!
```

# <span id='index2'>• 第一份合约</span>  
我们将 Solidity 源文件 ( .sol) 存储在一个contracts目录中。这相当于src您可能从其他语言中熟悉的目录。

我们现在可以编写我们的第一个简单的智能合约，称为Box：它将让人们存储一个可以稍后检索的值。

我们将此文件另存为contracts/Box.sol. 每个.sol文件都应该包含单个合约的代码，并以它命名。
```
// contracts/Box.sol
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Box {
    uint256 private _value;

    // Emitted when the stored value changes
    event ValueChanged(uint256 value);

    // Stores a new value in the contract
    function store(uint256 value) public {
        _value = value;
        emit ValueChanged(value);
    }

    // Reads the last stored value
    function retrieve() public view returns (uint256) {
        return _value;
    }
}
```

# <span id='index3'>• 编译 Solidity</span>  
以太坊虚拟机（EVM）不能直接执行 Solidity 代码：我们首先需要将其编译成 EVM 字节码。

我们的Box.sol合约使用 Solidity 0.8，因此我们需要首先配置 Truffle 以使用适当的 solc 版本。

我们在truffle-config.js.
```
// truffle-config.js
  ...

  // Configure your compilers
  compilers: {
    solc: {
      version: "0.8.4",    // Fetch exact version from solc-bin (default: truffle's version)
      // docker: true,        // Use "0.5.1" you've installed locally with docker (default: false)
      // settings: {          // See the solidity docs for advice about optimization and evmVersion
      //  optimizer: {
      //    enabled: false,
      //    runs: 200
      //  },
      //  evmVersion: "byzantium"
      // }
    }
  },

  ...
```

然后可以通过运行单个编译命令来实现编译：
```
npx truffle compile

Compiling your contracts...
===========================
✔ Fetching solc version list from solc-bin. Attempt #1
✔ Downloading compiler. Attempt #1.
> Compiling ./contracts/Box.sol
> Compiling ./contracts/Migrations.sol
> Artifacts written to /home/openzeppelin/learn/build/contracts
> Compiled successfully using:
   - solc: 0.8.4+commit.c7e474f2.Emscripten.clang
```

该compile命令将自动查找contracts目录中的所有合约，并使用 Solidity 编译器使用truffle-config.js.

您会注意到build/contracts创建了一个目录：它包含已编译的工件（字节码和元数据），它们是 .json 文件。将此目录添加到您的.gitignore.

# <span id='index4'>• 使用 OpenZeppelin 合约</span>  
可重用的模块和库是伟大软件的基石。OpenZeppelin Contracts包含许多有用的构建块，可用于构建智能合约。在构建它们时您可以高枕无忧：它们已经过多次审计，其安全性和正确性经过实战考验。

库中的许多合约都不是独立的，也就是说，您不需要按原样部署它们。相反，您将使用它们作为起点，通过向它们添加功能来构建您自己的合约。Solidity 提供了多重继承作为实现这一点的机制：查看Solidity 文档了解更多详细信息。

例如，Ownable合约将部署者账户标记为合约的所有者，并提供一个名为onlyOwner. 应用于函数时，onlyOwner将导致所有非来自所有者帐户的函数调用还原。还提供转让和放弃所有权的功能。

当以这种方式使用时，继承成为一种强大的机制，允许模块化，而不会强迫您部署和管理多个合约。

# <span id='index5'>• 导入 OpenZeppelin 合约</span>  
可以通过运行以下命令下载最新发布的 OpenZeppelin Contracts 库：
```
$ npm install @openzeppelin/contracts
```

您应该始终使用这些已发布版本中的库：将库源代码复制粘贴到您的项目中是一种危险的做法，它很容易在您的合约中引入安全漏洞。

要使用 OpenZeppelin 合约之一，import它通过在其路径前加上@openzeppelin/contracts. 例如，为了替换我们自己的Auth合约，我们将导入@openzeppelin/contracts/access/Ownable.sol以添加访问控制Box：
```
// contracts/Box.sol
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

// Import Ownable from the OpenZeppelin Contracts library
import "@openzeppelin/contracts/access/Ownable.sol";

// Make Box inherit from the Ownable contract
contract Box is Ownable {
    uint256 private _value;

    event ValueChanged(uint256 value);

    // The onlyOwner modifier restricts who can call the store function
    function store(uint256 value) public onlyOwner {
        _value = value;
        emit ValueChanged(value);
    }

    function retrieve() public view returns (uint256) {
        return _value;
    }
}
```

OpenZeppelin合约文档是学习开发安全智能合约系统的好地方。它具有指南和详细的 API 参考：例如，请参阅访问控制指南以了解有关Ownable上述代码示例中使用的合约的更多信息。

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
