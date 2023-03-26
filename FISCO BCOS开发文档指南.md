# FISCO BCOS开发文档指南

## Linux常用命令

docker

```shell
$ docker ps // 查看所有正在运行容器
$ docker stop containerId // containerId 是容器的ID

$ docker ps -a // 查看所有容器
$ docker ps -a -q // 查看所有容器ID

$ docker stop $(docker ps -a -q) //  stop停止所有容器
$ docker  rm $(docker ps -a -q) //   remove删除所有容器
```

查看端口

```shell
lsof -i
```

# FISCO BCOS快速搭建区块链网络（使用脚本）

## 1. 搭建单群组FISCO BCOS联盟链

### 第一步. 安装依赖

**安装ubuntu依赖**

```shell
sudo apt install -y openssl curl
```

### 第二步. 创建操作目录, 下载安装脚本

```shell
## 创建操作目录
cd ~ && mkdir -p fisco && cd fisco

## 下载脚本
curl -#LO https://github.com/FISCO-BCOS/FISCO-BCOS/releases/download/v2.9.1/build_chain.sh
```

```shell
##如果因为网络问题导致长时间无法下载build_chain.sh脚本，请尝试 
curl -#LO https://osp-1257653870.cos.ap-guangzhou.myqcloud.com/FISCO-BCOS/FISCO-BCOS/releases/v2.9.1/build_chain.sh && chmod u+x build_chain.sh
```

### 第三步. 搭建单群组4节点联盟链

在fisco目录下执行下面的指令，生成一条单群组4节点的FISCO链。 请确保机器的`30300~30303，20200~20203，8545~8548`端口没有被占用。

```shell
使用lsof -i查看端口占用
```

搭建

```shell
bash build_chain.sh -l 127.0.0.1:4 -p 30300,20200,8545
```

安装完成输出：

```shell
[INFO] Downloading fisco-bcos binary from https://github.com/FISCO-BCOS/FISCO-BCOS/releases/download/v2.9.1/fisco-bcos.tar.gz ...
#                                                2.8%curl: (28) Operation too slow. Less than 102400 bytes/sec transferred the last 20 seconds

[INFO] Download speed is too low, try https://osp-1257653870.cos.ap-guangzhou.myqcloud.com/FISCO-BCOS/FISCO-BCOS/releases/v2.9.1/fisco-bcos.tar.gz
############################################## 100.0%
==============================================================
Generating CA key...
==============================================================
Generating keys and certificates ...
Processing IP=127.0.0.1 Total=4 Agency=agency Groups=1
==============================================================
Generating configuration files ...
Processing IP=127.0.0.1 Total=4 Agency=agency Groups=1
==============================================================
[INFO] Start Port      : 30300 20200 8545
[INFO] Server IP       : 127.0.0.1:4
[INFO] Output Dir      : /home/ubuntu/fisco/nodes
[INFO] CA Path         : /home/ubuntu/fisco/nodes/cert/
[INFO] RSA channel     : true
==============================================================
[INFO] Execute the download_console.sh script in directory named by IP to get FISCO-BCOS console.
e.g.  bash /home/ubuntu/fisco/nodes/127.0.0.1/download_console.sh -f
==============================================================
[INFO] All completed. Files in /home/ubuntu/fisco/nodes
```

### 第四步. 启动FISCO BCOS链

- 启动所有节点

```shell
bash nodes/127.0.0.1/start_all.sh
```

输出成果结果

```shell
try to start node0
try to start node1
try to start node2
try to start node3
 node1 start successfully
 node2 start successfully
 node0 start successfully
 node3 start successfully
```

### 第五步. 检查进程

- 检查进程是否启动

```shell
ps -ef | grep -v grep | grep fisco-bcos
```

正常情况会有类似下面的输出； 如果进程数不为4，则进程没有启动（一般是端口被占用导致的）

输出结果

```shell
ubuntu   1738799       1  1 13:04 pts/0    00:00:00 /home/ubuntu/fisco/nodes/127.0.0.1/node0/../fisco-bco -c config.ini
ubuntu   1738805       1  1 13:04 pts/0    00:00:00 /home/ubuntu/fisco/nodes/127.0.0.1/node2/../fisco-bco -c config.ini
ubuntu   1738807       1  1 13:04 pts/0    00:00:00 /home/ubuntu/fisco/nodes/127.0.0.1/node3/../fisco-bco -c config.ini
ubuntu   1738809       1  1 13:04 pts/0    00:00:00 /home/ubuntu/fisco/nodes/127.0.0.1/node1/../fisco-bco -c config.ini
```

### 第六步. 检查日志输出

- 如下，查看节点node0链接的节点数

```shell
tail -f nodes/127.0.0.1/node0/log/log*  | grep connected
```

正常情况会不停地输出连接信息，从输出可以看出node0与另外3个节点有连接。

```shell
info|2019-01-21 17:30:58.316769| [P2P][Service] heartBeat,connected count=3
info|2019-01-21 17:31:08.316922| [P2P][Service] heartBeat,connected count=3
info|2019-01-21 17:31:18.317105| [P2P][Service] heartBeat,connected count=3
```

- 执行下面指令，检查是否在共识

```shell
tail -f nodes/127.0.0.1/node0/log/log*  | grep +++
```

正常情况会不停输出`++++Generating seal`，表示共识正常。

## 2. 配置及使用控制台

在控制台链接FISCO BCOS节点，实现**查询区块链状态、部署调用合约**等功能，能够快速获取到所需要的信息。2.6版本控制台指令详细介绍[参考这里](https://fisco-bcos-documentation.readthedocs.io/zh_CN/latest/docs/console/console_of_java_sdk.html)，1.x版本控制台指令详细介绍[参考这里](https://fisco-bcos-documentation.readthedocs.io/zh_CN/latest/docs/console/console.html)。

### 第一步. 准备依赖

- 安装java （推荐使用java 14）.

```shell
# ubuntu系统安装java
sudo apt install -y default-jdk
```

通过`java -version`查看当前java版本：

```shell
openjdk version "11.0.18" 2023-01-17
OpenJDK Runtime Environment (build 11.0.18+10-post-Ubuntu-0ubuntu120.04.1)
OpenJDK 64-Bit Server VM (build 11.0.18+10-post-Ubuntu-0ubuntu120.04.1, mixed mode, sharing)
```

- 获取控制台并回到fisco目录

```shell
cd ~/fisco && curl -LO https://github.com/FISCO-BCOS/console/releases/download/v2.9.2/download_console.sh && bash download_console.sh
```

```shell
如果因为网络问题导致长时间无法下载，请尝试 cd ~/fisco && curl -#LO https://gitee.com/FISCO-BCOS/console/raw/master-2.0/tools/download_console.sh && bash download_console.sh
```

- 拷贝控制台配置文件

若节点未采用默认端口，请将文件中的20200替换成节点对应的channel端口。

```shell
# 最新版本控制台使用如下命令拷贝配置文件
cp -n console/conf/config-example.toml console/conf/config.toml
```

- 配置控制台证书

```shell
cp -r nodes/127.0.0.1/sdk/* console/conf/
```

### 第二步. 启动并使用控制台

- 启动

```shell
cd ~/fisco/console && bash start.sh
```

输出`FISCO BCOS`标志标识输出成果，否则请检查conf/config.toml中节点端口配置是否正确

- 用控制台获取信息

  ```shell
  # 获取客户端版本
  [group:1]> getNodeVersion---->这一个是命令
  # 下面的是输出
  ClientVersion{
      version='2.6.0',
      supportedVersion='2.6.0',
      chainId='1',
      buildTime='20200819 15:47:59',
      buildType='Darwin/appleclang/RelWithDebInfo',
      gitBranch='HEAD',
      gitCommitHash='e4a5ef2ef64d1943fccc4ebc61467a91779fb1c0'
  }
  # 获取节点信息
  [group:1]> getPeers---->这一个是命令
  # 下面的是输出
  [
      PeerInfo{
          nodeID='c1bd77e188cd0783256ee06838020f24a697f9af785438403d3620967a4a3612e3abc4bbe986d1e9dddf62d4236bff0b7d19a935a3cd44889f681409d5bf8692',
          ipAndPort='127.0.0.1:30302',
          agency='agency',
          topic=[
  
          ],
          node='node2'
      },
      PeerInfo{
          nodeID='7f27f5d67f104eacf689790f09313e4343e7887a1a7b79c31cd151be33c7c8dd57c895a66086c3c8e0b54d2fa493407e0d9646b2bd9fc29a94fd3663a5332e6a',
          ipAndPort='127.0.0.1:57266',
          agency='agency',
          topic=[
              _block_notify_1
          ],
          node='node1'
      },
      PeerInfo{
          nodeID='862f26d9681ed4c12681bf81a50d0b8c66dd5b6ee7b0b42a4af12bb37b1ad2442f7dcfe8dac4e737ce9fa46aa94d904e8c474659eabf575d6715995553245be5',
          ipAndPort='127.0.0.1:30303',
          agency='agency',
          topic=[
  
          ],
          node='node3'
      }
  ]
  
  [group:1]>
  ```

## 3. 部署及调用HelloWorld合约

### 第一步. 编写HelloWorld合约

HelloWorld合约提供两个接口，分别是`get()`和`set()`，用于获取/设置合约变量`name`。合约内容如下:

```go
pragma solidity ^0.4.24;

contract HelloWorld {
    string name;

    function HelloWorld() {
        name = "Hello, World!";
    }

    function get()constant returns(string) {
        return name;
    }

    function set(string n) {
        name = n;
    }
}
```

### 第二步. 部署HelloWorld合约

为了方便用户快速体验，HelloWorld合约已经内置于控制台中，位于控制台目录下`contracts/solidity/HelloWorld.sol`，参考下面命令部署即可。

```shell
# 在控制台输入以下指令 部署成功则返回合约地址
[group:1]> deploy HelloWorld
transaction hash: 0xd0305411e36d2ca9c1a4df93e761c820f0a464367b8feb9e3fa40b0f68eb23fa
contract address:0xb3c223fc0bf6646959f254ac4e4a7e355b50a344
```

### 第三步. 调用HelloWorld合约

注：这里的交易地址需要填写自己上面成果部署的contract地址：

```shell
contract address:0xb3c223fc0bf6646959f254ac4e4a7e355b50a344
```

```shell
# 查看当前块高
[group:1]> getBlockNumber
1

# 调用get接口获取name变量 此处的合约地址是deploy指令返回的地址
[group:1]> call HelloWorld 0xb3c223fc0bf6646959f254ac4e4a7e355b50a344 get
---------------------------------------------------------------------------------------------
Return code: 0
description: transaction executed successfully
Return message: Success
---------------------------------------------------------------------------------------------
Return values:
[
    "Hello,World!"
]
---------------------------------------------------------------------------------------------

# 查看当前块高，块高不变，因为get接口不更改账本状态
[group:1]> getBlockNumber
1

# 调用set设置name
[group:1]> call HelloWorld 0xb3c223fc0bf6646959f254ac4e4a7e355b50a344 set "Hello, FISCO BCOS"
-----------------------------
transaction hash: 0x7e742c44091e0d6e4e1df666d957d123116622ab90b718699ce50f54ed791f6e
---------------------------------------------------------------------------------------------
transaction status: 0x0
description: transaction executed successfully
---------------------------------------------------------------------------------------------
Output
Receipt message: Success
Return message: Success
---------------------------------------------------------------------------------------------
Event logs
Event: {}

# 再次查看当前块高，块高增加表示已出块，账本状态已更改
[group:1]> getBlockNumber
2

# 调用get接口获取name变量，检查设置是否生效
[group:1]> call HelloWorld 0xb3c223fc0bf6646959f254ac4e4a7e355b50a344 get
---------------------------------------------------------------------------------------------
Return code: 0
description: transaction executed successfully
Return message: Success
---------------------------------------------------------------------------------------------
Return values:
[
    "Hello,FISCO BCOS"
]
---------------------------------------------------------------------------------------------

# 退出控制台
[group:1]> quit
```

可以看到，现在我们生成了2个区块，一个是 `Hello World` 一个是 `Hello ficso bcos` 他们的交易地址是不一样的，分别是

```
0xb3c223fc0bf6646959f254ac4e4a7e355b50a344
```

```
0x7e742c44091e0d6e4e1df666d957d123116622ab90b718699ce50f54ed791f6e
```

# 开发第一个区块链应用

要求用户熟悉Linux操作环境，具备Java开发的基本技能，能够使用Gradle工具，熟悉[Solidity语法](https://solidity.readthedocs.io/en/latest/)。

## 1. 了解应用需求

区块链天然具有防篡改，可追溯等特性，这些特性决定其更容易受金融领域的青睐。本示例中，将会提供一个简易的资产管理的开发示例，并最终实现以下功能：

- 能够在区块链上进行资产注册
- 能够实现不同账户的转账
- 可以查询账户的资产金额

## 2. 设计与开发智能合约

在区块链上进行应用开发时，结合业务需求，首先需要设计对应的智能合约，确定合约需要储存的数据，在此基础上确定智能合约对外提供的接口，最后给出各个接口的具体实现。

### 第一步. 设计智能合约

**存储设计**

FISCO BCOS提供[合约CRUD接口](https://fisco-bcos-documentation.readthedocs.io/zh_CN/latest/docs/manual/smart_contract.html#crud)开发模式，可以通过合约创建表，并对创建的表进行增删改查操作。针对本应用需要设计一个存储资产管理的表`t_asset`，该表字段如下：

- account: 主键，资产账户(string类型)
- asset_value: 资产金额(uint256类型)

其中account是主键，即操作`t_asset`表时需要传入的字段，区块链根据该主键字段查询表中匹配的记录。`t_asset`表示例如下：

| account | asset_value |
| ------- | ----------- |
| Alice   | 10000       |
| Bob     | 20000       |

**接口设计**

按照业务的设计目标，需要实现资产注册，转账，查询功能，对应功能的接口如下：

```java
// 查询资产金额
function select(string account) public constant returns(int256, uint256)
// 资产注册
function register(string account, uint256 amount) public returns(int256)
// 资产转移
function transfer(string from_asset_account, string to_asset_account, uint256 amount) public returns(int256)
```

### 第二步. 开发源码

根据我们第一步的存储和接口设计，创建一个Asset的智能合约，实现注册、转账、查询功能，并引入一个叫Table的系统合约，这个合约提供了CRUD接口。

```shell
# 进入console/contracts目录
cd ~/fisco/console/contracts/solidity
# 创建Asset.sol合约文件
vi Asset.sol

# 将Assert.sol合约内容写入。
# 并键入wq保存退出。
```

Asset.sol的内容如下：

```solidity
pragma solidity ^0.4.24;

import "./Table.sol";

contract Asset {
    // event
    event RegisterEvent(int256 ret, string account, uint256 asset_value);
    event TransferEvent(int256 ret, string from_account, string to_account, uint256 amount);

    constructor() public {
        // 构造函数中创建t_asset表
        createTable();
    }

    function createTable() private {
        TableFactory tf = TableFactory(0x1001);
        // 资产管理表, key : account, field : asset_value
        // |  资产账户(主键)      |     资产金额       |
        // |-------------------- |-------------------|
        // |        account      |    asset_value    |
        // |---------------------|-------------------|
        //
        // 创建表
        tf.createTable("t_asset", "account", "asset_value");
    }

    function openTable() private returns(Table) {
        TableFactory tf = TableFactory(0x1001);
        Table table = tf.openTable("t_asset");
        return table;
    }

    /*
    描述 : 根据资产账户查询资产金额
    参数 ：
            account : 资产账户

    返回值：
            参数一： 成功返回0, 账户不存在返回-1
            参数二： 第一个参数为0时有效，资产金额
    */
    function select(string account) public constant returns(int256, uint256) {
        // 打开表
        Table table = openTable();
        // 查询
        Entries entries = table.select(account, table.newCondition());
        uint256 asset_value = 0;
        if (0 == uint256(entries.size())) {
            return (-1, asset_value);
        } else {
            Entry entry = entries.get(0);
            return (0, uint256(entry.getInt("asset_value")));
        }
    }

    /*
    描述 : 资产注册
    参数 ：
            account : 资产账户
            amount  : 资产金额
    返回值：
            0  资产注册成功
            -1 资产账户已存在
            -2 其他错误
    */
    function register(string account, uint256 asset_value) public returns(int256){
        int256 ret_code = 0;
        int256 ret= 0;
        uint256 temp_asset_value = 0;
        // 查询账户是否存在
        (ret, temp_asset_value) = select(account);
        if(ret != 0) {
            Table table = openTable();

            Entry entry = table.newEntry();
            entry.set("account", account);
            entry.set("asset_value", int256(asset_value));
            // 插入
            int count = table.insert(account, entry);
            if (count == 1) {
                // 成功
                ret_code = 0;
            } else {
                // 失败? 无权限或者其他错误
                ret_code = -2;
            }
        } else {
            // 账户已存在
            ret_code = -1;
        }

        emit RegisterEvent(ret_code, account, asset_value);

        return ret_code;
    }

    /*
    描述 : 资产转移
    参数 ：
            from_account : 转移资产账户
            to_account ： 接收资产账户
            amount ： 转移金额
    返回值：
            0  资产转移成功
            -1 转移资产账户不存在
            -2 接收资产账户不存在
            -3 金额不足
            -4 金额溢出
            -5 其他错误
    */
    function transfer(string from_account, string to_account, uint256 amount) public returns(int256) {
        // 查询转移资产账户信息
        int ret_code = 0;
        int256 ret = 0;
        uint256 from_asset_value = 0;
        uint256 to_asset_value = 0;

        // 转移账户是否存在?
        (ret, from_asset_value) = select(from_account);
        if(ret != 0) {
            ret_code = -1;
            // 转移账户不存在
            emit TransferEvent(ret_code, from_account, to_account, amount);
            return ret_code;

        }

        // 接受账户是否存在?
        (ret, to_asset_value) = select(to_account);
        if(ret != 0) {
            ret_code = -2;
            // 接收资产的账户不存在
            emit TransferEvent(ret_code, from_account, to_account, amount);
            return ret_code;
        }

        if(from_asset_value < amount) {
            ret_code = -3;
            // 转移资产的账户金额不足
            emit TransferEvent(ret_code, from_account, to_account, amount);
            return ret_code;
        }

        if (to_asset_value + amount < to_asset_value) {
            ret_code = -4;
            // 接收账户金额溢出
            emit TransferEvent(ret_code, from_account, to_account, amount);
            return ret_code;
        }

        Table table = openTable();

        Entry entry0 = table.newEntry();
        entry0.set("account", from_account);
        entry0.set("asset_value", int256(from_asset_value - amount));
        // 更新转账账户
        int count = table.update(from_account, entry0, table.newCondition());
        if(count != 1) {
            ret_code = -5;
            // 失败? 无权限或者其他错误?
            emit TransferEvent(ret_code, from_account, to_account, amount);
            return ret_code;
        }

        Entry entry1 = table.newEntry();
        entry1.set("account", to_account);
        entry1.set("asset_value", int256(to_asset_value + amount));
        // 更新接收账户
        table.update(to_account, entry1, table.newCondition());

        emit TransferEvent(ret_code, from_account, to_account, amount);

        return ret_code;
    }
}
```

Asset.sol所引用的Table.sol已在`~/fisco/console/contracts/solidity`目录下。该系统合约文件中的接口由FISCO BCOS底层实现。当业务合约需要操作CRUD接口时，均需要引入该接口合约文件。Table.sol 合约详细接口参考[这里](https://fisco-bcos-documentation.readthedocs.io/zh_CN/latest/docs/manual/smart_contract.html#crud)。

查看当前目录是否存在Table.sol:

```shell
ubuntu@VM-12-11-ubuntu:~/fisco/console/contracts/solidity$ ls
Asset.ol        KVTableTest.sol  TableTest.sol
Crypto.sol      ShaTest.sol
HelloWorld.sol  Table.sol
```

## 3. 编译智能合约

**`.sol`的智能合约需要编译成ABI和BIN文件才能部署至区块链网络上。有了这两个文件即可凭借Java SDK进行合约部署和调用**。但这种调用方式相对繁琐，需要用户根据合约ABI来传参和解析结果。为此，控制台提供的编译工具不仅可以编译出ABI和BIN文件，还可以自动生成一个与编译的智能合约同名的合约Java类。这个Java类是根据ABI生成的，帮助用户解析好了参数，提供同名的方法。当应用需要部署和调用合约时，可以调用该合约类的对应方法，传入指定参数即可。使用这个合约Java类来开发应用，可以极大简化用户的代码。

```shell
# 创建工作目录~/fisco
mkdir -p ~/fisco
# 下载控制台
cd ~/fisco && curl -#LO https://github.com/FISCO-BCOS/console/releases/download/v2.9.2/download_console.sh && bash download_console.sh

#如果因为网络问题导致长时间无法下载，请尝试
 cd ~/fisco && curl -#LO https://gitee.com/FISCO-BCOS/console/raw/master-2.0/tools/download_console.sh

# 切换到fisco/console/目录
cd ~/fisco/console/

# 若控制台版本大于等于2.8.0，编译合约方法如下:（可通过bash sol2java.sh -h命令查看该脚本使用方法）
bash sol2java.sh -p org.fisco.bcos.asset.contract

# 若控制台版本小于2.8.0，编译合约(后面指定一个Java的包名参数，可以根据实际项目路径指定包名)如下：
./sol2java.sh org.fisco.bcos.asset.contract
```

运行成功之后，将会在`console/contracts/sdk`目录生成java、abi和bin目录，如下所示。

```shell
# 其它无关文件省略
|-- abi # 生成的abi目录，存放solidity合约编译生成的abi文件
|   |-- Asset.abi
|   |-- Table.abi
|-- bin # 生成的bin目录，存放solidity合约编译生成的bin文件
|   |-- Asset.bin
|   |-- Table.bin
|-- contracts # 存放solidity合约源码文件，将需要编译的合约拷贝到该目录下
|   |-- Asset.sol # 拷贝进来的Asset.sol合约，依赖Table.sol
|   |-- Table.sol # 实现系统CRUD操作的合约接口文件
|-- java  # 存放编译的包路径及Java合约文件
|   |-- org
|        |--fisco
|             |--bcos
|                  |--asset
|                       |--contract
|                             |--Asset.java  # Asset.sol合约生成的Java文件
|                             |--Table.java  # Table.sol合约生成的Java文件
|-- sol2java.sh
```

java目录下生成了`org/fisco/bcos/asset/contract/`包路径目录，该目录下包含`Asset.java`和`Table.java`两个文件，其中`Asset.java`是Java应用调用`Asset.sol`合约需要的文件。

`Asset.java`的主要接口：

```java
package org.fisco.bcos.asset.contract;

public class Asset extends Contract {
    // Asset.sol合约 transfer接口生成
    public TransactionReceipt transfer(String from_account, String to_account, BigInteger amount);
    // Asset.sol合约 register接口生成
    public TransactionReceipt register(String account, BigInteger asset_value);
    // Asset.sol合约 select接口生成
    public Tuple2<BigInteger, BigInteger> select(String account) throws ContractException;

    // 加载Asset合约地址，生成Asset对象
    public static Asset load(String contractAddress, Client client, CryptoKeyPair credential);

    // 部署Assert.sol合约，生成Asset对象
    public static Asset deploy(Client client, CryptoKeyPair credential) throws ContractException;
}
```

其中load与deploy函数用于构造Asset对象，其他接口分别用来调用对应的solidity合约的接口。

## 4. 创建区块链应用项目

### 第一步. 安装环境

首先，我们需要安装JDK以及集成开发环境

- Java：JDK 14 （JDK1.8 至JDK 14都支持）

  首先，在官网上下载JDK14并安装

  然后，修改环境变量

  ```shell
  # 确认您当前的java版本
  $ java -version
  # 确认您的java路径
  $ ls /Library/Java/JavaVirtualMachines
  # 返回
  # jdk-14.0.2.jdk
  
  # 如果使用的是bash
  $ vim .bash_profile 
  # 在文件中加入JAVA_HOME的路径
  # export JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk-14.0.2.jdk/Contents/Home 
  $ source .bash_profile
  
  # 如果使用的是zash
  $ vim .zashrc
  # 在文件中加入JAVA_HOME的路径
  # export JAVA_HOME = Library/Java/JavaVirtualMachines/jdk-14.0.2.jdk/Contents/Home 
  $ source .zashrc
  
  # 确认您的java版本
  $ java -version
  # 返回
  # java version "14.0.2" 2020-07-14
  # Java(TM) SE Runtime Environment (build 14.0.2+12-46)
  # Java HotSpot(TM) 64-Bit Server VM (build 14.0.2+12-46, mixed mode, sharing)
  ```

·IDE：IntelliJ IDE.

安装jetbrain 的ide

### 第二步. 创建一个Java工程

在IntelliJ IDE中创建一个gradle项目，勾选Gradle和Java，并输入工程名`asset-app`

在IntelliJ IDE中创建一个gradle项目，勾选Gradle和Java，并输入工程名`asset-app`。

![../../_images/create_app_mid.gif](https://fisco-bcos-documentation.readthedocs.io/zh_CN/latest/_images/create_app_mid.gif)

注意：该项目的源码可以用以下方法获得并参考。（此步骤为非必须步骤）

```shell
$ cd ~/fisco
$ curl -#LO https://github.com/FISCO-BCOS/LargeFiles/raw/master/tools/asset-app.tar.gz
# 解压得到Java工程项目asset-app
$ tar -zxf asset-app.tar.gz
```

注解

- 如果因为网络问题导致长时间无法下载，请尝试将`199.232.28.133 raw.githubusercontent.com`追加到`/etc/hosts`中，或者请尝试 curl -#LO https://osp-1257653870.cos.ap-guangzhou.myqcloud.com/FISCO-BCOS/FISCO-BCOS/tools/asset-app.tar.gz

### 第三步. 引入FISCO BCOS Java SDK

在build.gradle文件中的`dependencies`下加入对FISCO BCOS Java SDK的引用。

```java
repositories {
    mavenCentral()
    maven {
        allowInsecureProtocol = true
        url "http://maven.aliyun.com/nexus/content/groups/public/"
    }
    maven {
        allowInsecureProtocol = true
        url "https://oss.sonatype.org/content/repositories/snapshots" 
    }
}
```

引入Java SDK jar包

```shell
testImplementation group: 'junit', name: 'junit', version: '4.12'
implementation ('org.fisco-bcos.java-sdk:fisco-bcos-java-sdk:2.9.1')
```

![import_sdk](/Users/fangxiao/Downloads/import_sdk.png)

### 第四步. 配置SDK证书

修改`build.gradle`文件，引入Spring框架。 ![../../_images/import_spring.png](https://fisco-bcos-documentation.readthedocs.io/zh_CN/latest/_images/import_spring.png)

```java
def spring_version = "4.3.27.RELEASE"
List spring = [
        "org.springframework:spring-core:$spring_version",
        "org.springframework:spring-beans:$spring_version",
        "org.springframework:spring-context:$spring_version",
        "org.springframework:spring-tx:$spring_version",
]

dependencies {
    testImplementation group: 'junit', name: 'junit', version: '4.12'
    implementation ("org.fisco-bcos.java-sdk:fisco-bcos-java-sdk:2.9.1")
    implementation spring
}
```

在`asset-app/test/resources`目录下创建配置文件`applicationContext.xml`，写入配置内容。各配置项的内容可参考[Java SDK 配置说明](https://fisco-bcos-documentation.readthedocs.io/zh_CN/latest/docs/sdk/java_sdk/configuration.html)，该配置说明以toml配置文件为例，本例中的配置项与该配置项相对应。 