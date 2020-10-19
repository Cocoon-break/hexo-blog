---
title: python 开发服务端
date: 2017-06-19 10:19:31
tags:
- Python
categories:
- Python
---

最新在开发微信公众号，选择了python做为后端开发的语言。之所以选择python作为开发语言，是因为python开发起来比较快，而且也容易上手。选用的python版本是python2.7.13



## 先了解下python的一些基本知识

- 在mac 上执行  `pip install xxx`会把包安装到哪里？

  如果Mac上装了多个版本的Python，可以同过`python  --version`查看，然后会将包安装到`/usr/local/lib/pythonxxx/site-packages`下

- 创建虚拟开发环境

  在使用Python开发的时候，各个应用所需的Python版本可能不同，这时我们就需要一个独立的Python开发环境。virtualenv就是用来为一个应用创建一套“隔离”的Python运行环境。

  - 安装virtualenv

    ```sh
    pip install virtualenv
    ```

  - 创建一个目录并创建一个独立的Python环境venv

    ```sh
    mkdir testPython
    cd testPython
    virtualenv --no-site-packages venv
    ```

    **注：** 参数`--no-site-packages`，不会将系统已安装的第三方包复制过来，这样我们就得到一个干净的Python开发环境

  - 使用这个venv环境

    ```sh
    source venv/bin/activate
    ```

    接下来`pip install xxx` 都会把Python包放在`testPython/venv/lib/python2.7/site-packages`目录下

  - 离开venv 环境

     ```sh
     deactivte
     ```

  virtualenv是如何创建“独立”的Python运行环境的呢？原理很简单，就是把系统Python复制一份到virtualenv的环境，用命令`source venv/bin/activate`进入一个virtualenv环境时，virtualenv会修改相关环境变量，让命令`python`和`pip`均指向当前的virtualenv环境。




​	**注：** 使用venv 环境之后，执行`pip install xxxx` ，它会把这些依赖包安装在`venv/lib/python2.7/site-packages`目录下

- 依赖第三方库的统一配置文件 **requirements.txt**

  ```text
  Flask==0.12
  ```

  通过`pip install -r requirements.txt`来安装文件中依赖的一些第三方库




<!-- more -->



## 项目开发

flask是python的一个轻量级Web应用框架，对于一些小的站点来说flask，已经能够满足需求了。执行`pip install flask`安装。[flask 官网](http://flask.pocoo.org/)

创建一个run.py文件，写如以下代码。然后执行python run.py，一个简易的web服务就OK了。访问localhost:8080端口

```python
from flask import Flask, request

app = Flask(__name__)

@app.route('/test', methods=['GET', 'POST'])
def test():
    return jsonify({'msg':'hello world'})

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=8080)
```



链接数据库使用的sqlalchemy,	sqlalchemy 是Python SQL工具包和对象关系映射器，为应用程序开发人员提供了SQL的全部功能和灵活性。[SQLAlchemy官网](http://www.sqlalchemy.org/)


- 本地机器已经安装了mysql，但是在使用sqlalchemy 报No module named mysql。

   ```python 
   pip install pymysql
   ```

- 链接数据库样例

   ```python
   from sqlalchemy import Column, String, create_engine

   # 初始化数据库连接:
   engine = create_engine('mysql+pymysql://'+username+':'+password+'@localhost:3306/doudouSpace')

   def create_user_table():
       connect = engine.connect()
       try:
           result = connect.execute('xxxx') #xxx 表示sql语句
           result.close()
       except Exception as e:
           print e

   ```

   **注：**更多详细文档请参考官网



日志模块也是开发工作中的一个重要部分，可以方便我们调试开发。我们使用python自带的logging模块，我们要将日志信息输出到文件中，而不是只输出到控制台中。

- 封装下日志模块

  ```python
  import logging
  from logging.handlers import RotatingFileHandler
  from . import config
  
  log_filename = 'output.log'
  logging.basicConfig(level=logging.DEBUG,
                      format='%(filename)s:%(lineno)s - %(asctime)s %(levelname)s %(message)s',
                      filename=log_filename,
                      filemode='a')
  
  logger = logging.getLogger()
  log_handler = logging.handlers.WatchedFileHandler(log_filename)
  logger.addHandler(log_handler)
   
  def debug(msg):
      # logging.debug(msg)
      logger.debug(msg)
  
  def info(msg):
      # logging.info(msg)
      logger.info(msg)
  
  def warning(msg):
      # logging.warning(msg)
      logger.warning(msg)
  
  def error(msg):
      # logging.error(msg)
      logger.error(msg)
      
  ```
  其他的一些在这里就不在过多的介绍了，我做了一个python开发后端rest接口的模版工程，放在GitHub上，有兴趣的话可以看看[python_tmplate](https://github.com/Cocoon-break/python_tmplate) 