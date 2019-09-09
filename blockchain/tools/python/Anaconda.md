Anaconda是一个开源的包、环境管理器，可以用于在同一个机器上安装不同版本的软件包及其依赖，并能够在不同的环境之间切换

### 安装
`linux & mac`
```
# 首先从清华的官方镜像下载(可以下载最新版本)
wget https://mirrors.tuna.tsinghua.edu.cn/anaconda/archive/Anaconda3-5.3.0-Linux-x86_64.sh

# 修改权限让脚本可以运行
chmod +x Anaconda3-5.3.0-Linux-x86_64.sh

# 运行该安装脚本
./Anaconda3-5.3.0-Linux-x86_64.sh

# 剩下就是一路Yes或者Enter好了...

# 重新加载一下 bash 就可以使用 `conda` 命令了
source ~/.bashrc
```
#然后尝试一下运行 `conda -V` 命令行看是否已经安装成功，如果返回对应的版本信息，则说明安装成功。


* 设置国内镜像


如果需要安装很多packages，你会发现conda下载的速度经常很慢，因为Anaconda.org的服务器在国外。所幸的是，清华TUNA镜像源有Anaconda仓库的镜像，我们将其加入conda的配置即可：

添加Anaconda的TUNA镜像
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/
TUNA的help中镜像地址加有引号，需要去掉
 
设置搜索时显示通道地址
conda config --set show_channel_urls yes


### 常用命令

* 查找虚拟环境

conda info --envs

conda env list

* 创建虚拟环境

conda create --name test python=3.7.4

* 切换虚拟环境

source activate test


* 退出环境

source deactivate

* 移除环境

conda remove -n your_env_name(虚拟环境名称) --all

* 删除环境中的某个包

conda remove --name $your_env_name  $package_name