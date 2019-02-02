
Dockerfile的指令是忽略大小写的，建议使用大写，使用#作为注释，每一行只支持一条指令，每条指令可以携带多个参数。  
Dockerfile 是一个文本文件，其内包含了一条条的指令(Instruction)，每一条指令构建一层， 因此每一条指令的内容，就是描述该层应当如何构建。  

### FROM指定基础镜像

### FROM (基础镜像) AS (别名)
给镜像起个别名，在后面引用这个阶段的镜像时直接使用别名

### RUN执行命令
格式如下两种：
1. shell格式：
`RUN echo '<h1>Hello, Docker!</h1>' >/usr/share/nginx/html/index.html`  
使用 /bin/sh -c 执行
2. exec格式：RUN ["可执行文件", "参数1", "参数2"]  
使用exec执行

### COPY复制文件
格式：  
COPY：<源路径>... <目标路径>   
COPY：["<源路径1>",... "<目标路径>"]  

### ADD更高级复制文件
ADD指令和COPY的格式和性质基本一致。  
<源路径>可以是一个URL，Docker引擎会试图去下载这个链接的文件放到<目标路径>去。下载后的文件权限自动设置为600
如果<源路径>为一个tar压缩文件的话，压缩格式为gzip , bzip2以及xz的情况 下，ADD指令将会自动解压缩这个压缩文件到<目标路径>去。

### CMD容器启动命令
CMD指令的格式：  
1. shell格式：CMD <命令>  
使用 /bin/sh -c 执行cmd  
2. exec 格式：CMD ["可执行文件", "参数1", "参数2"...] 参数列表格式：CMD ["参数1", "参数2"...]。在指定了 ENTRYPOINT指令后，用CMD指定具体的参数。
使用exec执行cmd  
如果用户启动容器时候指定了运行的命令，则会覆盖掉 CMD 指定的命令。

### ENTRYPOINT入口点
ENTRYPOINT的格式 RUN指令格式一样，分为exec 格式和shell格式。  
ENTRYPOINT的目的和CMD一样，都是在指定容器启动程序及参数。ENTRYPOINT 在运行时也可以替代，需要通过docker run的参数 -entrypoint来指定。
当指定了ENTRYPOINT后，CMD的含义就发生了改变，不再是直接的运行其命令，而是将CMD的内容作为参数传给ENTRYPOINT指令，换句话说实际执行时，将变为：
<ENTRYPOINT> "<CMD>"
不可被 docker run 提供的参数覆盖，而是将docker run指定的参数当做ENTRYPOINT指定命令的参数。

### ENV 设置环境变量
ENV环境变量格式：  
1. ENV <key> <value>  
2. ENV <key1>=<value1> <key2>=<value2>...   
通过$key ${key}访问

### ARG构建参数
格式：ARG <参数名>[=<默认值>]  
通过$key ${key}访问  
构建参数和ENV的效果一样，都是设置环境变量。所不同的是，ARG所设置的构建环境的环境变量，在将来容器运行时是不会存在这些环境变量的。  
Dockerfile中的ARG指令是定义参数名称，以及定义其默认值。该默认值可以在构建命令docker build中用--build-arg<参数名>=<值>来覆盖。  

### VOLUME定义匿名卷
格式为：VOLUME ["<路径1>", "<路径2>"...]  
效果等同于docker run -v 路径  

### EXPOSE声明端口
格式为: EXPOSE <端口1> [<端口2>...]  
EXPOSE 指令是声明运行时容器提供服务端口，这只是一个声明，在运行时并不会因为这个声明应用就会开启这个端口的服务。
我们不能指定Host主机对应的PORT，Expose只能告诉docker：这个端口可以被link的兄弟container访问到。

### WORKDIR指定工作目录
格式为: WORKDIR <工作目录路径>  
在执行RUN后面的shell命令前会先cd进WORKDIR后面的目录

### USER指定当前用户
格式：USER <用户名>  
