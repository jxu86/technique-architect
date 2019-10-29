


* 镜像源切换
    * 临时使用  
        ```
        npm --registry https://registry.npm.taobao.org install express
        ```
    * 持久使用  
        ```
        npm config set registry https://registry.npm.taobao.org
        npm config get registry     # 查看镜像源
        npm config --list           # 查看全部配置
        ```
    * 通过cnpm使用  
        ```
        npm install -g cnpm --registry=https://registry.npm.taobao.org
        cnpm install express
        ```
    * nrm切换源
        ```
        npm install -g nrm

        ➜  technique-architect git:(master) ✗ nrm ls
        * npm -------- https://registry.npmjs.org/
        yarn ------- https://registry.yarnpkg.com/
        cnpm ------- http://r.cnpmjs.org/
        taobao ----- https://registry.npm.taobao.org/
        nj --------- https://registry.nodejitsu.com/
        npmMirror -- https://skimdb.npmjs.com/registry/
        edunpm ----- http://registry.enpmjs.org/

        nrm use taobao # 切换源

        nrm add <源名称> <源地址>       #增加源
        nrm del                       #删除源

        
        nrm test npm                  #测试源的响应时间
        nrm test                      #测试所以源的响应时间
        ```

参考:  
[npm镜像切换大法](https://blog.csdn.net/bbatyki/article/details/82918851)