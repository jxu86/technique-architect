

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

参考:

[解决 "brew update" 无响应](https://www.jianshu.com/p/631e63dab0a0)