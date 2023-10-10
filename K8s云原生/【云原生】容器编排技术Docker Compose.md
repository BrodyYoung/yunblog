
## 为什么需要Docker Compose
Docker帮助我们解决服务的打包安装的问题，随着而来的问题就是服务过多的带来如下问题：

1. 多次使用Dockerfile Build Image或者DockerHub拉取Image;
2. 需要创建多个Container，多次编写启动命令；
3. Container互相依赖的如何进行管理和编排；

当我们服务数量增多的时候，上面三个问题就会更加的被放大，如果这三个问题不解决，其实从虚拟机到容器化除了机器减少一些浪费以外，好像没有更多的变化。Docker有没有什么好的方法，可以让我们通过一个配置就搞定容器编排和运行呢?这个时候Docker Compose就站出来了。

Docker Compose可以做到以下几点：

1. 提供工具用于定义和运行多个docker容器应用；
1. 使用yaml文件来配置应用服务(docker-compse.yml)；
1. 可以通过一个简单的命令docker-compse up可以按照依赖关系启动所有服务；
1. 可以通过一个简单的命令docker-compose down停止所有服务；
1. 当一个服务需要的时候，可以很简单地通过--scale进行扩容；

Docker Compose有以下特征:

1. 更高的可移植性，Docker Compose仅需一个docker-compse up可以完成按照依赖关系启动所有服务，然后使用docker-compose down轻松将其拆解。帮助我们更轻松地部署复杂的应用程序；
1. 单个主机上的多个隔离环境，Compose可以使用项目名称将环境彼此隔离，这带可以在一台计算机上运行同一环境的多个副本，它可以防止不同的项目和服务相互干扰；
## Docker Compose介绍
1. Docker Compose是一个工具，用于定义和运行多容器应用程序的工具；

1. Docker Compose通过yml文件定义多容器的docker应用；

1. Docker Compose通过一条命令根据yml文件的定义去创建或管理多容器；
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/ec7d4d07138f447a969c0c23559393f6.png)

Docker Compose 是用来做Docker 的多容器控制，是一个用来把 Docker 自动化的东西。有了 Docker Compose 你可以把所有繁复的 Docker 操作全都一条命令，自动化的完成。

## Docker Compose安装
Docker Compose安装的最新的版本1.29.2，对于Mac和Windows安装好Docker以后，就已经安装好Docker Compose，不需要手动安装，这里的安装方式是基于Linux的Cnetos的，大家也可以参考官方网站去安装，

具体步骤如下:

1. 下载 Docker Compose 二进制文件，版本1.29.2是目前最新最稳定的版本，要下载旧版本的大家可以更改版本号，可以参考github的版本号进行选择；

```bash
sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```

2. 对二进制文件应用可执行权限;

```bash
sudo chmod +x /usr/local/bin/docker-compose
```

3. 安装以后通过docker-compose --version命令检查是否安装成功；

![在这里插入图片描述](https://img-blog.csdnimg.cn/bf4824af321d459d9161120a05a4cd28.png)

## Docker Compose版本介绍
Docker Compose版本与引擎的对应关系如下，可以看到中间主要有两个版本2和版本3两种格式，目前大家使用比较多也就是这两种，对于这两个版本的差别给大家介绍一下:

1. v3 版本不支持 volume_from 、extends、group_add等属性;
2. cpu 和 内存属性的设置移到了 deploy 中;
3. v3 版本支持 Docker Swarm，而 v2 版本不支持;

注意：官方目前在 1.20.0 引入了一个新--compatibility标志，帮助开发人员轻松的过渡到v3，目前还有些问题官方还不建议直接使用到生产，建议大家直接上手v3版本。

![在这里插入图片描述](https://img-blog.csdnimg.cn/76d99c266c5b44849bb6f7d471dbedb6.png)

## Docker Compose基本命令介绍
Docker Compose命令基本上和Docker相差不多，主要就是对Docker Compose生命周期控制、日志格式等相关命令，可以通过docker-compose --help进行帮助。

```bash
#构建建启动nignx容器
docker-compose up -d nginx                     

#进入nginx容器中
docker-compose exec nginx bash            

#将会停止UP命令启动的容器，并删除容器
docker-compose down                             

#显示所有容器
docker-compose ps                                   

#重新启动nginx容器
docker-compose restart nginx                   

#构建镜像
docker-compose build nginx      

#不带缓存的构建
docker-compose build --no-cache nginx 

#查看nginx的日志
docker-compose logs  nginx                      

#查看nginx的实时日志
docker-compose logs -f nginx                   

#验证（docker-compose.yml）文件配置，
#当配置正确时，不输出任何内容，当文件配置错误，输出错误信息
docker-compose config  -q                        

#以json的形式输出nginx的docker日志
docker-compose events --json nginx       

#暂停nignx容器
docker-compose pause nginx                 

#恢复ningx容器
docker-compose unpause nginx             

#删除容器
docker-compose rm nginx                       

#停止nignx容器
docker-compose stop nginx                    

#启动nignx容器
docker-compose start nginx                 
```


## Docker Compose实战
我们构建一个如下的应用，通过Nginx转发给后端的两个Java应用;

![在这里插入图片描述](https://img-blog.csdnimg.cn/6429492ccdf0462691c04fb58c5ea94e.png)

1. 新建Spring Boot应用，增加一个HelloController，编写一个hello方法，返回请求的端口和IP；

```java
/**
 * hello
 *
 * @author wangtongzhou
 * @since 2021-07-25 09:43
 */
@RestController
public class HelloController {

    @GetMapping("/hello")
    public String hello(HttpServletRequest req) throws UnknownHostException {
        return "hello";
    }

}
```

2. 指定Spring Boot的启动入口;


```java
  <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <!-- 指定该Main Class为全局的唯一入口 -->
                    <mainClass>cn.wheel.getway.WheelGetWay</mainClass>
                </configuration>
                <executions>
                    <execution>
                        <goals>
                            <!--可以把依赖的包都打包到生成的Jar包中-->
                            <goal>repackage</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
```

3. 打包Spring Boot应用；

```bash
mvn package
```

4. 上传文件到Linux服务器/usr/local/docker-compose-demo的目录；
5. 在/usr/local/docker-compose-demo的目录编辑Dockerfile；

```yaml
#指定基础镜像
FROM java:8
LABEL name="docker-compose-demo" version="1.0" author="wtz"
COPY ./getway-1.0-SNAPSHOT.jar ./docker-compose-demo.jar
#启动参数
CMD ["java","-jar","docker-compose-demo.jar"]
```

6. 编辑docker-compose.yml文件；

```yaml
version: '3.0'

networks:
  docker-compose-demo-net:
    driver: bridge
    ipam:
      config:
        - subnet: 192.168.1.0/24
          gateway: 192.168.1.1


services:
  docker-compose-demo01:
    build:
      #构建的地址
      context: /usr/local/docker-compose-demo
      dockerfile: Dockerfile
    image: docker-compose-demo
    container_name: docker-compose-demo01
    #选择网络
    networks:
      - docker-compose-demo-net
    #选择端口
    ports:
      - 8081:8080/tcp
    restart: always

  docker-compose-demo02:
    build:
      #构建的地址
      context: /usr/local/docker-compose-demo
      dockerfile: Dockerfile
    image: docker-compose-demo
    container_name: docker-compose-demo02
    #选择网络
    networks:
      - docker-compose-demo-net
    #选择端口
    ports:
      - 8082:8080/tcp
    restart: always

  nginx:
    image: nginx:latest
    container_name: nginx-demo
    networks:
      - docker-compose-demo-net
    ports:
      - 80:80/tcp
    restart: always
    volumes:
      - /usr/local/docker-compose-demo/nginx.conf:/etc/nginx/nginx.conf:rw


volumes:
  docker-compose-demo-volume: {}
```

7. 编写nginx.conf，实现负载均衡到每个应用，这里通过容器名称访问，因此不需要管每个容器的ip是多少，这个也是自定义网络的好处；

```yaml
user nginx;
worker_processes  1;
events {
    worker_connections  1024;
}
http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;

server {
    listen 80;
    location / {
     proxy_pass http://docker-compose-demo;
     proxy_set_header  Host $host;
	     proxy_set_header  X-real-ip $remote_addr;
	     proxy_set_header  X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}

upstream docker-compose-demo{
   server docker-compose-demo01:8080;
   server docker-compose-demo02:8080;
}
include /etc/nginx/conf.d/*.conf;


server {
    listen 80;
    location / {
     proxy_pass http://docker-compose-demo;
     proxy_set_header  Host $host;
	     proxy_set_header  X-real-ip $remote_addr;
	     proxy_set_header  X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}

upstream docker-compose-demo{
   server docker-compose-demo01:8080;
   server docker-compose-demo02:8080;
}
include /etc/nginx/conf.d/*.conf;
}
```

8. 查看/usr/local/docker-compose-demo目录，有以下确保有以下四个文件；
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/a14c5e585e584dbe9af063887f1cbe78.png)

9. 检查docker-compose.yml的语法是否正确，如果不发生报错，说明语法没有发生错误;
   docker-compose config
10. 启动docker-compose.yml定义的服务；

```bash
docker-compose up
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/b08bb4529c8a4a54860e7e5c5a04d119.png)

11. 验证服务是否正确；

```bash
#查看宿主机ip
ip add

#访问对应的服务
curl http://172.21.122.231/hello
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/94be492f3217497ca4e97d5c66584946.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/c90bc4e3873f411482d59e1780ea4606.png)


## Docker Compose Yml文件介绍
**version**
指定使用的版本；

**Services**
每个Service代表一个Container，与Docker一样，Container可以是从DockerHub中拉取到的镜像，也可以是本地Dockerfile Build的镜像。

**image**
标明image的ID，这个image ID可以是本地也可以是远程的，如果本地不存在，Docker Compose会尝试pull下来;


```yaml
image: ubuntu
```

**build**
该参数指定Dockerfile文件的路径，Docker Compose会通过Dockerfile构建并生成镜像，然后使用该镜像;

```yaml
build:
  #构建的地址
  context: /usr/local/docker-compose-demo
  dockerfile: Dockerfile
```

**ports**
暴露端口，指定宿主机到容器的端口映射，或者只指定容器的端口，则表示映射到主机上的随机端口，一般采用主机:容器的形式来映射端口；

```yaml
#暴露端口
ports:
  - 8081:8080/tcp
```

**expose**
暴露端口，但不需要建立与宿主机的映射，只是会向链接的服务提供；

**environment**
加入环境变量，可以使用数组或者字典，只有一个key的环境变量可以在运行compose的机器上找到对应的值；

**env_file**
从一个文件中引入环境变量，该文件可以是一个单独的值或者一个列表，如果同时定义了environment，则environment中的环境变量会重写这些值；

**depends_on**
定义当前服务启动时，依赖的服务，当前服务会在依赖的服务启动后启动;

```yaml
depends_on: 
  - docker-compose-demo02
  - docker-compose-demo01
```

**deploy**
该配置项在version 3里才引入，用于指定服务部署和运行时相关的参数；

**replicas**
指定副本数；

```yaml
version: '3.4'
services:
  worker:
    image: nginx:latest
    deploy:
      replicas: 6
```

**restart_policy**
指定重启策略;

```yaml
version: "3.4"
services:
  redis:
    image: redis:latest
    deploy:
      restart_policy:
        condition: on-failure   #重启条件：on-failure, none, any
        delay: 5s   # 等待多长时间尝试重启
        max_attempts: 3 #尝试的次数
        window: 120s    # 在决定重启是否成功之前等待多长时间
```

**update_config**
定义更新服务的方式，常用于滚动更新;

```yaml
version: '3.4'
services:
  vote:
    image: docker-compose-demo
    depends_on:
      - redis
    deploy:
      replicas: 2
      update_config:
        parallelism: 2  # 一次更新2个容器
        delay: 10s  # 开始下一组更新之前，等待的时间
        failure_action：pause  # 如果更新失败，执行的动作：continue, rollback, pause，默认为pause
        max_failure_ratio： 20 # 在更新过程中容忍的失败率
        order: stop-first   # 更新时的操作顺序，停止优先（stop-first，先停止旧容器，再启动新容器）还是开始优先（start-first，先启动新容器，再停止旧容器），默认为停止优先，从version 3.4才引入该配置项
```

**resources**
限制服务资源；

```yaml
version: '3.4'
services:
  redis:
    image: redis:alpine
    deploy:
      resources:
        #限制CPU的使用率为50%内存50M
        limits:
          cpus: '0.50'
          memory: 50M
        #始终保持25%的使用率内存20M
        reservations:
          cpus: '0.25'
          memory: 20M
```

**healthcheck**
执行健康检查;

```yaml
healthcheck:
  test: ["CMD", "curl", "-f", "http://localhost"]   # 用于健康检查的指令
  interval: 1m30s   # 间隔时间
  timeout: 10s  # 超时时间
  retries: 3    # 重试次数
  start_period: 40s # 启动多久后开始检查
```

**restart**
重启策略;

```yaml
#默认的重启策略，在任何情况下都不会重启容器
restart: "no"
#容器总是重新启动
restart: always
#退出代码指示失败错误，则该策略会重新启动容器
restart: on-failure
#重新启动容器，除非容器停止
restart: unless-stopped
```

networks
网络类型，可指定容器运行的网络类型;

```yaml
#指定对应的网络
networks:
  - docker-compose-demo-net
  
  
networks:
  docker-compose-demo-net:
    driver: bridge
    ipam:
      config:
        - subnet: 192.168.1.0/24
          gateway: 192.168.1.1
```

ipv4_address, ipv6_address
加入网络时，为此服务指定容器的静态 IP 地址；

```yaml
version: "3.9"

services:
  app:
    image: nginx:alpine
    networks:
      app_net:
        ipv4_address: 172.16.238.10
        ipv6_address: 2001:3984:3989::10

networks:
  app_net:
    ipam:
      driver: default
      config:
        - subnet: "172.16.238.0/24"
        - subnet: "2001:3984:3989::/64"
```

**Networks**
网络决定了服务之间以及服务和外界之间如何去通信，在执行docker-compose up的时候，docker会默认创建一个默认网络，创建的服务也会默认的使用这个默认网络。服务和服务之间，可以使用服务的名字进行通信，也可以自己创建网络，并将服务加入到这个网络之中，这样服务之间可以相互通信，而外界不能够与这个网络中的服务通信，可以保持隔离性。

**Volumes**
挂载主机路径或命名卷，指定为服务的子选项。可以将主机路径挂载为单个服务定义的一部分，无需在顶级volume中定义。如果想在多个服务中重用一个卷，则在顶级volumes key 中定义一个命名卷，将命名卷与服务一起使用。

## 总结
　　

Docker Compose 的整体使用步骤还是比较简单的，三个步骤为：

- 使用 Dockerfile 文件定义应用程序的环境；
- 使用 docker-compose.yml 文件定义构成应用程序的服务，这样它们可以在隔离环境中一起运行；
- 最后，执行 docker-compose up 命令来创建并启动所有服务。

虽然 docker-compose.yml 文件详解和Compose 常用命令这两大块的内容比较多，但是如果要快速入门使用 Compose，其实只需要了解其中部分内容即可。后期大家可在项目生产环境中根据自身情况再进一步深入学习即可。
