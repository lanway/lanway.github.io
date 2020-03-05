## windows下scrapyd安装以及使用

---

1. ### 安装

   ```python
   pip3 install scrapy
   pip3 install scrapyd
   pip3 install scrapyd-client
   版本要求：scrapy=1.6.0
   		Twisted=18.9.0	
   ```

   最早的时候用过版本scrapy=1.8.0，Twisted=19.10.0

   后来scrapyd-deploy部署的时候失败，就回退到之前的版本	

2. ### 部署scrapyd.

   在python虚拟环境中找到scrapyd，命令行中输入scrapyd

   ```
   D:\Pycharmproject\crawlproject\venv\Scripts>scrapyd
   ```

   默认端口为6800，浏览器中访问本地的6800端口可以查看scrapyd的监控界面

![image-20200301231233677](https://lanway.github.io/img/post/2020-03-04-windows下scrapyd安装以及使用/image1.png)

3. ### scrapyd-client配置

   安装并部署好scrapyd之后，要将爬虫项目部署到scrapyd之中。

   * 配置需要部署的项目

     编辑需要部署的项目的scrapy.cfg文件(需要将哪一个爬虫部署到scrapyd中，就配置该项目的该文件)

     ```python
     [deploy:部署名(部署名可以自行定义)]
      url = http://127.0.0.1:6800/
      project = 项目名(创建爬虫项目时使用的名称)
     ```

     <img src="https://lanway.github.io/img/post/2020-03-04-windows下scrapyd安装以及使用/image2.png" alt="image-20200301231735138" style="zoom: 200%;" />

   * 部署项目到scrapyd中

     在scrapy项目路径下执行：

     ```
     scrapyd-deploy 部署名 -p 项目名称 --version
     scrapyd-deploy <target>  -p <project> --version
     ```

     部署名就是代码中deploy之后的名字，项目名称就是在scrapy中你想要叫的名字，version是项目的版本。例如：

     <img src="https://lanway.github.io/img/post/2020-03-04-windows下scrapyd安装以及使用/image3.png" alt="image-20200301232150817" style="zoom:200%;" />

     在window下会出现找不到scrapyd-deploy的情况:

     我们会看到在python环境中只有scrapyd-deploy这个没有后缀的文件，因此我们新建scrapyd-deploy.bat，其中的内容为：

     ```
     @echo off
     
     "D:\Pycharmproject\crawlproject\venv\Scripts\python.exe" "D:\Pycharmproject\crawlproject\venv\Scripts\scrapyd-deploy" %*
     ```

     就可以使用scrapyd-deploy命令了。

   * 生成蛋文件

     ```
    scrapyd-deploy --build-egg output.egg
     ```
   
   4. ### scrapyd api调用（python库python-scrapyd-api）
   
      * 检查服务的负载状态 daemonstatus.json ，在终端/Xshell输入如下命令 **（GET请求）**

      ```
   $ curl http://localhost:6800/daemonstatus.json
      
      结果返回：{"node_name": "ubuntu", "status": "ok", "finished": 0, "pending": 0, "running": 0}
      ```
   
      * 增加项目到服务器，如果项目已经存在，则增加一个新的版本 addversion.json **（POST请求）**

      ```
   $ curl http://localhost:6800/addversion.json -F project=crawltest -F version=LV1.0 -F egg=crawltest.egg
      
      结果返回 : {"status": "ok", "spiders": 3}
      ```
   
      * 列出所有项目 listprojects.json ：**(get请求)**

      ```
   $ curl http://localhost:6800/listprojects.json
      
      结果返回：{"status": "ok", "projects": ["Douban", "otherproject"]}
      ```
   
      * 启动一个爬虫项目 **（post请求）**

      ```
   $ curl http://localhost:6800/schedule.json -d project=crawltest -d spider=testspider
      
      结果返回：{"status": "ok", "jobid": "任意jobID ”}
      
      setting (string, 可选) - Scrapy爬虫运行的配置
      jobid (string, 可选) - 识别任务的id, 重写默认生成的 UUID
      _version (string, 可选) - 指定项目使用的版本号
      其他任何参数都可以作为爬虫参数
      ```
   
   * 取消、删除 **（post请求）**
   
     ```
     如果指定任务处于运行状态则会被终止, 如果处于待处理状态则被删除(cancel.json)
     
      $curl http://localhost:6800/cancel.json -d project=Douban -d job= 任意jobID
     
      结果返回 ：{"status": "ok", "prevstate": "running"}
     ```
   
      * 获取指定项目的待处理, 正在运行和已完成的任务列表listjobs.json**（GET请求）**
   
     ```
      $ curl http://localhost:6800/listjobs.json?project=myproject
          
      结果返回：
          
      {
          "status": "ok",
     
         "pending": [{"id": "78391cc0fcaf11e1b0090800272a6d06", "spider": "spider1"}],
     
         "running": [{"id": "422e608f9f28cef127b3d5ef93fe9399", "spider": "spider2", "start_time": "2020-03-01 10:14:03.594664"}],
     
         "finished": [{"id": "2f16646cfcaf11e1b0090800272a6d06", "spider": "spider3", "start_time": "2020-03-01 10:14:03.594664", "end_time": "2020-03-01 10:24:03.594664"}]
        
        }
     ```
   
   * 删除项目及其所有在服务器上的版本**delproject.json（POST请求）**
   
     ```
     $ curl http://localhost:6800/delproject.json -d project=myproject
     
        结果返回：{"status": "ok"}
     ```
   
     
   
   5. ### 坑
   
      在我运行爬虫时出现了一个这样的问题：
   
      爬虫运行了一秒就直接关闭了，在log文件中也没有报任何的错误，找了好久好久都没有发现问题在哪。
   
      最后在我的配置文件中发现了这么一段代码：
   
      ```
      custom_settings = {
              'LOG_LEVEL': 'DEBUG',
              'LOG_FILE': log_file_path
          }
      ```
   
      由于scrapyd会自动配置log_file的位置，所以但我们手动也配置了之后，两个地方的配置造成冲突，是的爬虫不能够正常启动，把这一段代码注释了之后，发现爬虫便可以正常工作了。
   
      