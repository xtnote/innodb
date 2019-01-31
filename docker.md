# docker

获取docker版本<br>
docker --version<br>
docker version<br>

获取docker信息<br>
docker info<br>

查看下载的image<br>
docker image ls<br>

查看容器(running, all, all in quiet mode)<br>
docker container ls<br>
docker container ls --all<br>
docker container ls -aq<br>

查看帮助<br>
docker container --help<br>

docker build -t restaurant:0.1 -f ./build/libs/DockerfileBaseapp .<br>
docker save -o restaurant.0.1.tar restaurant:0.1<br>


自定义网络：<br>
创建  docker network create my-net<br>
删除  docker network rm my-net<br>
创建容器时连接  docker create --name my-nginx --network my-net --publish 8080:80 nginx:latest<br>
运行时连接     docker network connect my-net my-nginx<br>
断开连接       docker network disconnect my-net my-nginx<br>

docker attach可以attach到一个已经运行的容器的stdin，然后进行命令执行的动作。 但是需要注意的是，如果从这个stdin中exit，会导致容器的停止。
docker exec -it bb2 /bin/bash   不会像attach方式因为退出，导致整个容器退出。 

在运行时使用-v来声明Volume,将挂载主机的/home/adrian/data目录到容器内的/data目录上：<br>
docker run -v /home/adrian/data:/data debian ls /data<br>
如果要授权一个容器访问另一个容器的Volume，我们可以使用-volumes-from参数来执行docker run<br>
docker run -it -h NEWCONTAINER --volumes-from container-test debian /bin/bash<br>

用纯数据容器来持久化数据库、配置文件或者数据文件等。官方的文档上有详细的解释。例如：<br>
docker run --name dbdata postgres echo "Data-only container for postgres"<br>

 #创建volume<br>
docker volume create volume-test1<br>
 #查看参数<br>
docker inspect volume-test1<br>
 #使用volume<br>
docker run -dit --name busybox3 -v volume-test1:/volume busybox<br>
 #查看<br>
docker inspect -f {{.Mounts}} busybox3<br>
 #查看docker数据卷<br>
docker volume ls
 #删除没使用的数据卷(谨慎使用)<br>
docker volume prune
