title: 使用Electron创建Mysql数据比对工具
date: 2016-04-26
tags: Electron,Mysql,数据比对
categories: Electron Mysql
---

# 使用Electron创建Mysql数据比对工具
## 项目由来
工作中的项目使用MySql，在学习系统的数据数量和测试服务时，经常需要去比对一个表两个时间点的数据变化。  
搜索相关工具不得，而之前又玩过Electron，一直没有找个机会练练手，因有此项目。  
项目源码已开源，地址：http://git.oschina.net/lontoken/MysqlComparer
<!--more-->


## 项目的用户手册
### 功能说明
MySql数据库的数据对比工具。  
比较同一个表两个时间点的数据，并在界面上展示比较结果。  
需要比较的数据库和表，可以通过配置文件配置。  

### 操作说明
单击"开始记录"按钮，将数据库中的数据加载到本地，做为初始镜像数据；  
单击"结束记录"按钮，将数据库中的数据加载到本地，做为终止镜像数据；  
单击"显示结果"按钮，会比较初始镜像数据和终止镜像数据，并将展示比较结果；  
"开始记录"和"结束记录"按钮可以单击多次，"结束记录"按钮将和最近的一次"开始记录"记录的数据比较；  

### 软件配置
通过config.json文件可以配置监控的数据库和表； 可以参考config_aliyun.json和config_local.json文件。


### 问题反馈
若有问题，请联系 ganlong@hundsun.com / lontoken@gmail.com。