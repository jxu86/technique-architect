`环境:`
`macbook pro`
`python v3.7.4`


### 一、环境安装
`前提条件：已安装好Python开发环境（推荐安装Python3.5及以上版本`
* 安装selenium
```
pip install selenium
```

* 安装webdriver
各大浏览器webdriver地址可参见：https://docs.seleniumhq.org/download/    
Firefox：https://github.com/mozilla/geckodriver/releases/   
Chrome：https://sites.google.com/a/chromium.org/chromedriver/ 或者  
http://chromedriver.storage.googleapis.com/index.html   
IE：http://selenium-release.storage.googleapis.com/index.html   
注：webdriver需要和对应的浏览器版本以及selenium版本对应 

* webdriver安装路径
    * Win：复制webdriver到Python安装目录下
    * Mac：复制webdriver到/usr/local/bin目录下

### 二、启动浏览器
```
from selenium import webdriver
browser = webdriver.Chrome()
browser.get('http://www.baidu.com/')
```




官方文档:   
https://selenium-python.readthedocs.io/index.html   
https://selenium.dev/selenium/docs/api/py/api.html  

参考:   
[Python+Selenium基础入门及实践](https://www.jianshu.com/p/1531e12f8852)