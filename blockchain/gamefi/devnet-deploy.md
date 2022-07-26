




## geth安装

到https://github.com/ethereum/go-ethereum下载版本1.10.8的源码到本地
然后执行`make all`安装相关的工具，设置环境变量等。   
安装完可以看到geth的版本(1.10.8-stable)
```
[root@localhost.localdomain geth-poa-node]# geth version
Geth
Version: 1.10.8-stable
Git Commit: 26675454bf93bf904be7a43cce6b3f550115ff90
Git Commit Date: 20210824
Architecture: amd64
Go Version: go1.16.4
Operating System: linux
GOPATH=/data/work/gopath
GOROOT=/usr/local/go
```

## 节点部署
部署教程: https://blog.csdn.net/wcc19840827/article/details/118891325   
根据上面教程可以部署出两个区块链节点。

## 智能合约部署
`环境:` 
`nodejs: v15.2.1`
`truffle: v5.4.10`

进入合约项目安装依赖包:
```
npm install
```
编译合约:
```
truffle compile
```
参数配置:
进入truffle-config.js对development进行设置
合约部署:
```
truffle mirgeta
```
部署过程:
```
Compiling your contracts...
===========================
> Everything is up to date, there is nothing to compile.



Starting migrations...
======================
> Network name:    'development'
> Network id:      5777
> Block gas limit: 20000000000000 (0x12309ce54000)


1_initial_migration.js
======================

   Replacing 'Migrations'
   ----------------------
   > transaction hash:    0x28793dbbeae8ca965ad0ca01a28e06dea9541847b93619e80483523ed74f3a42
   > Blocks: 0            Seconds: 0
   > contract address:    0xc6fB3eAeE16aB463767CAccaff19F5031114B4Ae
   > block number:        10771
   > block timestamp:     1654595164
   > account:             0xB717AD9e996ACd68D1D48C30f0cCeD6D0bB32DDa
   > balance:             99993.637405573134566685
   > gas used:            178486 (0x2b936)
   > gas price:           0.000000001 gwei
   > value sent:          0 ETH
   > total cost:          0.000000000000178486 ETH


   > Saving migration to chain.
   > Saving artifacts
   -------------------------------------
   > Total cost:     0.000000000000178486 ETH


2_initial_mecha.js
==================

   Replacing 'MechaCore'
   ---------------------
   > transaction hash:    0x3beb4c46b9aff44b0439489a059e3bc0b7a1c019bae231def514b2370d6ec444
   > Blocks: 0            Seconds: 0
   > contract address:    0x44E0f78C79B30Fe7D00c1725E791AC02a38Ded2b
   > block number:        10773
   > block timestamp:     1654595166
   > account:             0xB717AD9e996ACd68D1D48C30f0cCeD6D0bB32DDa
   > balance:             99993.637405573128757102
   > gas used:            5767305 (0x580089)
   > gas price:           0.000000001 gwei
   > value sent:          0 ETH
   > total cost:          0.000000000005767305 ETH


   > Saving migration to chain.
   > Saving artifacts
   -------------------------------------
   > Total cost:     0.000000000005767305 ETH


3_initial_sec.js
================

   Replacing 'SECERC20'
   --------------------
   > transaction hash:    0x94fa38a5b9752e6e41d08ecfd661f3ea37a396736faeb1d8d87b2489da9f7fce
   > Blocks: 0            Seconds: 0
   > contract address:    0x11a7412a00Cba5c84F8F6B53B82528099456976E
   > block number:        10775
   > block timestamp:     1654595167
   > account:             0xB717AD9e996ACd68D1D48C30f0cCeD6D0bB32DDa
   > balance:             99993.637405573127922412
   > gas used:            807412 (0xc51f4)
   > gas price:           0.000000001 gwei
   > value sent:          0 ETH
   > total cost:          0.000000000000807412 ETH


   > Saving migration to chain.
   > Saving artifacts
   -------------------------------------
   > Total cost:     0.000000000000807412 ETH


4_initial_mec.js
================

   Replacing 'MECERC20'
   --------------------
   > transaction hash:    0x560baa4633dc9a5e8cbeb7f6f63984b6455687e60bb1aa3d3109cf4b206a5243
   > Blocks: 0            Seconds: 0
   > contract address:    0x478aD46dc753B13A41AF7c04d438647C5e9e56f7
   > block number:        10777
   > block timestamp:     1654595168
   > account:             0xB717AD9e996ACd68D1D48C30f0cCeD6D0bB32DDa
   > balance:             99993.637405573127045041
   > gas used:            850093 (0xcf8ad)
   > gas price:           0.000000001 gwei
   > value sent:          0 ETH
   > total cost:          0.000000000000850093 ETH


   > Saving migration to chain.
   > Saving artifacts
   -------------------------------------
   > Total cost:     0.000000000000850093 ETH


5_initial_hlc.js
================

   Replacing 'HLCERC20'
   --------------------
   > transaction hash:    0x2f5aaefb9b5432de95dd0364d8707752714e04c41aa8b717594263ec2d2386d8
   > Blocks: 0            Seconds: 0
   > contract address:    0xC901a333a5A307F91A376d308BaF5A9dFC5Aa001
   > block number:        10779
   > block timestamp:     1654595168
   > account:             0xB717AD9e996ACd68D1D48C30f0cCeD6D0bB32DDa
   > balance:             99993.63740557312581608
   > gas used:            1201683 (0x125613)
   > gas price:           0.000000001 gwei
   > value sent:          0 ETH
   > total cost:          0.000000000001201683 ETH


   > Saving migration to chain.
   > Saving artifacts
   -------------------------------------
   > Total cost:     0.000000000001201683 ETH


6_initial_mlc.js
================

   Replacing 'MLCERC20'
   --------------------
   > transaction hash:    0xdae1501abf06c49fb5e16ebdde94de3d18eec4e84e0aa9025ab00d0ca5bfc05b
   > Blocks: 0            Seconds: 0
   > contract address:    0x4a2e38eCacF901A041D62aE4100620Ab93160782
   > block number:        10781
   > block timestamp:     1654595169
   > account:             0xB717AD9e996ACd68D1D48C30f0cCeD6D0bB32DDa
   > balance:             99993.637405573124587119
   > gas used:            1201683 (0x125613)
   > gas price:           0.000000001 gwei
   > value sent:          0 ETH
   > total cost:          0.000000000001201683 ETH


   > Saving migration to chain.
   > Saving artifacts
   -------------------------------------
   > Total cost:     0.000000000001201683 ETH


7_initial_llc.js
================

   Replacing 'LLCERC20'
   --------------------
   > transaction hash:    0x58d5a9af6cfcf11b1ea52f15571c6619ce5ed9a28328db243d2ad00533e31967
   > Blocks: 0            Seconds: 0
   > contract address:    0x408Bb545E188bfa9Edd0659c749C5e01E4e1E0d7
   > block number:        10783
   > block timestamp:     1654595170
   > account:             0xB717AD9e996ACd68D1D48C30f0cCeD6D0bB32DDa
   > balance:             99993.637405573123358158
   > gas used:            1201683 (0x125613)
   > gas price:           0.000000001 gwei
   > value sent:          0 ETH
   > total cost:          0.000000000001201683 ETH


   > Saving migration to chain.
   > Saving artifacts
   -------------------------------------
   > Total cost:     0.000000000001201683 ETH


Summary
=======
> Total deployments:   7
> Final cost:          0.000000000011208345 ETH
```