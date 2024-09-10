## 常见命令
  ```
  docker ps #查看正在运行的所有容器
  docker ps -a #查看所有容器 包括已停止的 
  docker exec -it ${容器id 或者 容器名} /bin/bash #进入指定容器
  docker images | grep adminconsole # 查询镜像
  docker ps --format "table {{.ID}}\t{{.Image}}\t{{.Ports}}\t{{.Status}}\t{{.Names}}" #清爽输出
  docker logs -f ${容器名} #查看容器日志
  exit #退出容器
  docker stats #查看容器CPU、内存等资源使用情况
  docker inspect <container_id> #资源配置 + 使用情况
  ```

- 配置环境变量
  ```shell
  # 运行容器时通过-e选项来设置环境变量
  docker run ... -e "SPRING_PROFILES_ACTIVE=dev" ...
  ```

- 容器端口映射宿主机端口
  ```shell
  # 宿主机端口:容器端口
  docker run -d -p 8080:8080 ...
  ```

- 容器运行管理
  ```
  docker rm 删除容器
  docker run 创建新容器并运行进程
  docker stop 将现存容器中的进程关闭，不删除容器
  docker start 将现存的容器中的进程拉起来
  ```

### 解析思考

```
在一台机器上部署多个项目，为什么使用docker更好？
使用虚机部署，不同项目的函数库、配置、依赖的组件可能会有冲突，例如项目A需要nodeVersion<18，项目B需要nodeVersion>18
```

```
docker是什么？ 是一个应用容器引擎
作用是什么？ 开发者打包他们的应用以及依赖包到一个轻量级、可移植的容器，这个容器可以部署到任何linux系统上
那么优点是？ 
传统部署方式：需要考虑部署环境的Linux系统架构，选择对应的组件安装包、安装包的依赖、处理可能会面临的不同的报错、一台机器上部署多个项目如果是虚机部署，则会有多个操作系统，耗费大量空间，而且要多次安装环境
docker部署方式：不需要考虑linux架构，将应用以及依赖包打包成镜像，即可在任何Linux机器上部署，镜像之间相互隔离，不会有冲突
```

```
既然容器之间相互隔离，那么外部怎么没被隔离呢，怎么访问呢？
宿主机ip:192.168.0.1 容器ip：192.168.0.2
启动容器时做端口映射
-p 小写p表示docker会选择一个具体的宿主机端口映射到容器内部开放的网络端口上。
-P 大写P表示docker会随机选择一个宿主机端口映射到容器内部开放的网络端口上。
docker run -ti -d --name my-nginx -p 8088:80 docker.io/nginx
docker run -ti -d --name my-nginx2 -P docker.io/nginx
这样外部就可以通过宿主机ip:8088 访问到容器ip:80了
```
### Docker的运行流程

### Docker命令解读
> -e 代表环境变量，需要特殊说明，不同的镜像需要不同的环境变量
>
> 如何知道这个镜像需要哪些环境变量呢？ 去dockerhub找官方文档
>
> hub.docker.com

### 数据卷

> 作用：为宿主机和容器之间的目录映射搭建一个桥梁

```shell
-v ${数据卷名称}:${容器中目录}
-v ${宿主机目录}:${容器中目录}
#宿主机默认映射目录 /var/lib/docker/volumes/${数据卷名称}/_data
```

### 自定义镜像

#### 镜像的本质

> 就是一个个压缩包，合并到一起，根据依赖关系视为层
>
> 底层常用基础镜像，从公共仓拉取即可（例如把ubuntu中java所需的函数库挑出来做成一个镜像，不需要整个ubuntu了）

#### 构建镜像

> Dockerfile作用：设置环境变量，拷贝本地文件到镜像中，指定基础镜像，编排安装过程，设置容器入口

#### 写好Dockerfile后，如何使用它构建一个镜像？

> 使用docker build -t ${镜像名}:${镜像版本} ${Dockerfile路径}

### 容器之间互联

> 容器之间可以通过ip访问
>
> 缺陷：某一个容器重启之后ip地址可能发生变化
>
> 优化：能不能使用容器名互联？ 可以 -> 自定义网络

```shell
docker network creat testNetwork #创建一个网络组
docker run ... --network testNetwork #容器运行时加入此网络组
ping ${容器名} #进入网络组中的一个容器，通过容器名Ping另一个容器
#yml配置文件、nginx配置文件中也可直接用容器名代替ip地址
原理：
创建自定义网络是，Docker会自动为该网络配置一个DNS服务器，所有链接到这个网络得到容器都会自动注册这个DNS服务器
容器启动时，会向这个DNS服务器注册自己的名称和ip，其他容器在进行域名解析时就可以通过容器名找到ip了
```

### 使用Docker部署后端服务

> 打jar包、写Dockerfile文件 -> docker build构建镜像 -> docker run创建容器启动镜像(端口映射、网络组)

### 使用Docker部署前端服务(容器中使用nginx)

> 前端镜像中包括基础镜像nginx，使用下图中的nginx.conf挂载并代替基础镜像中自带的nginx.conf配置文件
>
> 基础镜像nginx会在容器启动后自动拉起nginx，不用为nginx再设置入口，即容器启动后下图中的nginx.conf立即生效
>
> 容器中的nginx会根据配置文件去监听端口，只需要将监听的端口映射到宿主机，即可直接通过宿主机访问容器中的静态资源
>
> 准备好nginx.conf、静态资源，写Dockerfile文件(基础镜像包括nginx) -> docker build构建镜像 -> docker run创建容器启动镜像(端口映射、网络组)
>
> 访问路径：宿主机ip:端口 -> 容器中的nginx 找到对应的容器端口 找到配置的端口对应的静态资源 -> 访问到容器内的静态ziyuan


