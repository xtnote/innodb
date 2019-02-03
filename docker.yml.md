
### yml demo
```yaml
version: '2'
services:
  web:
    image: dockercloud/hello-world
    ports:
      - 8080
    networks:
      - front-tier
      - back-tier

  redis:
    image: redis
    links:
      - web
    networks:
      - back-tier

  lb:
    image: dockercloud/haproxy
    ports:
      - 80:80
    links:
      - web
    networks:
      - front-tier
      - back-tier
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock

networks:
  front-tier:
    driver: bridge
  back-tier:
driver: bridge
```

```yaml
version: "3"
services:

  redis:
    image: redis:alpine
    ports:
      - "6379"
    networks:
      - frontend
    deploy:
      replicas: 2
      update_config:
        parallelism: 2
        delay: 10s
      restart_policy:
        condition: on-failure

  db:
    image: postgres:9.4
    volumes:
      - db-data:/var/lib/postgresql/data
    networks:
      - backend
    deploy:
      placement:
        constraints: [node.role == manager]

  vote:
    image: dockersamples/examplevotingapp_vote:before
    ports:
      - 5000:80
    networks:
      - frontend
    depends_on:
      - redis
    deploy:
      replicas: 2
      update_config:
        parallelism: 2
      restart_policy:
        condition: on-failure

  result:
    image: dockersamples/examplevotingapp_result:before
    ports:
      - 5001:80
    networks:
      - backend
    depends_on:
      - db
    deploy:
      replicas: 1
      update_config:
        parallelism: 2
        delay: 10s
      restart_policy:
        condition: on-failure

  worker:
    image: dockersamples/examplevotingapp_worker
    networks:
      - frontend
      - backend
    deploy:
      mode: replicated
      replicas: 1
      labels: [APP=VOTING]
      restart_policy:
        condition: on-failure
        delay: 10s
        max_attempts: 3
        window: 120s
      placement:
        constraints: [node.role == manager]

  visualizer:
    image: dockersamples/visualizer:stable
    ports:
      - "8080:8080"
    stop_grace_period: 1m30s
    volumes:
      - "/var/run/docker.sock:/var/run/docker.sock"
    deploy:
      placement:
        constraints: [node.role == manager]

networks:
  frontend:
  backend:

volumes:
  db-data:
```

### 时间
支持下面的写法
```
2.5s
10s
1m30s
2h32m
5h34m56s
```
支持的单位有us, ms, s, m, h

### 容量
支持下面的写法
```
2b
1024kb
2048k
300m
1gb
```
支持的单位有b, k, m, g, kb, mb, gb

### 版本声明  
`version: '3'`  
---

# services下的标签  
在services标签下的第二级标签是容器名，这个名字是用户自己自定义，它就是服务名称。
---

### services.container.image
```yaml
services:
  web:
    image: hello-world
```
```yaml
image: redis
image: ubuntu:14.04
image: tutum/influxdb
image: example-registry.com:4000/postgresql
image: a4bc65fd
```
image指定服务的镜像名称或镜像ID。如果镜像在本地不存在，Compose将会尝试拉取这个镜像。

### services.container.build
基于Dockerfile构建镜像
1. 字符串  
```yaml
services:
  web:
    build: /path/to/build/dir`
    #build: ./dir
```
指向context的目录  
2. 对象  
```yaml
image: webapp:tag
build:
  context: ../
  dockerfile: path/of/Dockerfile
  cache_from:
    - alpine:latest
    - corp/web_app:3.14
  labels:
      com.example.description: "Accounting webapp"
      com.example.department: "Finance"
      com.example.label-with-empty-value: ""
  args:
    - buildno=1
    - password=secret
    - buildno
    - password
  shm_size: '2gb'
  target: prod
```
* context是一个包含dockerfile的目录，或者一个git的url，如果是一个相对目录，它是相对于compose文件的地址
* dockerfile构建用的文件配置
* args中的arg必须提前在dockerfile中声明，然后又compose传入，arg是允许空值的,构建过程可以向它们赋值，
* labels设置生成镜像的元数据
* shm_size设置/dev/shm的大小
* target多阶段构建的目标
* 如果同时指定了image和build两个标签，那么Compose会构建镜像并且把镜像命名为image后面的那个名字。
* YAML 的布尔值（true, false, yes, no, on, off）必须要使用引号引起来（单引号、双引号均可），否则会当成字符串解析。

### services.container.cap_add, services.container.cap_drop
```
cap_add:
  - ALL
cap_drop:
  - NET_ADMIN
  - SYS_ADMIN
```
添加或删除容器的权限，查看权限man 7 capabilities

### services.container.cgroup_parent
`cgroup_parent: m-executor-abcd`  
指定一个容器的父级cgroup。

### services.container.command
覆盖容器启动后默认执行的命令。`command: bundle exec thin -p 3000`  
也可以写成类似 Dockerfile 中的格式：`command: ["bundle", "exec", "thin", "-p", "3000"]`  

### services.container.container_name
`container_name: my-web-container`  
指定自定义的容器名，容器名必须全局唯一，如果指定了名字就不能起多个了
Compose 的容器名称格式是：<项目名称><服务名称><序号>,可以自定义项目名称、服务名称:  
`container_name: app`  

### services.container.devices
```yaml
devices:
  - "/dev/ttyUSB0:/dev/ttyUSB0"
```
设置映射列表，与 Docker 客户端的 --device 参数类似

### services.container.depends_on
这个标签解决了容器的依赖、启动先后的问题。它仅仅解决启动顺序，并不会等待服务就绪。  
links, volumes_from, network_mode: "service:..."也潜在的存在依赖关系
例如下面容器会先启动 redis 和 db 两个服务，最后才启动 web 服务：
```yaml
version: '2'
services:
  web:
    build: .
    depends_on:
      - db
      - redis
  redis:
    image: redis
  db:
    image: postgres
```

### services.container.devices
```yaml
devices:
  - "/dev/ttyUSB0:/dev/ttyUSB0"
```
设备映射列表。与Docker client的--device参数类似。

### services.container.dns
`dns: 8.8.8.8`
```yaml
dns:
  - 8.8.8.8
  - 9.9.9.9
```
自定义DNS，可以是单个值或列表

### services.container.dns_search
```yaml
dns_search: example.com
dns_search:
  - dc1.example.com
  - dc2.example.com
```
自定义DNS搜索域

### services.container.entrypoint
`entrypoint: /code/entrypoint.sh`  
```yaml
entrypoint:
    - php
    - -d
    - zend_extension=/usr/local/lib/php/extensions/no-debug-non-zts-20100525/xdebug.so
    - -d
    - memory_limit=-1
    - vendor/bin/phpunit
```
覆盖 Dockerfile 中的定义，并忽略CMD的影响

### services.container.env_file
```yaml
env_file: .env
env_file:
  - ./common.env
  - ./apps/web.env
  - /opt/secrets.env
```
通过文件添加ENV变量，会被environment中的变量覆盖  
如果通过 docker-compose -f FILE 指定了compose文件，则 env_file 中路径会使用compose文件路径。  
如果在配置文件中有 build 操作，这些变量并不会进入构建过程中，如果要在构建中使用变量还是首选 arg 标签。  
配置文件中的变量定义以 VAR=VAL 格式，其中以 # 开头的被解析为注释而被忽略

### services.container.environment
```yaml
environment:
  RACK_ENV: development
  SHOW: 'true'
  SESSION_SECRET:

environment:
  - RACK_ENV=development
  - SHOW=true
  - SESSION_SECRET
```
如果在配置文件中有 build 操作，这些变量并不会进入构建过程中，如果要在构建中使用变量还是首选 arg 标签。

### services.container.expose
```yaml
expose:
 - "3000"
 - "8000"
```
用于指定暴露给linked服务的端口，端口映射需要使用ports。

### services.container.external_links
```yaml
external_links:
 - redis_1
 - project_db_1:mysql
 - project_db_1:postgresql
```
它可以让Compose项目里面的容器连接到外部的容器（前提是外部容器中必须至少有一个容器是连接到与项目内的服务的同一个网络里面）。

### services.container.extra_hosts
添加主机名的标签，就是往/etc/hosts文件中添加一些记录，与Docker client的--add-host类似：
```yaml
extra_hosts:
 - "somehost:162.242.195.82"
 - "otherhost:50.31.209.229"
```
启动之后查看容器内部/etc/hosts：
```
162.242.195.82  somehost
50.31.209.229   otherhost
```

### services.container.healthcheck
用于检查容器是否正常
```yaml
healthcheck:
  test: ["CMD", "curl", "-f", "http://localhost"]
  interval: 1m30s
  timeout: 10s
  retries: 3
  start_period: 40s
```
interval，timeout 以及 start_period 都定为持续时间  
test 必须是字符串或列表，如果它是一个列表，第一项必须是 NONE，CMD 或 CMD-SHELL,如果它是一个字符串，则相当于指定CMD-SHELL 后跟该字符串。
```yaml
# Hit the local web app
test: ["CMD", "curl", "-f", "http://localhost"]
# As above, but wrapped in /bin/sh. Both forms below are equivalent.
test: ["CMD-SHELL", "curl -f http://localhost || exit 1"]
test: curl -f https://localhost || exit 1
```
如果需要禁用检查，可以使用 disable:true,相当于 test:["NONE"]
```yaml
healthcheck:
  disable: true
```

### services.container.labels
```yaml
labels:
  com.example.description: "Accounting webapp"
  com.example.department: "Finance"
  com.example.label-with-empty-value: ""
labels:
  - "com.example.description=Accounting webapp"
  - "com.example.department=Finance"
  - "com.example.label-with-empty-value"
```
向容器添加元数据，和Dockerfile的LABEL指令一个意思

### services.container.links
```yaml
links:
 - db
 - db:database
 - redis
```
这个标签用于容器连接，与Docker client的--link一样效果，会连接到其它服务中的容器。  
使用的别名将会自动在服务容器中的/etc/hosts里创建。例如：
```
172.12.2.186  db
172.12.2.186  database
172.12.2.187  redis
```
如果同时定义了links和networks，links之间的services必须共享至少一个network

### services.container.logging
```yaml
logging:
  driver: syslog
  options:
    syslog-address: "tcp://192.168.0.42:123"
```
这个标签用于配置日志，默认的driver是json-file。只有json-file和journald对docker-compose有效

### services.container.network_mode
```yaml
network_mode: "bridge"
network_mode: "host"
network_mode: "none"
network_mode: "service:[service name]"
network_mode: "container:[container name/id]"
```
网络模式，与Docker client的--net参数类似，只是相对多了一个service:[service name] 的格式  
可以指定使用服务或者容器的网络。

### services.container.networks
```yaml
services:
  some-service:
    networks:
     - some-network
     - other-network
```
加入指定网络  
aliases设置服务别名的标签,相同的服务可以在不同的网络有不同的别名：
```yaml
services:
  some-service:
    networks:
      some-network:
        aliases:
         - alias1
         - alias3
      other-network:
        aliases:
         - alias2
```
使用ipv4_address、ipv6_address为服务的容器指定一个静态 IP 地址，网络必须配置ipam块的subnet
```yaml
version: '2.1'
services:
 app:
   image: busybox
   command: ifconfig
   networks:
     app_net:
       ipv4_address: 172.16.238.10
       ipv6_address: 2001:3984:3989::10

networks:
 app_net:
   driver: bridge
   enable_ipv6: true
   ipam:
     driver: default
     config:
     - subnet: 172.16.238.0/24
     - subnet: 2001:3984:3989::/64
```

### services.container.pid
`pid: "host"`  
将PID模式设置为主机PID模式，跟主机系统共享进程命名空间。容器使用这个标签将能够访问和操纵其他容器和宿主机的名称空间。

### services.container.ports
映射端口，与network_mode: host不兼容。  
* SHORT 语法, 使用HOST:CONTAINER格式或者只是指定容器的端口(宿主机会随机映射端口)。  
```yaml
ports:
 - "3000"
 - "3000-3005"
 - "8000:8000"
 - "9090-9091:8080-8081"
 - "49100:22"
 - "127.0.0.1:8001:8001"
 - "127.0.0.1:5000-5010:5000-5010"
 - "6060:6060/udp"
```
注意：当使用HOST:CONTAINER格式来映射端口时，如果你使用的容器端口小于60你可能会得到错误得结果，因为YAML将会解析xx:yy这种数字格式为60进制。所以建议采用字符串格式。  
* LONG 语法  
target：容器内的端口  
published：公开的端口  
protocol： 端口协议（tcp 或 udp）  
mode：通过host 用在每个节点发布的主机端口或使用 ingress 用于集群模式端口进行平衡负载  
```yaml
ports:
  - target: 80
    published: 8080
    protocol: tcp
    mode: host
```

### services.container.restart
```yaml
restart: "no"
restart: always
restart: on-failure
restart: unless-stopped
```

### services.container.secrets
获取访问权限
* SHORT 语法
```yaml
version: "3.1"
services:
  redis:
    image: redis:latest
    deploy:
      replicas: 1
    secrets:
      - my_secret
      - my_other_secret
secrets:
  my_secret:
    file: ./my_secret.txt
  my_other_secret:
    external: true
```
值指定名字列表，容器会mount`/run/secrets/<secret_name>`
* LONG 语法
```yaml
version: "3.1"
services:
  redis:
    image: redis:latest
    deploy:
      replicas: 1
    secrets:
      - source: my_secret
        target: redis_secret
        uid: '103'
        gid: '103'
        mode: 0440
secrets:
  my_secret:
    file: ./my_secret.txt
  my_other_secret:
    external: true
```
1. source: 名字.
2. target: 容器mount的名字`/run/secrets/<target>`
3. uid and gid: 容器中target的 UID or GID
4. mode: 容器中target的权限0440.

### services.container.security_opt
```yaml
security_opt:
  - label:user:USER
  - label:role:ROLE
```
为每个容器覆盖默认的标签。简单说来就是管理全部服务的标签。比如设置全部服务的user标签值为USER。

### services.container.stop_grace_period
```yaml
stop_grace_period: 1s
stop_grace_period: 1m30s
```
使用SIGKILL强行关闭容器前，等待容器终止的时间（通过SIGTERM或者stop_signal）,默认10秒

### services.container.stop_signal
`stop_signal: SIGUSR1`  
设置停止容器信号。在默认是SIGTERM。

### services.container.sysctls
```yaml
sysctls:
  net.core.somaxconn: 1024
  net.ipv4.tcp_syncookies: 0

sysctls:
  - net.core.somaxconn=1024
  - net.ipv4.tcp_syncookies=0
```
设置容器的内核参数，可以为数组或字典

### services.container.tmpfs
```yaml
tmpfs: /run
tmpfs:
  - /run
  - /tmp
```
挂载临时目录到容器内部

### services.container.ulimits
```yaml
ulimits:
  nproc: 65535
  nofile:
    soft: 20000
    hard: 40000
```
设置容器的ulimits，可以设为一个整数，也可以用soft/hard 限制

### services.container.userns_mode
`userns_mode: "host"`
禁用用户命名空间

### volumes
挂载一个主机目录或者一个已存在的数据卷，如果需要再容器之间共享数据，需要使用命名数据卷。  
```yaml
version: "3.2"
services:
  web:
    image: nginx:alpine
    volumes:
      - type: volume
        source: mydata
        target: /data
        volume:
          nocopy: true
      - type: bind
        source: ./static
        target: /opt/app/static

  db:
    image: postgres:latest
    volumes:
      - "/var/run/postgres/postgres.sock:/var/run/postgres/postgres.sock"
      - "dbdata:/var/lib/postgresql/data"

volumes:
  mydata:
  dbdata:
```
* SHORT 语法
```
volumename:containerdir
hostdir:containerdir
hostdir:containerdir:ro
```
可以使用相对目录，相对于Compose文件，使用 . 或者 .. 来指定相对目录。  
```yaml
volumes:
  # 只是指定一个路径，Docker 会自动在创建一个数据卷（这个路径是容器内部的）。
  - /var/lib/mysql
  # 使用绝对路径挂载数据卷
  - /opt/data:/var/lib/mysql
  # 以 Compose 配置文件为中心的相对路径作为数据卷挂载到容器。
  - ./cache:/tmp/cache
  # 使用用户的相对路径（~/ 表示的目录是 /home/<用户目录>/ 或者 /root/）。
  - ~/configs:/etc/configs/:ro
  # 已经存在的命名的数据卷。
  - datavolume:/var/lib/mysql
```
* LONG 语法
type：mount类型，可以为 volume、bind 或 tmpfs  
source：主机目录或命名数据卷,不适用于 tmpfs 类型安装。  
target：容器中的路径  
read_only：数据在容器中是否只读  
bind：配置额外的绑定选项   
volume：配置额外的音量选项  
tmpfs：配置额外的 tmpfs 选项  

```yaml
version: "3.2"
services:
  web:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - type: volume
        source: mydata
        target: /data
        volume:
          nocopy: true
      - type: bind
        source: ./static
        target: /opt/app/static

networks:
  webnet:

volumes:
  mydata:
```

### services.container.[domainname|hostname|ipc|mac_address|privileged|read_only|shm_size|stdin_open| tty|user|working_dir]
```yaml
user: postgresql
working_dir: /code

domainname: foo.com
hostname: foo
ipc: host
mac_address: 02:42:ac:11:65:43

privileged: true


read_only: true
shm_size: 64M
stdin_open: true
tty: true
```
都是一个单值的标签，类似于使用docker run的效果。

### services.container.secrets
通过 secrets为每个服务授予相应的访问权限
* SHORT 语法
```yaml
version: "3.1"
services:
  redis:
    image: redis:latest
    deploy:
      replicas: 1
    secrets:
      - my_secret
      - my_other_secret
secrets:
  my_secret:
    file: ./my_secret.txt
  my_other_secret:
    external: true
```
* LONG 语法
LONG 语法可以添加其他选项  
source：secret 名称  
target：在服务任务容器中需要装载在 /run/secrets/ 中的文件名称，如果 source 未定义，那么默认为此值  
uid&gid：在服务的任务容器中拥有该文件的 UID 或 GID 。如果未指定，两者都默认为 0。  
mode：以八进制表示法将文件装载到服务的任务容器中 /run/secrets/ 的权限。例如，0444 代表可读。  
```yaml
version: "3.1"
services:
  redis:
    image: redis:latest
    deploy:
      replicas: 1
    secrets:
      - source: my_secret
        target: redis_secret
        uid: '103'
        gid: '103'
        mode: 0440
secrets:
  my_secret:
    file: ./my_secret.txt
  my_other_secret:
    external: true
```
---
# volumes下的标签
```yaml
version: "3"

services:
  db:
    image: db
    volumes:
      - data-volume:/var/lib/db
  backup:
    image: backup-service
    volumes:
      - data-volume:/var/lib/backup/data

volumes:
  data-volume:
```
创建容器间可复用的命名卷，不要使用volumes_from

### volumes.volume.driver
设置使用的volume驱动，默认是local

### volumes.volume.driver_opt
```
volumes:
  example:
    driver_opts:
      type: "nfs"
      o: "addr=10.40.0.199,nolock,soft,rw"
      device: ":/docker/example"
```      
传给driver的参数

### volumes.volume.external
```
version: '3'

services:
  db:
    image: postgres
    volumes:
      - data:/var/lib/postgresql/data

volumes:
  data:
    external: true
  data2:
    external:
      name: actual-name-of-volume
```
如果是true或卷名，表示数据卷是compose外部创建的  

### volumes.volume.labels
```
labels:
  com.example.description: "Database volume"
  com.example.department: "IT/Ops"
  com.example.label-with-empty-value: ""

labels:
  - "com.example.description=Database volume"
  - "com.example.department=IT/Ops"
  - "com.example.label-with-empty-value"
```

### volumes.volume.name
```
version: '3.4'
volumes:
  data:
    name: my-app-data
  data1:
    external: true
    name: my-app-data1
```
设置自定义卷名  
---

# networks

### networks.network.driver
`driver: overlay`  
设置driver，单主机默认是bridge，swarm是overlay

### networks.network.driver_opts
```
driver_opts:
    foo: "bar"
    baz: 1
```
传给driver的参数

### networks.network.attachable
```
networks:
  mynet1:
    driver: overlay
    attachable: true
```
driver为overlay时有效，如果是true，独立容器可以连接到这个network

### networks.network.ipam
```
ipam:
  driver: default
  config:
    - subnet: 172.28.0.0/16
```

### networks.network.internal
如果是true,隔离外部网络

### networks.network.labels
```
labels:
  com.example.description: "Financial transaction network"
  com.example.department: "Finance"
  com.example.label-with-empty-value: ""

labels:
  - "com.example.description=Financial transaction network"
  - "com.example.department=Finance"
  - "com.example.label-with-empty-value"
```
### networks.network.external
```
version: '3'

services:
  proxy:
    build: ./proxy
    networks:
      - outside
      - default
  app:
    build: ./app
    networks:
      - default

networks:
  outside:
    external: true
  outside1:
    external:
      name: actual-name-of-network
```
如果是true或者网络名,表示network是compose外部创建的

### networks.network.name
```
version: '3.5'
networks:
  network1:
    name: my-app-net
  network2:
    external: true
    name: my-app-net2
```
设置network名

deploy/configs仅对swarm(docker stack deploy)有效，被docker-compose忽略
### deploy
指定与部署和运行服务相关的配置
```yaml
version: '3'
services:
  redis:
    image: redis:alpine
    deploy:
      replicas: 6
      update_config:
        parallelism: 2
        delay: 10s
      restart_policy:
        condition: on-failure
```

### endpoint_mode
指定连接到群组外部客户端服务发现方法
* endpoint_mode: vip ：Docker 为该服务分配了一个虚拟 IP(VIP),作为客户端的 “前端“ 部位用于访问网络上的服务。
* endpoint_mode: dnsrr : DNS轮询（DNSRR）服务发现不使用单个虚拟 IP。Docker为服务设置 DNS 条目，使得服务名称的 DNS 查询返回一个 IP 地址列表，并且客户端直接连接到其中的一个。如果想使用自己的负载平衡器，或者混合 Windows 和 Linux 应用程序，则 DNS 轮询调度（round-robin）功能就非常实用。

### labels
指定服务的标签，这些标签仅在服务上设置。
```yaml
version: "3"
services:
  web:
    image: web
    deploy:
      labels:
        com.example.description: "This label will appear on the web service"
```
通过将 deploy 外面的 labels 标签来设置容器上的 labels
```yaml
version: "3"
services:
  web:
    image: web
    labels:
      com.example.description: "This label will appear on all containers for the web service"
```

### mode
* global:每个集节点只有一个容器  
* replicated:指定容器数量（默认）  
```yaml
version: '3'
services:
  worker:
    image: dockersamples/examplevotingapp_worker
    deploy:
      mode: global
```

### placement
指定 constraints 和 preferences
```yaml
version: '3'
services:
  db:
    image: postgres
    deploy:
      placement:
        constraints:
          - node.role == manager
          - engine.labels.operatingsystem == ubuntu 14.04
        preferences:
          - spread: node.labels.zone
```

### replicas
如果服务是 replicated（默认)，需要指定运行的容器数量
```yaml
version: '3'
services:
  worker:
    image: dockersamples/examplevotingapp_worker
    networks:
      - frontend
      - backend
    deploy:
      mode: replicated
      replicas: 6
```

### resources
配置资源限制
```yaml
version: '3'
services:
  redis:
    image: redis:alpine
    deploy:
      resources:
        limits:
          cpus: '0.50'
          memory: 50M
        reservations:
          cpus: '0.25'
          memory: 20M
```

### restart_policy
配置容器的重新启动，代替 restart

condition:值可以为 none 、on-failure 以及 any(默认)  
delay: 尝试重启的等待时间，默认为 0  
max_attempts:在放弃之前尝试重新启动容器次数（默认：从不放弃）。如果重新启动在配置中没有成功，则此尝试不计入配置max_attempts 值。例如，如果 max_attempts 值为 2，并且第一次尝试重新启动失败，则可能会尝试重新启动两次以上。  
windows:在决定重新启动是否成功之前的等时间，指定为持续时间（默认值：立即决定）。  
```yaml
version: "3"
services:
  redis:
    image: redis:alpine
    deploy:
      restart_policy:
        condition: on-failure
        delay: 5s
        max_attempts: 3
        window: 120s
```

### update_config
配置更新服务，用于无缝更新应用（rolling update)

parallelism：一次性更新的容器数量  
delay：更新一组容器之间的等待时间。  
failure_action：如果更新失败，可以执行的的是 continue、rollback 或 pause （默认）  
monitor：每次任务更新后监视失败的时间(ns|us|ms|s|m|h)（默认为0）  
max_failure_ratio：在更新期间能接受的失败率  
order：更新次序设置，top-first（旧的任务在开始新任务之前停止）、start-first（新的任务首先启动，并且正在运行的任务短暂重叠）（默认 stop-first）  
```yaml
version: '3.4'
services:
  vote:
    image: dockersamples/examplevotingapp_vote:before
    depends_on:
      - redis
    deploy:
      replicas: 2
      update_config:
        parallelism: 2
        delay: 10s
        order: stop-first
```
---
