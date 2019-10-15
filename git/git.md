



* 同时提交多个git仓库  
    * git pull & git push 更新主仓库代码,保持本地与主仓库代码一致
    * 配置项目路径下的.git/config文件，添加备份仓库的git地址到url，每多一个仓库增加一条url配置
    ```
    [remote "origin"]
	url = http://10.1.53.122/xjcun/asset-data-sync.git
        url = https://gitee.com/JC86/asset-data-sync.git
	fetch = +refs/heads/*:refs/remotes/origin/*
    ```
    * git push -f 强制提交到所有仓库，统一版本

* tag  
    * 创建tag
    ```
    git tag v1.0

    ```
    * 同步tag到远程服务器
    ```
    git push origin v1.0
    ```
    
    参考:  
    https://www.jianshu.com/p/cdd80dd15593
