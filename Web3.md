# Web3

## Prologue

[Web3学习路线图]: https://learnblockchain.cn/maps/Roadmap

[TOC]

> 中间件

- 

## Block Chain Basic Knowledge

- 什么是区块链	**通过hash 串联的区块结构	区块内的交易通过 Merkel 树保存	⼀个由多个节点组成的⽹络**

![区块链示例图](https://htps-public.oss-cn-nanjing.aliyuncs.com/typora/web3/%E5%8C%BA%E5%9D%97%E9%93%BE%E7%A4%BA%E4%BE%8B%E5%9B%BE.jpg)

- 以太坊 账户

  - EOA

    - Externally Owned Account
    - 外部控制账户
    - 该类账户由公钥-私钥对控制（由人控制）

  - CA

    - Contract account
    - 合约控制账户
    - 该类账户由存储在账户中的代码控制

  - 外部账户（EOA）与 合约账户在 EVM 层⾯是等效的，皆由四个基本组成部分组成：

    - **nonce**

      - 如果账户是一个外部用户账户，nonce代表从此账户地址发送的交易序号。如果账户是一个合约账户，nonce代表此账户创建的合约序号。

      - > 提示：以太坊中有两种nonce ， 一种是账号nonce——表示一个账号的交易数量；一种是工作量证明nonce——一个用于计算满足工作量证明的随机数。

    - **balance**

      - 此地址拥有的以太币余额数量。单位是`Wei`，`1 ether`=`10^18 wei`。当向地址发送带有以太币的交易时，balance会随之改变。外部用户账户和合约账户都可以有余额；合约账户使用代码管理所拥有的资金，外部用户账户则是使用私钥签名来花费资金；合约账户存储了代码，外部用户账户则没有。

    - **storageRoot**

      - Merkle Patricia树的根节点哈希值。Merkle树会将此账户存储内容的哈希值进行编码，默认是空值。

    - **codeHash**

      - 此账户代码的哈希值。对于合约账户，它是合约代码被哈希计算之后的结果作为codeHash保存。对于外部用户账户，codeHash 是一个空字符串的哈希值。

  - 但是在表现上有不⼀样：

    - **交易**只能从外部账号发出，合约只能被动相应执⾏，合约之间的交互通常称为**消息**，所有的⼿续费 gas 只能由外部账号⽀付

- 以太坊中只包含3种交易

  - 普通交易

    - ```
      {
      	to: ‘0x687422…’,
      	value: 0.0005,
      	data: "0x" // 也可以附加消息
      }
      ```

  - 创建合约

    - ```
      {
      	to: ‘’,
      	value: 0.0,
      	data: "0x606060405234156100057xl06…………"
      }
      ```

  - 调用合约

    - ```
      {
      	to: ‘0x687422eEA2cB73..’, //合约地址 
          value: 0.0,
      	data: "0x06661abd"
      }
      ```

      



![区块链客户端](https://htps-public.oss-cn-nanjing.aliyuncs.com/typora/web3/%E5%8C%BA%E5%9D%97%E9%93%BE%E5%AE%A2%E6%88%B7%E7%AB%AF%E5%85%B3%E7%B3%BB.png)

> 钱包	账号管理⼯具，进⾏签名发起交易（管理助记词、私钥）
>
> - Metamask
> - Imtoken
> - TrustWallet



> Gas	
>
> - EVM 的计价规则，也防⽌图灵死机问题。
>
> - GAS 是⼀个⼯作量单位，复杂度越⼤，所需 gas 越多。
>
> - ⼿续费⽤= gas 数量(gas limit) * gas price单价（以太币计价gwei）
>
> Gas机制
>
> - gas（工作量单位）	:	汽油（升）
> - gas Limit	:	提供多少升汽油（⽤不完可退）
> - gas价格（gwei）	:	每升汽油的价格（元）
> - ⼿续费：gas ⽤量 * gas价格	:	运费：汽油⽤量 * 汽油价格
> - 费⽤由发起交易的账号⽀付	:	谁提任务谁付运费



> 以太单位
>
> - 最⼩单位: Wei (伟)
> - 10^9 Wei = 1 Gwei
> - 10^12 Wei = 1 szabo (萨博)
> - 10^15 Wei = 1 finey (芬尼)
> - 10^18 Wei = 1 Ether



> 网络	https://chainlist.org/





## Develop Framework

> IDE
>
> - Remix
> - Vscode + Solidity<by Nomic Foundation>

> Framework
>
> - Truffle
>
>   - 编译、部署、测试合约的⼀整套开发⼯具
>   - ⽂档：https://trufflesuite.com/docs/truffle/
>   - 中⽂⽂档：https://learnblockchain.cn/docs/truffle/
>
> - Ganache
>
>   - Ganache 是一个 **以太坊开发工具**，用于本地搭建一个 **私人以太坊区块链**，方便开发、测试和调试智能合约，而无需连接到真正的以太坊主网或测试网。
>   - Ganache 主要特点
>     1. **本地以太坊区块链**：
>        - 提供快速的区块生成，不需要等待区块确认。
>        - 允许重置链，方便重复测试。
>     2. **内置测试账户**：
>        - 默认生成 10 个预充值的测试账户，方便开发者进行交易和合约部署。
>     3. **交易和合约调试**：
>        - 记录所有交易和合约执行情况，包括 `gas` 消耗、事件日志等。
>     4. **两种版本**：
>        - **Ganache UI**（图形界面）
>        - **Ganache CLI**（命令行工具，适用于自动化测试）
>
> - Hardhat
>
>   - 描述
>
>     - 编译、部署、测试和调试以太坊应⽤的开发环境，围绕 task(任务)和plugins(插件)概念设计。
>     - 在命令⾏运⾏Hardhat时，都是在运⾏任务，例如： npx hardhat compile正在运⾏compile任务。
>
>   - Hardhat node：开发区块链，提供本地模拟的链上环境
>
>   - 官⽅⽂档：https://hardhat.org/getting-started/
>
>   - 中⽂⽂档：https://learnblockchain.cn/docs/hardhat/getting-started/
>
>   - Hardhat ⼯程包含
>
>     - contracts：智能合约⽬录
>     - ignotion：部署脚本⽂件
>     - test：智能合约测试⽤例⽂件夹。
>     - hardhat.config.js：配置⽂件，配置hardhat连接的⽹络及编译选项。
>
>   - Hardhat - 部署 - 配置⽹络
>
>     1. 配置⽹络
>
>        - ```js
>          module.exports = {
>              networks: {
>          		development: {
>          			url: “http://127.0.0.1:8545”,  chainId: 31337
>                  }
>              }
>          };
>          ```
>
>          
>
>     2. 编写部署脚本
>                
>        - script/xxx_deploy.js
>                
>        - ```js
>          async function main(){
>              //await hre.run('compile');
>              const Counter = await ether.getContractFactory("Counter");
>              const counter = await Counter.deploy();
>              await counter.deployed();
>              console.log("Counter deployed to:", counter.address);
>          }
>          main();
>          ```
>                
>     3. 启动⽹络（本地）
>                
>        - ```
>          npx hardhat node
>          ```
>                
>     4. 执⾏部署
>                
>        - ```
>          npx hardhat run script/xxx_deploy.js [ —network ⽹络]
>          ```
>
>   - Hardhat test
>
>     - ```
>       npx hardhat test
>       Greeter
>       Deploying a Greeter with greeting: Hello,world! Changing greeting from 'Hello, world!' to 'Hola,mundo!'
>       Should return the new greeting once it's changed  (694ms)
>       1 passing(697ms)
>       ```
>
>       
>
> - Foundry



