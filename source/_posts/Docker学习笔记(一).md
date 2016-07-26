---
title: Docker学习笔记(一):Java Web
date: 2016-07-26 18:30:00
categories:
- Docker
tags:
- Docker
comments: true
---
因偶然接触到Docker，详细了解了Docker的应用场景觉得Docker的容器化确实是一种未来的趋势难怪自Docker发布以来一直这么火爆，必定在多种场景中都会有所应用，所以我利用空余时间来学下Docker，在博客里记录下学习过程。Docker基本操作这系列文章里就不表述了。

在Java web应用比如利用SSH框架编写的web应用，使用Tomcat部署运行，在这个典型的场景中使用Docker来将其容器化，以便在开发-测试-部署一整套流程中避免遇到环境不一致等很多不必要的麻烦。

以此我们使用3个docker容器来运行Web程序。分别运行tomcat、mysql以及一个数据存储容器专门用来存储数据库数据，将数据库程序与数据存储分离的好处就是但需要升级或者移值数据库时可以更加的方便。

这里我们使用Docker Compose来进行编排，什么是Docker Compose，官方文档描述的很清楚：
<!-- more -->
>Compose is a tool for defining and running multi-container Docker applications. With Compose, you use a Compose file to configure your application’s services. Then, using a single command, you create and start all the services from your configuration. 

简单的来说就是利用Docker Compose这一工具可以方便运行多容器的复杂Docker应用，将每个应用定义为服务，使用`docker-compose.yml`文件进行配置，docker compose将根据这一配置文件来自动构建及运行Docker应用。本文所描述的场景最简单配置如下：

``` YMAL
version: '2'
services:
  db:
    image: mysql:latest
    environment:
      MYSQL_ROOT_PASSWORD: 12345
      MYSQL_DATABASE: es
    volumes_from: 
      - dbstore

  web:
    image: tomcat:8.5
    volumes: 
      - "./es/target/es.war:/usr/local/tomcat/webapps/es.war"
    depends_on:
      - db
    ports:
      - "8888:8080"
    links:
      - db

  dbstore:
    image: centos:latest
    volumes:
      - "/var/lib/mysql"
```
通过`services`定义了3个服务，分别是db、web、dbstore，db即mysql数据库，web为tomcat，dbstore用来存储数据库数据,image属性定义了容器所使用的镜像，通过volumes_from定义挂载的Data Container。此外在hibernate的数据库连接url应该使用`jdbc:mysql://db:3306/es`来配置db为数据库容器主机名，es为具体的数据库，当全部配置完成后即可通过`docker-compose up`来自动创建以及运行容器。

若需要对数据库文件进行备份可以这样操作：

`docker run --rm --volumes-from dbstore -v $(pwd):/backup ubuntu tar cvf /backup/backup.tar /var/lib/mysql`
将会打包成tar文件，存储在执行命令的当前目录下。

需要恢复数据库则执行：

`docker run --rm --volumes-from dbstore2 -v $(pwd):/backup ubuntu bash -c "cd /dbdata && tar xvf /backup/backup.tar --strip 1"`

Docker Compose的功能很强大，这里只是最简单的使用，Oops!刚接触Docker中，还需要继续摸索



转载请注明出处，谢谢。

