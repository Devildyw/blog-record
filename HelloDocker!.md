### Hello Docker !

```BASH
#注意如果想在右侧使用命令行模拟操作，请先输入
#service docker start
#否则将不能执行docker命令

#拉取busybox官方镜像，启动容器并执行输出"Hello Docker"
#拉取busybox官方最新镜像
docker pull busybox
#********** Begin *********#
docker run -it busybox echo "Hello Docker"
#********** End **********#
```

### 拉取镜像

```BASH
#注意如果想在右侧使用命令行模拟操作，请先输入
#service docker start
#否则将不能执行docker命令


#拉取busybox:1.27镜像
#********** Begin *********#
docker pull busybox:1.27
#********** End **********#
```

### 启动一个容器

```BASH
#注意如果想在右侧使用命令行模拟操作，请先输入
#service docker start
#否则将不能执行docker命令

#创建并启动一个容器，容器名为firstContainer，具备busybox的运行环境。并输出hello world
#拉取busybox最新镜像
docker pull busybox
#********** Begin *********#
docker run --name firstContainer busybox echo "hello world"
#********** End **********#
```

### 停止一个容器

```BASH
#注意如果想在右侧使用命令行模拟操作，请先输入
#service docker start
#否则将不能执行docker命令

#以ubuntu镜像为基础，创建并在后台启动了一个名为firstContainer的容器（-d看不懂没关系，下一关会介绍的）
#拉取ubutun 最新镜像，实际生产中，docker pull ubutun可以省略，docker run的时候会自己去拉取。
docker pull ubuntu
docker run -itd --name firstContainer ubuntu /bin/bash
#将firstContainer容器停止！
#********** Begin *********#
docker stop firstContainer
#********** End **********#
```

### 进入一个容器

```BASH
#注意如果想在右侧使用命令行模拟操作，请先输入
#service docker start
#否则将不能执行docker命令

#基于ubuntu镜像创建并在后台启动一个名为container2的容器
#拉取ubutun 最新镜像，实际生产中，docker pull ubutun可以省略，docker run的时候会自己去拉取。
docker pull ubuntu
docker run -itd --name container2 ubuntu /bin/bash
#由于测试环境不允许从终端输入，所以请使用docker exec完成任务
#********** Begin *********#
docker exec container2 touch 1.txt
#********** End **********#
```

### 删除容器

```BASH
#注意如果想在右侧使用命令行模拟操作，请先输入
#service docker start
#否则将不能执行docker命令
#拉取ubutun ，busybox最新镜像，实际生产中，docker pull 这一步可以省略，docker run的时候会自己去拉取。
docker pull ubuntu
docker pull busybox
#创建两个容器
docker run -itd ubuntu /bin/bash
docker run busybox echo "hello world"
#删除所有容器
#********** Begin *********#
docker rm -f $(docker ps -a -q)
#********** End **********#
```

------

### 基于Commit定制镜像

```BASH
#以busybox镜像创建一个容器，在容器中创建一个hello.txt的文件。
#拉取busybox 最新镜像，实际生产中，docker pull 这一步可以省略，docker run的时候会自己去拉取。
docker pull busybox
docker run --name container1 busybox touch hello.txt
#将对容器container1做出的修改提交为一个新镜像，镜像名为busybox:v1
#********** Begin *********#
docker commit container1 busybox:v1
#********** End **********#
```

### 基于save保存镜像与基于load加载镜像

```BASH
#首先拉取一个busybox镜像
docker pull busybox:latest

#1.将busybox:latest镜像保存到tar包
#********** Begin *********#
docker save busybox:latest > busybox.tar
#********** End **********#

#删除busybox:latest镜像
docker rmi busybox:latest

#2.从tar包加载busybox:latest镜像
#********** Begin *********#
docker load < busybox.tar
#********** End **********#
```

### 导入导出容器

```BASH
#以busybox为镜像创建一个容器，容器名为busyboxContainer
#拉取busybox 最新镜像，实际生产中，docker pull 这一步可以省略，docker run的时候会自己去拉取。
docker pull busybox

docker run --name busyboxContainer busybox echo "hello"
#1.然后将busyboxContainer导出为容器快照：busybox.tar
#********** Begin *********#
docker export busyboxContainer > busybox.tar
#********** End **********#

#2.最后使用该容器快照导入镜像，镜像名为busybox:v1.0。
#********** Begin *********#
cat busybox.tar | docker import - busybox:v1.0
#********** End **********#
```

### 删除镜像

```BASH
#以busybox为基础镜像创建一个容器，容器名为container3
#拉取busybox 最新镜像，实际生产中，docker pull 这一步可以省略，docker run的时候会自己去拉取。
docker pull busybox

docker run --name container3 busybox:latest echo "hello"
#然后将busybox:latest镜像删除
#********** Begin *********#
docker rmi -f busybox:latest
#********** End **********#
```

### 构建私有Registry

```BASH
#构建一个私人仓库
docker pull registry:2
docker run -d -p 5000:5000 --restart=always --name myregistry registry:2

#拉取busybox镜像
docker pull busybox

#1.使用docker tag给busybox加上一个标签localhost:5000/my-busybox:latest
#********** Begin *********#
docker tag busybox localhost:5000/my-busybox:latest
#********** End **********#

#2.将localhost:5000/my-busybox:latest镜像推送到私人仓库
#********** Begin *********#
docker push localhost:5000/my-busybox:latest
#********** End **********#

#删除本地镜像
docker rmi localhost:5000/my-busybox:latest

#3.从私人仓库拉取localhost:5000/my-busybox:latest镜像
#********** Begin *********#
docker pull localhost:5000/my-busybox:latest
#********** End **********#

#删除私人仓库并将私人仓库中的镜像也删除掉
docker rm -vf myregistry
```

------

### 初识Dockerfile

```BASH
#创建一个空文件夹，并进入其中
mkdir newdir1
cd newdir1
#创建一个Dockerfile文件
touch Dockerfile
#假设我的Dockerfile文件为
#FROM ubuntu
#RUN mkdir dir1
#可以这么写：
# echo 'FROM ubuntu' > Dockerfile
# echo 'RUN mkdir dir1'>> Dockerfile
#输入Dockerfile文件内容
#********** Begin *********#
#以busybox为基础镜像

#在基础镜像的基础上，创建一个hello.txt文件

#********** End **********#
#使用Dockerfile创建一个新镜像，镜像名为busybox:v1
docker build -t busybox:v1 .
```

### docker build、COPY和ADD

```BASH
#创建一个空文件夹，并进入其中
mkdir newdir2
cd newdir2
#创建一个文件夹dir1，将其压缩，然后删除dir1
mkdir dir1 && tar -cvf dir1.tar dir1 && rmdir dir1
#创建一个Dockerfile文件
touch Dockerfile
#假设我的Dockerfile文件为
#FROM ubuntu
#RUN mkdir dir1
#可以这么写：
# echo 'FROM ubuntu' > Dockerfile
# echo 'RUN mkdir dir1'>> Dockerfile
#输入Dockerfile文件内容
#********** Begin *********#
#以busybox为基础镜像
echo 'FROM busybox' > Dockerfile
#并将上下文目录下的dir1.tar“解压提取后”，拷贝到busybox:v3的/
echo 'RUN mkdir dir1' >> Dockerfile
echo 'ADD ./dir1.tar /' >> Dockerfile
#********** End **********#

#文件内容完毕，在当前文件夹中执行
#********** Begin *********#
#以该Dockerfile构建一个名为busybox:v3的镜像
docker build -t busybox:v3 .
#********** End **********#
```

### CMD和ENTRYPOINT指令

```BASH
#创建一个空文件夹，并进入其中
mkdir newdir3
cd newdir3
#创建一个Dockerfile文件
touch Dockerfile
#假设我的Dockerfile文件为
#FROM ubuntu
#RUN mkdir dir1
#可以这么写：
# echo 'FROM ubuntu' > Dockerfile
# echo 'RUN mkdir dir1'>> Dockerfile
#输入Dockerfile文件内容
#********** Begin *********#
#以busybox为基础镜像
echo 'FROM busybox' > Dockerfile
#默认情况下，将启动命令设置为df -Th。要求df命令不能被覆盖，但-Th能够被覆盖。
echo 'ENTRYPOINT ["df"]' >> Dockerfile
echo 'CMD ["-Th"]' >> Dockerfile
#********** End **********#

#文件内容完毕，在当前文件夹中执行
#********** Begin *********#
#以该Dockerfile构建一个名为mydisk:latest的镜像
docker build -t mydisk:latest .
#********** End **********#
```

### ENV、EXPOSE、WORKDIR、ARG指令

```BASH
#创建一个空文件夹，并进入其中
mkdir newdir4
cd newdir4
#创建一个Dockerfile文件
touch Dockerfile
#假设我的Dockerfile文件为
#FROM ubuntu
#RUN mkdir dir1
#可以这么写：
# echo 'FROM ubuntu' > Dockerfile
# echo 'RUN mkdir dir1'>> Dockerfile
#输入Dockerfile文件内容
#********** Begin *********#
#以busybox为基础镜像
echo 'FROM busybox' > Dockerfile
#声明暴露3000端口
echo 'EXPOSE 3000'>> Dockerfile
#将变量var1=test设置为环境变量
echo 'ENV var1=test'>> Dockerfile
#设置工作目录为/tmp
echo 'WORKDIR /tmp'>> Dockerfile
#在工作目录下创建一个1.txt文件
echo "RUN touch 1.txt" >> Dockerfile
#********** End **********#
#文件内容完毕，在当前文件夹中执行
#********** Begin *********#
#以该Dockerfile构建一个名为testimage:v1的镜像
docker build -t testimage:v1 .
#********** End **********#
```

------

### 创建一个数据卷

```BASH
#创建一个名为vo1的数据卷，并将该数据卷挂载到container1容器的/dir1目录。
#拉取ubutun 最新镜像，实际生产中，docker pull 这一步可以省略，docker run的时候会自己去拉取。
docker pull ubuntu

#********** Begin *********#
docker volume create vo1
docker run -itd --name container1 -v vo1:/dir1 ubuntu /bin/bash
#********** End **********#
```

### 挂载和共享数据卷

```BASH
#1.创建一个名为container1的容器，并将本地主机的/dir1目录挂载到容器中的/codir1中。
#********** Begin *********#
docker run -d --name container1 -v /dir1:/codir1 ubuntu 
#********** End **********#
#2.创建一个名为container2的容器，与container1共享数据卷。
#********** Begin *********#
docker run -d --name container2 --volumes-from container1 ubuntu
#********** End **********#
```

### 查看数据卷的信息

```BASH
#创建一个容器，并创建一个随机名字的数据卷挂载到容器的/data目录
#拉取ubutun 最新镜像，实际生产中，docker pull 这一步可以省略，docker run的时候会自己去拉取。
docker pull ubuntu &> /dev/null 
docker rm container1 -f &>/dev/null
docker run -v /data --name container1 ubuntu
#输出容器container1创建的数据卷的名字
#********** Begin *********#
docker inspect --type container --format='{{range .Mounts}}{{.Name}}{{end}}' container1
#********** End **********#
```

### 删除数据卷

```BASH
#创建一个名为container1的容器，创建一个数据卷挂载到容器的/data目录
#拉取ubutun 最新镜像，实际生产中，docker pull 这一步可以省略，docker run的时候会自己去拉取。
docker pull ubuntu

docker run -v vo4:/data --name container1 ubuntu
#删除container1对应的数据卷
#********** Begin *********#
docker rm -f -v container1
docker volume rm vo4
#********** End **********#
```

### 备份、恢复数据卷

```BASH
#拉取ubutun 最新镜像，实际生产中，docker pull 这一步可以省略，docker run的时候会自己去拉取。
docker pull ubuntu
# 创建一个vo1的数据卷，并在数据卷中添加1.txt文件
docker run --name vocontainer1 -v vo1:/dir1 ubuntu touch /dir1/1.txt
#1.将vo1数据卷的数据备份到宿主机的/newback中,将容器的/backup路径挂载上去，并将容器内/dir1文件夹打包至/backup/backup.tar
#********** Begin *********#
docker run --volumes-from vocontainer1 -v /newback:/backup ubuntu tar -cvf /backup/backup.tar /dir1
#********** End **********#
#删除所有的容器以及它使用的数据卷
docker rm -vf $(docker ps -aq)
docker volume rm vo1
#在次创建一个vo1的数据卷
docker run -itd --name vocontainer2 -v vo1:/dir1 ubuntu /bin/bash
#2.将保存在宿主机中备份文件的数据恢复到vocontainer2的/中
#********** Begin *********#
docker run --volumes-from vocontainer2 -v /newback:/backup ubuntu tar -xvf /backup/backup.tar -C /
#********** End **********#
```

### 备份、恢复数据卷

```BASH
#!/bin/bash
#拉取ubutun 最新镜像，实际生产中，docker pull 这一步可以省略，docker run的时候会自己去拉取。
docker pull ubuntu
# 创建一个vo1的数据卷，并在数据卷中添加1.txt文件
docker run --name vocontainer1 -v vo1:/dir1 ubuntu touch /dir1/1.txt
#1.将vo1数据卷的数据备份到宿主机的/newback中,将容器的/backup路径挂载上去，并将容器内/dir1文件夹打包至/backup/backup.tar
#********** Begin *********#
docker run --volumes-from vocontainer1 -v /newback:/backup ubuntu tar -cvf /backup/backup.tar /dir1
#********** End **********#
#删除所有的容器以及它使用的数据卷
docker rm -vf $(docker ps -aq)
docker volume rm vo1
#在次创建一个vo1的数据卷
docker run -itd --name vocontainer2 -v vo1:/dir1 ubuntu /bin/bash
#2.将保存在宿主机中备份文件的数据恢复到vocontainer2的/中
#********** Begin *********#
docker run --volumes-from vocontainer2 -v /newback:/backup ubuntu tar -xvf /backup/backup.tar -C /
#********** End **********#
```