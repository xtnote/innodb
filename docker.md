# docker

获取docker版本
docker --version
docker version

获取docker信息
docker info

查看下载的image
docker image ls

查看容器(running, all, all in quiet mode)
docker container ls
docker container ls --all
docker container ls -aq

查看帮助
docker container --help

docker build -t restaurant:0.1 -f ./build/libs/DockerfileBaseapp .
docker save -o restaurant.0.1.tar restaurant:0.1


自定义网络：
创建  docker network create my-net
删除  docker network rm my-net
创建容器时连接  docker create --name my-nginx --network my-net --publish 8080:80 nginx:latest
运行时连接     docker network connect my-net my-nginx
断开连接       docker network disconnect my-net my-nginx
