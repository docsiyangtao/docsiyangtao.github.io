## 参考

官方文档地址:https://www.docker.com/get-started

中文参考手册:https://docker_practice.gitee.io/zh-cn/

远程仓库：https://hub.docker.com/

## 安装

```sh
# 卸载旧版本
sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine

# 设置yum源
sudo yum install -y yum-utils
sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo

# 安装最新版
sudo yum install docker-ce docker-ce-cli containerd.io

# 查看版本列表
yum list docker-ce --showduplicates | sort -r

# 安装指定版本
sudo yum install docker-ce-<VERSION_STRING> docker-ce-cli-<VERSION_STRING> containerd.io
```

## 生命周期理解

![image-20220425141826314](https://picgo-1304850123.cos.ap-guangzhou.myqcloud.com/image-20220425141826314.png)

## 镜像

```sh
# 镜像搜索
docker search [名称]

# 获取镜像
docker pull [选项] [Docker Registry 地址[:端口号]/]仓库名[:标签]

# 查看镜像列表
docker images [选项] [名称]
	-a：列出所有镜像（包含中间映像层）
	-q：只显示镜像id
	名称：只搜索相关的images，相当于过滤条件
	-f：其他过滤

# 查看docker占用空间
docker system df

# 查看虚悬(旧)镜像
docker images -f dangling=true

# 虚悬镜像删除
docker image prune

# 删除镜像
docker rmi [选项] 镜像名:tag | 镜像id
	-f：强制删除
	
# 构建镜像
docker build [选项] PATH
	-f：指定Dockerfile路径
	-t：指定镜像名及标签

# 镜像导入
docker load -i 名称.tar

# 镜像打包
docker save 镜像名 -o 名称.tar

# 镜像推送
docker push USERNAME/IMAGES_NAME
```

## 容器基础

```sh
# 运行容器
docker run [选项] 镜像名:tag|镜像id
	-f：--name 起别名
	-d：后台启动
	-p：端口映射（【宿主机端口】:【容器内部端口】）
	-v [:ro]：数据卷映射（【宿主机路径】:【容器内路径】），ro容器只读不写，路径不存在则自动创建
	--mount type=bind,source=宿主机路径(绝对),target=容器路径[,readonly]：挂载目录，路径不存在报错
	--network [网桥名]：指定网桥

# 启动/关闭/重启/强杀
docker start 容器名或id
docker stop 容器名或id
docker restart 容器名或id
docker kill 容器名或id

# 删除容器
 docker [container] rm [选项] 容器名或id
 docker container prune   # 删除所有终止状态的容器
 	-f：强制

# 组合命令
docker rm -f $(docker ps -aq)

# 容器日志
docker log [选项] 容器名或id
	-f：实时展示
	-tf：加入时间戳的实时展示
	--tail：只显示尾部日志

# 容器内部进程
docker top 容器名或id

# 容器细节
docker inspect 容器名或id

# 进入容器
docker exec [选项] 容器名或id bash
	-i：交互模式
	-t：分配一个伪终端

# 复制文件
docker cp 文件|目录 容器名或id:容器内资源路径
docker cp 容器名或id:容器内资源路径 主机目录

# 容器打包
docker commit -m "描述信息" -a "作者信息" （容器id或者名称） 打包的镜像名称:标签
```

## 网络

```sh
# 查看网络信息
docker network ls

# 新建网桥
docker network create [选项] 网桥名称
	-d：网络类型，有 bridge overlay
	
# 查看网桥
docker inspect 网桥名

# 删除网桥
docker network rm 网桥名称
```

## Dockerfile

| **保留字**     | **作用**                                                     |
| -------------- | ------------------------------------------------------------ |
| **FROM**       | **当前镜像是基于哪个镜像的 `第一个指令必须是FROM`**          |
| **MAINTAINER** | **镜像维护者的姓名和邮箱地址**                               |
| **RUN**        | **构建镜像时需要运行的指令**                                 |
| **EXPOSE**     | **当前容器对外暴露出的端口号**                               |
| **WORKDIR**    | **指定在创建容器后，终端默认登录进来的工作目录，一个落脚点** |
| **ENV**        | **用来在构建镜像过程中设置环境变量**                         |
| **ADD**        | **将宿主机目录下的文件拷贝进镜像且ADD命令会自动处理URL和解压tar包** |
| **COPY**       | **类似于ADD，拷贝文件和目录到镜像中<br/>将从构建上下文目录中<原路径>的文件/目录复制到新的一层的镜像内的<目标路径>位置** |
| **VOLUME**     | **用来定义容器运行时可以挂载到宿主机的目录，用于数据保存和持久化工作** |
| **CMD**        | **指定一个容器启动时要运行的命令<br/>Dockerfile中可以有多个CMD指令，但只有最后一个生效，CMD会被docker run之后的参数替换** |
| **ENTRYPOINT** | **指定一个容器启动时要运行的命令<br/>ENTRYPOINT的目的和CMD一样，都是在指定容器启动程序及其参数** |

## Docker Compose

```yaml
version: '3'
networks: 		# 定义网桥
  blog_bridge: 
services:
  nginx:
    image: nginx:latest
    ports:
      - "80:8080"
    volumes:
      - /blog/nginx/html:/usr/share/nginx/html
      - /blog/nginx/nginx.conf:/etc/nginx/nginx.conf
    privileged: true  # Nginx内部权限
    environment:
      - SET_CONTAINER_TIMEZONE=true
      - CONTAINER_TIMEZONE=Asia/Shanghai
  mysql:
    image: mysql:5.7.27
    ports:
      - "3306:3306"
    environment:
      - MYSQL_ROOT_PASSWORD=root
      - SET_CONTAINER_TIMEZONE=true
      - CONTAINER_TIMEZONE=Asia/Shanghai
  redis:
    image: redis:latest
    ports:
      - "6379:6379"
    volumes:
      - /blog/redis/redis.conf:/usr/local/etc/redis/redis.conf
    command:
      redis-server /usr/local/etc/redis/redis.conf
    privileged: true
    environment:
      - SET_CONTAINER_TIMEZONE=true
      - CONTAINER_TIMEZONE=Asia/Shanghai
  rabbitmq:
    image: daocloud.io/library/rabbitmq:3.8.7
    restart: no
    ports:
      - "5672:5672"
      - "15672:15672"
    volumes:
      - /blog/data:/var/lib/rabbitmq/
    environment:
      - SET_CONTAINER_TIMEZONE=true
      - CONTAINER_TIMEZONE=Asia/Shanghai
  blog:
    build: 
      context: .				# Dockerfile所在目录
      dockerfile: Dockerfile	# 构建文件
    ports:
      - "8888:8888"
    depends_on:					# 依赖于哪些容器
      - mysql
      - redis
      - rabbitmq
    environment:
      - SET_CONTAINER_TIMEZONE=true
      - CONTAINER_TIMEZONE=Asia/Shanghai
```

