# 系统架构介绍
### 一、该项目对外提供即使通讯服务（单聊和群聊）的同时，可以实现的自定义多端(web、ios、android )消息同步、离线消息缓存、在线状态变更、多系统的接入、智能对话（对接通义千问大模型）等功能。
#### 1、业务系统：开启 Netty 服务（实现自定义协议）实现单聊/群聊的消息发送、即时通讯业务其他的业务逻辑。
#### 2、第三方业务系统：实现三业务系统的登陆和自定义的业务功能。
#### 3、前台服务：实现即时通讯的用户交互
### 二、技术栈
#### Netty：开启 netty 服务端，实现 tcp 连接，并实现自定义协议完成客户端和服务端的实时通讯。
#### dubbo： 实现 tcp微服务 和 业务微服务之间的接口调用
#### rabbitmq： 完成消息的分发
#### mysql： 业务数据存储
#### redis： 离线消息存储和热点数据存储
#### naocs： 实现 tcp 微服务服务注册和发现、dubbo注册中心
#### Spring Boot: 微服务框架
### 三、后台服务模块介绍
#### im-codec：netty 自定义协议相关的实现
#### im-common：公共实体类和工具类
#### im-message-store：消息持久化微服务
#### im-service：IM服务业务微服务-实现业务逻辑
#### im-tcp： netty服务-实现和客户端的连接-消息交互模块
### 四、业务优化
#### 消息的即时性：即如何保证发送端的消息及时响应给接收端，基本思路是减少消息发送到消息接受这一业务逻辑的功能单一行：1、使用线程池实现并发（引起消息的有序性问题） 2、消息的存储使用异步实现，避免入库的耗时操作。 3、消息的发送合法性校验操作提前到 tcp 层，使用 dubbo 远程调用 servcie 层的业务方法，避免非法发送请求占用不必要的资源
#### 消息的有序性：消息的发送使用了线程并发和异步存储，则无法保证消息的有序性。需要引入 sequence 该字段表示 - 实现使用 redis 自增机制获取 seq 的值。
#### 消息的可靠性：要保证消息一定发送到接收端：1、添加 ack 表示，消息到达服务端和消息到达接受端需要给客户端回复 ack 标识消息的发送状态 2、客户端提供容错机制，没有接收到发送成功的 ack 之间需要提供重发机制（引发幂等性问题）
#### 消息的幂等性：因为消息可能存在重发的机制，客户端接收消息只能有一次，所以服务需要确保相同的消息只处理一次。使用 redis 保存已经处理过的消息的 key - 设置超时事件，如果已经存在缓存说明已经处理过了，不再处理重复的消息。

![系统架构介绍](https://github.com/user-attachments/assets/37f8f29a-a389-4578-86f8-c61b3915f432)
# 数据流转时序图
![数据流转时序图](https://github.com/user-attachments/assets/ed849f9d-4797-40e5-9248-a5b2fe298a1f)

# 功能介绍
### 1、登录页面
![登录页面](https://github.com/user-attachments/assets/b87d0d1c-2809-415f-9f7d-4a35a6c29025)
### 2、聊天首页
![聊天页面](https://github.com/user-attachments/assets/e5325ee7-3309-4bd1-ab1c-44e3048bc69f)
### 3、智能问答
![智能问答](https://github.com/user-attachments/assets/61fda008-da21-44fd-b8be-7502724a50c9)
### 4、添加好友页面
![image](https://github.com/user-attachments/assets/a21ba3c0-d3a9-4013-a460-a1fa9d881bbf)
### 5、好友详情页面
![image](https://github.com/user-attachments/assets/9c0ae696-ea16-4d91-9eaa-cd53879f0175)
### 6、单聊展示
![单聊展示](https://github.com/user-attachments/assets/1b9ef42c-b37b-49ba-91bc-3a2d9a6e550d)
### 7、群聊展示
![群聊展示](https://github.com/user-attachments/assets/25e011db-b2ca-4bc5-b444-c7e1d81afbf6)
### 8、登出页面
![登出页面](https://github.com/user-attachments/assets/aef6d0c7-ad26-402a-a56d-31a4fb051cc4)



# IM-SERVICE 即时通讯系统

基于 Spring Cloud、Netty 的分布式即时通讯系统。

## 系统架构

系统包含以下主要服务：

- `im-service`: 核心业务服务
- `im-message-store`: 消息存储服务
- `im-tcp`: TCP 长连接服务

## 环境要求

- JDK 8+
- Maven 3.6+
- Docker 20.10+
- Docker Compose 2.0+

## 快速开始

### 1. 项目打包

```bash
# 进入项目根目录
cd IM-SERVICE

# 打包所有服务（跳过测试）
mvn clean package -DskipTests
```

### 2. 构建微服务镜像

#### 2.1 构建基础镜像

首先需要构建一个包含基础环境的 Docker 镜像，所有服务都将基于此镜像：

```bash
# 构建基础镜像
docker build -t im-base:latest -f Dockerfile.base .
```

#### 2.2 构建各服务镜像

可以使用以下命令分别构建各个服务的镜像：

```bash
# 构建 im-service 镜像
cd im-service
docker build -t im-service:latest .

# 构建 im-message-store 镜像
cd ../im-message-store
docker build -t im-message-store:latest .

# 构建 im-app-business 镜像
cd ../im-app-business
docker build -t im-app-business:latest .

# 构建 im-tcp 镜像
cd ../im-tcp
docker build -t im-tcp:latest .

# 关闭本地 mysql
按下 Win + R 打开运行窗口，输入 services.msc，回车。
在服务列表中找到你的 MySQL 服务（名称可能是 MySQL, MySQL80, MySQL57 等，取决于你的版本）。
```

或者使用 docker-compose 一次性构建所有服务镜像：

```bash
# 构建所有服务镜像
docker-compose build

# 构建单个服务镜像（例如只重新构建 im-service）
docker-compose build im-service
```

#### 2.3 镜像构建说明

各服务的 Dockerfile 主要包含以下步骤：

1. im-service Dockerfile:
```dockerfile
FROM im-base:latest
COPY target/im-service-1.0-SNAPSHOT.jar app.jar
EXPOSE 8000
HEALTHCHECK --interval=30s --timeout=10s --retries=3 CMD curl -f http://localhost:8000/actuator/health || exit 1
ENTRYPOINT ["java", "-jar", "app.jar"]
```

2. im-message-store Dockerfile:
```dockerfile
FROM im-base:latest
COPY target/im-message-store-1.0-SNAPSHOT.jar app.jar
EXPOSE 8001
HEALTHCHECK --interval=30s --timeout=10s --retries=3 CMD curl -f http://localhost:8001/actuator/health || exit 1
ENTRYPOINT ["java", "-jar", "app.jar"]
```

3. im-tcp Dockerfile:
```dockerfile
FROM im-base:latest
COPY target/im-tcp-1.0-SNAPSHOT.jar app.jar
EXPOSE 8002 9000
HEALTHCHECK --interval=30s --timeout=10s --retries=3 CMD curl -f http://localhost:8002/actuator/health || exit 1
ENTRYPOINT ["java", "-jar", "app.jar"]
```

#### 2.4 镜像管理命令

```bash
# 查看构建的镜像
docker images | grep im-

# 删除特定镜像
docker rmi [image_name]:[tag]

# 清理所有未使用的镜像
docker image prune -a

# 查看镜像构建历史
docker history [image_name]:[tag]

# 保存镜像到文件
docker save -o [filename].tar [image_name]:[tag]

# 从文件加载镜像
docker load -i [filename].tar
```

### 3. 构建基础设施

项目依赖以下基础服务：

- MySQL 8.0: 数据持久化
- Redis 6.2: 缓存服务
- RabbitMQ 3.x: 消息队列
- Nacos 2.2.3: 服务注册与发现

这些服务已在 docker-compose.yml 中配置好，无需单独安装。

### 4. 初始化配置

```bash
# 创建 RabbitMQ 初始化脚本
cat > init-rabbitmq.sh << 'EOF'
#!/bin/sh
until rabbitmqctl wait --timeout 60 /var/lib/rabbitmq/mnesia/rabbit@$HOSTNAME.pid; do
    echo "Waiting for RabbitMQ to start..."
    sleep 2
done
rabbitmqctl add_vhost im
rabbitmqctl set_permissions -p im guest ".*" ".*" ".*"
echo "RabbitMQ has been initialized!"
EOF

# 添加执行权限
chmod +x init-rabbitmq.sh
```

### 5. 启动服务

```bash
# 启动所有服务
docker-compose up -d

# 查看服务启动状态
docker-compose ps

# 查看服务日志
docker-compose logs -f [service_name]

# 停止并删除现有容器和网络
docker-compose down -v
```

服务启动顺序：
1. 基础设施服务（MySQL, Redis, RabbitMQ, Nacos）
2. im-service
3. im-message-store
4. im-tcp
![image](https://github.com/user-attachments/assets/b40a4ec0-25ee-496a-a851-a93175644156)


### 6. 服务访问

服务启动后，可通过以下地址访问：

- Nacos 控制台: http://localhost:8848/nacos
  - 用户名: nacos
  - 密码: nacos
- RabbitMQ 管理界面: http://localhost:15672
  - 用户名: guest
  - 密码: guest
- 业务服务接口:
  - im-service: http://localhost:8000
  - im-message-store: http://localhost:8001
  - im-tcp: 
    - HTTP: http://localhost:8002
    - TCP: localhost:9000

### 7. 常用运维命令

```bash
# 停止所有服务
docker-compose down

# 重新构建并启动特定服务
docker-compose up -d --build [service_name]

# 查看服务日志
docker-compose logs -f [service_name]

# 进入容器内部
docker-compose exec [service_name] sh

# 查看服务状态
docker-compose ps

# 重启特定服务
docker-compose restart [service_name]
```

### 8. 数据持久化

所有服务的数据都通过 Docker volumes 持久化：

- MySQL 数据: mysql_data
- Redis 数据: redis_data
- RabbitMQ 数据: rabbitmq_data
- Nacos 数据: nacos_data

### 9. 网络配置

所有服务都在 `im-network` 网络下运行，服务间通过服务名互相访问：

- MySQL: im-mysql:3306
- Redis: im-redis:6379
- RabbitMQ: im-rabbitmq:5672
- Nacos: im-nacos:8848

### 10. 故障排查

1. 服务启动失败
   ```bash
   # 查看服务日志
   docker-compose logs -f [service_name]
   ```

2. 网络连接问题
   ```bash
   # 检查网络配置
   docker network inspect im-network
   ```

3. 数据持久化问题
   ```bash
   # 查看数据卷
   docker volume ls
   docker volume inspect [volume_name]
   ```

### 11. 注意事项

1. 首次启动时，确保 Maven 已正确安装并配置
2. 确保 Docker 和 Docker Compose 已正确安装
3. 确保所需端口未被占用
4. 服务启动可能需要一定时间，请耐心等待
5. 如遇到权限问题，请使用管理员权限运行命令

## 技术栈

- Spring Boot
- Spring Cloud
- Netty
- MySQL
- Redis
- RabbitMQ
- Nacos
- Docker
- Docker Compose

## 贡献指南

欢迎提交 Issue 和 Pull Request。

## 许可证

[MIT License](LICENSE)
