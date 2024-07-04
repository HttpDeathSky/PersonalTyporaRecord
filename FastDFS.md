# FastDFS

## 概述

### 简介

FastDFS是基于互联网应用的开源分布式文件系统，主要用于大中型网站存储资源文件，如图片、文档、音频、视频等。FastDFS采用类似GFS的架构，用纯C语言实现，支持Linux、FreeBSD、AIX等UNIX 系统。用户端只能通过专有API对文件进行存取访问，不支持POSIX接口方式。准确地讲，GFS以及 FastDFS、mogileFS、HDFS、TFS等类GFS系统都不是系统级的分布式文件系统，而是应用级的分布式文件存储服务。



1. FastDFS概述：

   - FastDFS是一个开源的轻量级分布式文件系统，他对文件进行管理，功能包括：文件存储、文件同步、文件访问（文件上传、下载）等，解决了大容量存储和负载均衡的问题，高度追求高性能和扩展性。特别适合以文件为载体的在线服务，如相册万盏、视频网站等等。
   - FastDFS是由纯C语言实现，支持Linux，FreeBSD的NUIX系统。类google FS，不是通用的文件系统，只能够同故宫转悠API进行访问，目前提供了C，Java，PHP API。另外，FastDFS可以看作是基于文件的key-Value存储系统，也可以称之为 分布式文件存储服务。
2. FastDFS提供的功能：
   - upload 上传文件
   - upload_appender：上传appender文件，后续可已对其进行append操作
   - upload_slave：上传从文件
   - download 下载文件
   - delete 删除文件
   - append：在已有文件后追加内容
   - set_metadata：设置文件附加属性
   - get_metadata：获取文件附加属性
3. FastDFS的特点：
   - 分组存储、灵活简洁
   - 对等结构、不存在单点
   - 文件ID有FastDFS生成，作为文件访问凭证。FastDFS不需要传统的name server
   - 和流行的web server无缝连接，FastDFS已提供apache和nginx扩展模块
   - 大、中、小文件均可以很好支持，支持海量小文件存储
   - 支持多块磁盘，支持但盘数据恢复
   - 支持相同文件内容只保存一份，节省存储空间
   - 存储服务器上可以保存文件附加属性
   - 下载文件支持多线程方式、支持断点续传
4. FastDFS架构解读：
   - 只有两个角色，tracker server和storage server，不需要存储文件索引信息
   - 所有服务器都是对等的，不存在Master-Slave关系
   - 存储服务器采用分组方式，同组内存储服务器上的文件完全相同
   - 不同组的storage server之间不会相互通信
   - 不同组的storage server之间不会相互通信
   - 有storage server主动向tracker server报告状态信息，tracker server 之间通常不会相互通信
5. FastDFS如何解决同步延迟问题？
   - storage生成的文件名中，包含源头storage IP地址和文件创建的时间戳
   - 源头storage定时向tracker报告同步情况，包括向目标服务器同步到的文件时间戳
   - tracker收到storage的同步报告后，找出该组内每台storage被同步到的时间戳（取最小值），作为storage属性保存到内存中
6. FastDFS扩展模块要点：
   - 使用扩展模块来解决文件同步延迟问题
   - 对每台storage server上部署web server，直接对外提供HTTP服务
   - tracker server上不需要部署web server
   - 如果请求文件在当前storage上不存在，通过文件ID反解出源storage，直接请求源storage
   - 目前已提供apache和nginx扩展模块
   - FastDFS扩展模块不依赖于FastDFS server，可以独立存在
7. FastDFS扩展模块特性：
   - 仅支持HTTP HEAD和GET
   - 支持token方式的防盗链（缺省是关闭的）
     - ts：生成token的时间（unix时间戳）
     - token：32未得token字符串（md5签名）
   - 支持制定保存的缺省文件名，URL参数名为filename
   - 支持断点续传
8. FastDFS工作原理
   - 参见博客：http://blog.csdn.net/liweizhong193516/article/details/52556526，中专门讲解FastDFS的工作原理，分析上传下载方式解析。

### 结构组成

`Client | Tracker Server | Storage Server`

## 大纲

## 结尾

## 附言