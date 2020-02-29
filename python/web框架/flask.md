
### 一、概述
* 优点
    * 轻巧，简洁，Flask的路由以及路由函数由修饰器设定，开发人员不需要借助其他文件匹配
    * 配置灵活，有多种方法配置，不同环境的配置也非常方便；环境部署简单，Flask运行不需要借助其他任何软件，只需要安装了Python的IDE，在命令行运行即可。只需要在Python中导入相应包即可满足所有需求；
    * 入门简单，通过官方指南便可以清楚的了解Flask的运行流程；
    * 低耦合，Flask可以兼容多种数据库、模板。
    * 适用于小型网站，适用于开发web服务的API


* 缺点
    * 对于大型网站开发，需要设计路由映射的规则，否则导致代码混乱


### 二、安装
```
$ pip install flask
```

### 三、demo
```
from flask import Flask
app = Flask(__name__)   # 创建实例

@app.route('/') # 构建根目录'/'路由
def index():    # 具体实现路由函数
    return '<h1>hello world, flask</h1>'

if __name__ == '__main__':
    app.run(debug=True, port=8777)
```

### 四、WSGI
实际生产中，python 程序是放在服务器的 http server（比如 apache， nginx 等）上的。现在的问题是 服务器程序怎么把接受到的请求传递给 python 呢，怎么在网络的数据流和 python 的结构体之间转换呢？这就是 wsgi 做的事情：一套关于程序端和服务器端的规范，或者说统一的接口。



参考:   
[Flask基本框架](https://www.jianshu.com/p/0f528c47c5bf)     
[flask 源码解析：简介](https://cizixs.com/2017/01/10/flask-insight-introduction/)   
[python wsgi 简介](https://cizixs.com/2014/11/08/understand-wsgi/)      
[Flask源码分析](https://pham-nuwens-document.readthedocs.io/zh/latest/flask%E5%AD%A6%E4%B9%A0/flask%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90.html)       
[Flask源码剖析](http://mingxinglai.com/cn/2016/08/flask-source-code/)