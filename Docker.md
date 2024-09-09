## 常见命令
docker ps #查看正在运行的所有容器
docker ps -a #查看所有容器 包括已停止的 
docker exec -it ${容器id 或者 容器名} /bin/bash #进入指定容器
docker images | grep adminconsole # 查询镜像
docker ps --format "table {{.ID}}\t{{.Image}}\t{{.Ports}}\t{{.Status}}\t{{.Names}}" #清爽输出
docker logs -f ${容器名} #查看容器日志
exit #退出容器

- 配置环境变量
  // 运行容器时通过-e选项来设置环境变量
  docker run ... -e "SPRING_PROFILES_ACTIVE=dev" ...

- 容器端口映射宿主机端口
  // 宿主机端口:容器端口
  docker run -d -p 8080:8080 ...

- 容器运行管理
  ```docker rm 删除容器
  docker run 创建新容器并运行进程
  docker stop 将现存容器中的进程关闭，不删除容器
  docker start 将现存的容器中的进程拉起来```

