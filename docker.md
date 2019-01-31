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
