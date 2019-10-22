

#### 1、查看brew的当前源:
执行下面命令可以看到, brew的官方镜像源为: https://github.com/Homebrew/homebrew
```
cd /usr/local/Homebrew
git remote -v
```

#### 2、更改brew镜像源
```
cd /usr/local/Homebrew
git remote set-url origin git://mirrors.tuna.tsinghua.edu.cn/homebrew.git
```
`
清华镜像源: git://mirrors.tuna.tsinghua.edu.cn/homebrew.git
中科大镜像源: http://mirrors.ustc.edu.cn/homebrew.git
`

#### 3、Mac中使用brew安装指定版本软件包
* 查看软件包来源
```
brew info solidity
```
执行如下:
```
➜  openzeppelin-demo brew info solidity
ethereum/ethereum/solidity: stable 0.5.12
The Solidity Contract-Oriented Programming Language
http://solidity.readthedocs.org
Not installed
From: https://github.com/ethereum/homebrew-ethereum/blob/master/solidity.rb
==> Dependencies
Build: cmake ✔, ccache ✔
Required: boost --c++11 ✔, z3 ✔
```

复制`https://github.com/ethereum/homebrew-ethereum/blob/master/solidity.rb`到浏览器地址栏
可以看到当前版本是:
```
url "https://github.com/ethereum/solidity/releases/download/v0.5.12/solidity_0.5.12.tar.gz"
version "0.5.12"
sha256 "cad9d5eaee79ce542f62c9c454f49b69bdb17d37f000029b425155e9c6816561"
然后按下面的操作:
```
* 点击History查看历史提交列表
* 找到需要的版本点击进去
* 点击Browse files查看完整历史项目
* 点击solidity.rb文件
* 点击Raw
* 复制地址栏url
* 然后在mac终端执行`brew install 地址栏url`


参考:

[解决 "brew update" 无响应](https://www.jianshu.com/p/631e63dab0a0)  
[Mac中使用brew安装指定版本软件包](https://segmentfault.com/a/1190000015346120)