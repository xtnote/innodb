
```
# Nginx
#
# VERSION               0.0.1

FROM      ubuntu
LABEL Description="This image is used to start the foobar executable" Vendor="ACME Products" Version="1.0"
RUN apt-get update && apt-get install -y inotify-tools nginx apache2 openssh-server
# Firefox over VNC
#
# VERSION               0.3

FROM ubuntu

# Install vnc, xvfb in order to create a 'fake' display and firefox
RUN apt-get update && apt-get install -y x11vnc xvfb firefox
RUN mkdir ~/.vnc
# Setup a password
RUN x11vnc -storepasswd 1234 ~/.vnc/passwd
# Autostart firefox (might not be the best way, but it does the trick)
RUN bash -c 'echo "firefox" >> /.bashrc'

EXPOSE 5900
CMD    ["x11vnc", "-forever", "-usepw", "-create"]
```
```
# Multiple images example
#
# VERSION               0.1

FROM ubuntu
RUN echo foo > bar
# Will output something like ===> 907ad6c2736f

FROM ubuntu
RUN echo moo > oink
# Will output something like ===> 695d7793cbe4

# You'll now have two images, 907ad6c2736f with /bar, and 695d7793cbe4 with
# /oink.
```

Dockerfile的指令是忽略大小写的，建议使用大写，使用#作为注释，每一行只支持一条指令，每条指令可以携带多个参数。  
Dockerfile 是一个文本文件，其内包含了一条条的指令(Instruction)，每一条指令构建一层， 因此每一条指令的内容，就是描述该层应当如何构建。  

### FROM指定基础镜像
`FROM <image> [AS <name>]`  
`FROM <image>[:<tag>] [AS <name>]`  
`FROM <image>[@<digest>] [AS <name>]`  
FROM必须是第一个或者在ARG之后  
AS <name> 使用方式 copy --from=<name>  
在FROM之前声明的ARG不能在FROM之后的任何指令中使用。要使用在第一个From之前声明的arg的默认值，请使用在生成阶段内没有值的arg指令：
```
ARG VERSION=latest
FROM busybox:$VERSION
ARG VERSION # 如果没有这一行，后续指令不能使用$VERSION
RUN echo $VERSION > image_version
```

### RUN执行命令
格式如下两种：
1. shell格式：`RUN <command>`  
在shell中执行，默认是/bin/sh -c command  
2. exec格式：`RUN ["executable", "param1", "param2"]`  
使用exec执行，使用bash执行RUN ["/bin/bash", "-c", "echo hello"]  

### COPY复制文件
格式：  
`COPY [--chown=<user>:<group>] <src>... <dest>`  
`COPY [--chown=<user>:<group>] ["<src>",... "<dest>"]`  

### ADD更高级复制文件
`ADD [--chown=<user>:<group>] <src>... <dest>`  
`ADD [--chown=<user>:<group>] ["<src>",... "<dest>"]`  
ADD指令和COPY的格式和性质基本一致。  
src可以是一个URL，Docker引擎会试图去下载这个链接的文件放到dest去。下载后的文件权限自动设置为600,
如果src为一个tar压缩文件的话，压缩格式为gzip , bzip2以及xz的情况 下，ADD指令将会自动解压缩这个压缩文件到dest去。

### CMD容器启动命令
1. shell格式：`CMD command param1 param2`  
使用 /bin/sh -c 执行cmd  
2. exec 格式：`CMD ["executable","param1","param2"]`  
使用exec执行cmd  
3. `CMD ["param1","param2"]`  
作为ENTRYPOINT的参数，ENTRYPOINT必须使用格式ENTRYPOINT ["executable", "param1", "param2"]  
如果用户启动容器时候指定了运行的命令，则会覆盖掉 CMD 指定的命令。只能有一个CMD，如果有多个，只有最后一个生效。  

### ENTRYPOINT入口点
1. `ENTRYPOINT ["executable", "param1", "param2"] (exec form, preferred)`  
2. `ENTRYPOINT command param1 param2 (shell form)`  
ENTRYPOINT的目的和CMD一样，都是在指定容器启动程序及参数。ENTRYPOINT 在运行时也可以替代，需要通过docker run的参数 -entrypoint来指定。  
当指定了ENTRYPOINT后，CMD的含义就发生了改变，不再是直接的运行其命令，而是将CMD的内容作为参数传给ENTRYPOINT指令，换句话说实际执行时，将变为：<ENTRYPOINT> "<CMD>"

### ENV 设置环境变量
ENV环境变量格式：  
```
ENV <key> <value>  
ENV <key>=<value> ...  
```
通过$key ${key}访问  
${variable:-word}表示如果设置了变量，则结果将是该值。如果未设置变量，则结果为Word。  
${variable:+word}表示如果设置了变量，那么word将是结果，否则结果是空字符串。  
在所有情况下，word可以是任何字符串，包括附加的环境变量。  

### ARG构建参数
格式：`ARG <name>[=<default value>]`  
通过$key ${key}访问  
作用范围从定义开始到build stage结束为止，会被同名的ENV变量覆盖  
构建参数和ENV的效果一样，都是设置环境变量。所不同的是，ARG所设置的构建环境的环境变量，在将来容器运行时是不会存在这些环境变量的。  
Dockerfile中的ARG指令是定义参数名称，以及定义其默认值。该默认值可以在构建命令docker build中用--build-arg <varname>=<value>来覆盖。  

### VOLUME定义匿名卷
格式为：`VOLUME ["<路径1>", "<路径2>"...]`  
效果等同于docker run -v 路径  

### EXPOSE声明端口
格式为: `EXPOSE <port> [<port>/<protocol>...]`  
EXPOSE 指令是声明运行时容器提供服务端口，这只是一个声明，在运行时并不会因为这个声明应用就会开启这个端口的服务。  
我们不能指定Host主机对应的PORT，Expose只能告诉docker：这个端口可以被link的兄弟container访问到。  
```
EXPOSE 80/tcp
EXPOSE 80/udp
```

### WORKDIR指定工作目录
格式为: `WORKDIR <工作目录路径>`  
在执行RUN后面的shell命令前会先cd进WORKDIR后面的目录  

### USER指定当前用户
格式：`USER <用户名>`  

### LABEL
格式：`LABEL <key>=<value> <key>=<value> <key>=<value> ...`  
向镜像添加元数据  
```
LABEL "com.example.vendor"="ACME Incorporated"
LABEL com.example.label-with-value="foo"
LABEL version="1.0"
LABEL description="This text illustrates \
that label-values can span multiple lines."
```
LABEL是key、value对，如果包含空格就用引号括起来  

### HEALTHCHECK
```
1. HEALTHCHECK [OPTIONS] CMD command  
2. HEALTHCHECK NONE  
OPTIONS：
* --interval=DURATION (default: 30s)
* --timeout=DURATION (default: 30s)
* --start-period=DURATION (default: 0s)
* --retries=N (default: 3)
```
执行结果有3个值  
0: success - 健康  
1: unhealthy - 不健康  
2: reserved - 未使用  
`HEALTHCHECK --interval=5m --timeout=3s CMD curl -f http://localhost/ || exit 1`  
