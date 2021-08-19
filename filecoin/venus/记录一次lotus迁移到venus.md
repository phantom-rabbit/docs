## 将lotus-miner迁移到venus
### 背景
  本着学习的态度开始研究venus,venus将lotus的模块拆分的更加细致对于初学者或者没有技术团队的公司或者个人来讲是非常容器上手的。这次学习的目标是将一个lotus-miner 运行的矿工转移到venus上运行，保证wdPoSt能正常证明。
+ lotus-miner 版本v1.10.1

### 环境准备
+ venus的组件分为公共组件和私有组件，公共组件由venus团队提供，本次学习旨在搭建私有组件用最少的资源启动成功
+ 设计到的代码仓库地址 https://github.com/ipfs-force-community/lotus 分支v1.10.1_incubation
+ 
+ 构建工具
  Ubuntu/Debian:
  ```
  sudo apt install mesa-opencl-icd ocl-icd-opencl-dev gcc git bzr jq pkg-config curl clang build-essential hwloc libhwloc-dev wget -y && sudo apt upgrade -y
  ```
  Rust
  ```
  curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
  ```
  Go 目前venus指定版本Go 1.16(尝试过1.17编译成功运行失败)
  ```
  wget -c https://golang.org/dl/go1.16.7.linux-amd64.tar.gz -O - | sudo tar -xz -C /usr/local
  ```
  将 /usr/local/go/bin 添加到您的路径并设置go env。对于大多数Linux系统，您可以运行以下内容：
  ```
  echo "export PATH=$PATH:/usr/local/go/bin" >> ~/.bashrc && source ~/.bashrc
  # setup go env
  go env -w GOPROXY=https://goproxy.io,direct
  go env -w GO111MODULE=on
  ```

## 钱包组件
  钱包组件类似于签名机，主要管理钱包私钥以及签名工作。
```
$ git clone https://github.com/filecoin-project/venus-wallet.git
# change directory to venus-wallet
$ cd venus-wallet
$ git checkout -b incubation origin/incubation
$ RUSTFLAGS="-C target-cpu=native -g" FFI_BUILD_FROM_SOURCE=0 make
```
###### 编译好之后运行并设置密码
```
$ ./venus-wallet run

# set password to protect wallet security (Used for AES encryption, private key, root token seed)
$ ./venus-wallet set-password
Password:******
Enter Password again:******
```
+ 注意
    请备份您的密码并妥善保存，否则你将无法使用wallet相关的功能。每次venus-wallet run启动时带--password标志会自动解锁钱包,如果没带,在wallet实例启动后你需要手动解锁:
    ```
    $ ./venus-wallet unlock
    Password: 

    # 查看解锁状态
    $ ./venus-wallet lock-state
    wallet state: unlocked
    ```
    在扇区封装的过程中需要调用wallet进行签名,如果不解锁,会导致签名失败,进而导致扇区任务失败.

###### 配置公共组件

    配置venus-wallet接入共享服务,使用您从共享模块管理员处获得的帐号信   息更改 ~/.venus_wallet/config.toml中的[APIRegisterHub] 部分
    ```
    [APIRegisterHub]
    RegisterAPI = ["/ip4/<IP_ADDRESS_OF_VENUS_GATEWAY>/tcp/ 45132"]
    Token = "<AUTH_TOKEN_FOR_ACCOUNT_NAME>"
    SupportAccounts = ["<ACCOUNT_NAME>"]
    ```
    配置完成后需要重启加载配置文件
    
###### 导入钱包
```
$ lotus-miner actor control list --verbose
# 将上面看到的地址全部导入到venus钱包
# 导入钱包需要先解锁钱包
```    

## 矿工

```
$ git clone https://github.com/ipfs-force-community/lotus
$ git checkout v1.10.1_incubation
$ cd lotus
$ go mod tidy
$ RUSTFLAGS="-C target-cpu=native -g" FFI_BUILD_FROM_SOURCE=1 make
```
+ 编译遇到以下错误需要执行 go get github.com/google/flatbuffers@v2.0.0 重新编译
![image](https://user-images.githubusercontent.com/49083897/130016951-23422e1c-1dbf-4445-94c0-ef3ab7d739ff.png)

+ 启动遇到以下错误需要指定golang 版本为1.16重新编译
![image](https://user-images.githubusercontent.com/49083897/130016735-c17fed91-878a-42e6-a315-9a04e0d9cce4.png)


###### 编译好之后先停掉lotus-miner(注意窗口期)
###### 修改lotus-miner 配置文件
```
$ vim ~/.lotusminer/config.conf
# 追加一下配置，配置内容想venus团队申请
[Venus]
  [Venus.Messager]
    Url = ""
    Token = ""
  [Venus.RegisterProofAPI]
    Urls = [""]
    Token = ""
  [Venus.Wallet]
    Url = ""
    Token = ""
```

+ 启动之前先联系venus团队，他们需要配置你的矿工号这样子你才能连接到公共组件

###### 启动
```
$ lotus-miner run
```
好了检查日志，没有报错就算启动成功啦，等待窗口挑战
