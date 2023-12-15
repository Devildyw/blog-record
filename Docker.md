# Docker

## 简介

Docker 是一个开源的应用容器引擎，基于 Go 语言 并遵从Apache2.0协议开源。Docker 可以让开发者打包他们的应用以及依赖包到一个轻量级、可移植的容器中，然后发布到任何流行的 Linux 机器上，也可以实现虚拟化。容器是完全使用沙箱机制，相互之间不会有任何接口（类似 iPhone 的 app）,更重要的是容器性能开销极低。

## 为什么会有Docker的出现

### 项目部署的问题

大型项目组件较多，运行环境也较为复杂，部署时会碰到一些问题：

- 依赖关系复杂，容易出现兼容性问题
- 开发、测试、生产环境有差异

[![image-20220819211543463](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208192116217.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208192116217.png)

传统上认为，软件编码开发/测试结束后，所产出的成果即是程序或是能够编译执行的二进制字节码等（Java为例）。而为了让这些程序可以顺利的执行，开发团队也得准备完成的部署文件，让运维团队得以部署应用程序，开发需要清楚的告诉运维部署团队，用的全部配置文件+所有软件环境。不过即便如此，任然常常发生部署失败的状况。Docker的出现使得Docker得以打破过期【程序即应用】的观念。通过镜像（images）将作业系统核心除外，运作应用程序所需要的系统环境，由下而上打包，达到系统程序跨平台间的无缝接轨运作。

### Docker如何解决依赖的兼容问题的？

- 将应用的Libs（函数库）、Deps（依赖）、配置与应用一起打包
- 将每个应用放大哦一个隔离容器去运行，避免互相干扰。

[![image-20220819211840031](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208192118091.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208192118091.png)

不同环境的操作系统不同，Docker如何解决？我们先来了解下操作系统结构

内核与硬件交互，提供操作硬件的指令系统应用封装内核指令为函数，便于程序员调用

用户程序基于系统函数库实现功能

[![image-20220819224708487](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208192247638.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208192247638.png)

Ubuntu和CentOS都是基于Linux内核，只是系统应用不同，提供的函数库有差异

[![image-20220819224909124](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208192249310.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208192249310.png)

Docker如何解决不同系统环境的问题？

- **Docker将用户程序与所需要调用的系统(比如Ubuntu)函数库一起打包**
- **Docker运行到不同操作系统时，直接基于打包的库函数，借助于操作系统的Linux内核来运行**

[![image-20220819225119661](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208192251796.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208192251796.png)

**总结：**

**Docker如何解决大型项目依赖关系复杂，不同组件依赖的兼容性问题？**

- Docker允许开发中将应用、依赖、函数库、配置一起**打包**，形成可移植镜像
- Docker应用运行在容器中，使用沙箱机制，相互**隔离**

**Docker如何解决开发、测试、生产环境有差异的问题**？

- Docker镜像中包含完整运行环境，包括系统函数库，仅依赖系统的Linux内核，因此可以在任意Linux操作系统上运行

## Docker的理念

Docker是基于Go语言实现的云开源项目。

Docker的主要目标是`“Build，Ship and Run Any App,Anywhere”，也就是通过对应用组件的封装、分发、部署、运行等生命周期的管理，使用户的APP（可以是一个WEB应用或数据库应用等等）及其运行环境能够做到“一次镜像，处处运行”。

[![image-20220515154552629](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220515154552629.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220515154552629.png)

Linux容器技术的出现就解决了这样一个问题，而 Docker 就是在它的基础上发展过来的。将应用打成镜像，通过镜像成为运行在Docker容器上面的实例，而 Docker容器在任何操作系统上都是一致的，这就实现了跨平台、跨服务器。只需要一次配置好环境，换到别的机子上就可以一键部署好，大大简化了操作。

**Docker解决了运行环境和配置问题的软件容器， 方便做持续集成并有助于整体发布的容器虚拟化技术。**

------

## Docker的优势

**简化程序：**
Docker 让开发者可以打包他们的应用以及依赖包到一个可移植的容器中，然后发布到任何流行的 Linux 机器上，便可以实现虚拟化。Docker改变了虚拟化的方式，使开发者可以直接将自己的成果放入Docker中进行管理。方便快捷已经是 Docker的最大优势，过去需要用数天乃至数周的 任务，在Docker容器的处理下，只需要数秒就能完成。

**节省开支：**
一方面，云计算时代到来，使开发者不必为了追求效果而配置高额的硬件，Docker 改变了高性能必然高价格的思维定势。Docker 与云的结合，让云空间得到更充分的利用。不仅解决了硬件管理的问题，也改变了虚拟化的方式。

## 容器与虚拟机的比较

上文我们提到了容器，那么容器和传统的虚拟机有什么区别呢？

### 容器发展简史

[![image-20220515155014898](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220515155014898.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220515155014898.png)

[![image-20220515155023912](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220515155023912.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220515155023912.png)

### 传统虚拟机

虚拟机（virtual machine）就是带环境安装的一种解决方案。

- 传统虚拟机技术：模拟一个完整的操作系统，先虚拟出一套硬件，然后在其上安装操作系统，最后在系统上再运行应用程序
  缺点：资源占用多，启动慢
  虚拟机偏向于硬件

使用虚拟机运行多个相互隔离的应用时，如下图：

[![image-20220819230904208](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208192309248.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208192309248.png)

从下到上理解上图：

**基础设施**（Infrastructure）。它可以是你的个人电脑，数据中心的服务器，或者是云主机。

**主操作系统**（Host Operating System）。你的个人电脑之上，运行的可能是MacOS，Windows或者某个Linux发行版。

**虚拟机管理系统**（Hypervisor）。利用Hypervisor，可以在主操作系统之上运行多个不同的从操作系统,实现硬件资源虚拟化。类型1的Hypervisor有支持MacOS的HyperKit，支持Windows的Hyper-V以及支持Linux的KVM。类型2的Hypervisor有VirtualBox和VMWare。

**从操作系统**（Guest Operating System）。假设你需要运行3个相互隔离的应用，则需要使用Hypervisor启动3个从操作系统，也就是3个虚拟机。这些虚拟机都非常大，也许有700MB，这就意味着它们将占用2.1GB的磁盘空间。更糟糕的是，它们还会消耗很多CPU和内存。

**各种依赖**。每一个从操作系统都需要安装许多依赖。如果你的的应用需要连接PostgreSQL的话，则需要安装libpq-dev；如果你使用Ruby的话，应该需要安装gems；如果使用其他编程语言，比如Python或者Node.js，都会需要安装对应的依赖库。

应用。安装依赖之后，就可以在各个从操作系统分别运行应用了，这样各个应用就是相互隔离的。

------

### Docker容器技术

- Docker容器技术：不是模拟一个完整的操作系统，没有进行硬件虚拟，而是对进程进行隔离，封装成容器，容器内的应用程序是直接使用宿主机的内核，且容器之间是互相隔离的，互不影响
  优点：更轻便、效率高、启动快、秒级
  Docker容器技术更多的偏向于软件

使用Docker容器运行多个相互隔离的应用时，如下图：

[![image-20220819230913550](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208192309591.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208192309591.png)

**主操作系统**（Host Operating System）。所有主流的Linux发行版都可以运行Docker。对于MacOS和Windows，也有一些办法”运行”Docker。

**Docker守护进程**（Docker Daemon）。Docker守护进程取代了Hypervisor，运行在docker容器上的程序直接使用的都是实际物理机的硬件资源。因此在CPU、内存利用率上docker将会在效率上有明显优势。它是运行在操作系统之上的后台进程，负责管理Docker容器。

**各种依赖**。对于Docker，应用的所有依赖都打包在Docker镜像中，Docker容器是基于Docker镜像创建的。

**应用**。应用的源代码与它的依赖都打包在Docker镜像中，不同的应用需要不同的Docker镜像。不同的应用运行在不同的Docker容器中，它们是相互隔离的。

------

### 两者之间的区别

- 传统虚拟机技术是虚拟出一套硬件后，在其上运行一个完整操作系统，在该系统上再运行所需应用进程；

- 而容器内的应用进程直接运行于宿主的内核，容器内没有自己的内核，而且也没有进行硬件虚拟。因此容器要比传统虚拟机更为轻便。

- 作为一种轻量级的虚拟化方式，Docker在运行应用上跟传统的虚拟机方式相比具有显著优势：

- Docker容器很快，启动和停止可以在秒级实现，这相比传统的虚拟机方式要快得多。

- Docker容器对系统资源需求很少，一台主机上可以同时运行数千个Docker容器。

- Docker通过类似Git的操作来方便用户获取、分发和更新应用镜像，指令简明，学习成本较低。

- Docker通过`Dockerfile`配置文件来支持灵活的自动化创建和部署机制，提高工作效率。

- Docker容器除了运行其中的应用之外，基本不消耗额外的系统资源，保证应用性能的同时，尽量减小系统开销。

- Docker利用Linux系统上的多种防护机制实现了严格可靠的隔离。从1.3版本开始，Docker引入了安全选项和镜像签名机制，极大地提高了使用Docker的安全性。

  | 特性       | 容器               | 虚拟机     |
  | ---------- | ------------------ | ---------- |
  | 启动速度   | 秒级               | 分钟级     |
  | 硬盘使用   | 一般为MB           | 一般为GB   |
  | 性能       | 接近原生           | 弱于原生   |
  | 系统支持量 | 单机支持上千个容器 | 一般几十个 |
  | 隔离性     | 安全隔离           | 完全隔离   |

## Docker能干嘛

- 开发/运维（DevOps）新一代开发工程师

  > - · 一次构建、随处运行
  > - · 更快速的应用交付和部署
  >   - 传统的应用开发完成后，需要提供一堆安装程序和配置说明文档，安装部署后需根据配置文档进行繁杂的配置才能正常运行。Docker化之后只需要交付少量容器镜像文件，在正式生产环境加载镜像并运行即可，应用安装配置在镜像里已经内置好，大大节省部署配置和测试验证时间。
  > - · 更便捷的升级和扩缩容
  >   - 随着微服务架构和Docker的发展，大量的应用会通过微服务方式架构，应用的开发构建将变成搭乐高积木一样，每个Docker容器将变成一块“积木”，应用的升级将变得非常容易。当现有的容器不足以支撑业务处理时，可通过镜像运行新的容器进行快速扩容，使应用系统的扩容从原先的天级变成分钟级甚至秒级。
  > - · 更简单的系统运维
  >   - 应用容器化运行后，生产环境运行的应用可与开发、测试环境的应用高度一致，容器会将应用程序相关的环境和状态完全封装起来，不会因为底层基础架构和操作系统的不一致性给应用带来影响，产生新的BUG。当出现程序异常时，也可以通过测试环境的相同容器进行快速定位和修复。
  > - · 更高效的计算资源利用
  >   - Docker是内核级虚拟化，其不像传统的虚拟化技术一样需要额外的Hypervisor支持，所以在一台物理机上可以运行很多个容器实例，可大大提升物理服务器的CPU和内存的利用率。

- Docker的应用场景

  [![image-20220515162158635](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220515162158635.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220515162158635.png)

## Docker安装

### 准备工作

------

#### 系统要求

Docker 支持 64 位版本 CentOS 7/8，并且要求内核版本不低于 3.10。 CentOS 7 满足最低内核的要求，但由于内核版本比较低，部分功能（如 `overlay2` 存储层驱动）无法使用，并且部分功能可能不太稳定。

#### 卸载旧版本

旧版本的 Docker 称为 `docker` 或者 `docker-engine`，使用以下命令卸载旧版本：

```BASH
$ sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-selinux \
                  docker-engine-selinux \
                  docker-engine
```

## 使用 yum 安装

执行以下命令安装依赖包：

```BASH
sudo yum install -y yum-utils
```

鉴于国内网络问题，强烈建议使用国内源，官方源请在注释中查看。

**这里演示的是阿里云的镜像库**

执行下面的命令添加 `yum` 软件源：

```BASH
$ sudo yum-config-manager \
    --add-repo \
    https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

$ sudo sed -i 's/download.docker.com/mirrors.aliyun.com\/docker-ce/g' /etc/yum.repos.d/docker-ce.repo

# 官方源
# $ sudo yum-config-manager \
#     --add-repo \
#     https://download.docker.com/linux/centos/docker-ce.repo
```

**安装Docker(此处我们安装社区版 docker-ce)**

```BASH
$ yum install docker-ce (这样写默认安装最新版本)
$ yum install  docker-ce-<VERSION_STRING> (指定安装版本) 
例： yum install docker-ce-18.03.1.ce
```

如果提示接受 GPG 密钥，请验证指纹是否匹配 `060A 61C5 1B55 8A7F 742B 77AA C52F EB6B 621E 9F35`，如果是，则接受它。

此命令会安装 Docker，但不会启动 Docker。它还会创建一个 `docker`组，但是默认情况下它不会将任何用户添加到该组中。启动 Docker

```BASH
sudo systemctl enable docker
sudo systemctl start docker
```

## 建立 docker 用户组

默认情况下，`docker` 命令会使用 [Unix socket](https://en.wikipedia.org/wiki/Unix_domain_socket) 与 Docker 引擎通讯。而只有 `root` 用户和 `docker` 组的用户才可以访问 Docker 引擎的 Unix socket。出于安全考虑，一般 Linux 系统上不会直接使用 `root` 用户。因此，更好地做法是将需要使用 `docker` 的用户加入 `docker` 用户组。

建立 `docker` 组：

```BASH
sudo groupadd docker
```

将当前用户加入 `docker` 组：

```BASH
sudo usermod -aG docker $USER
```

退出当前终端并重新登录，进行如下测试。

## 测试 Docker 是否安装正确

```BASH
$ docker run --rm hello-world

Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
b8dfde127a29: Pull complete
Digest: sha256:308866a43596e83578c7dfa15e27a73011bdd402185a84c5cd7f32a88b501a24
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/
```

若能正常输出以上信息，则说明安装成功。

## Docker镜像加速

如果在使用过程中发现拉取 Docker 镜像十分缓慢，可以配置 Docker 国内镜像加速。

**这里演示的是阿里云的镜像加速器**

配置镜像加速器:(针对Docker客户端版本大于 1.10.0 的用户)

```BASH
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://3h5lqxa4.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```

------

## 基本架构

**Docker 使用客户端-服务器 (C/S) 架构模式，使用远程API来管理和创建Docker容器。**

[![image-20220819231434843](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208192314918.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208192314918.png)

**Docker组件:**

> - docker Client客户端————>通过命令或RestAPI向Docker服务端发送指令。可以在本地或远程向服务端发送指令，如:创建、停止、销毁容器等操作
> - docker Server服务器进程—–>Docker守护进程，负载处理Docker指令，管理镜像、容器等。
> - docker Registry镜像仓库——>镜像存放的中央仓库，可看作是存放二进制的scm

------

### 镜像

#### Docker镜像

镜像名称一般分两部分组成：[repository]:[tag]。

操作系统分为**内核**和**用户空间**.对于`linux`这类操作系统而言,内核启动后,会挂载`root`文件文件系统为其提供用户空间支持。而**Docker镜像**（`image`），就相当于是一个`root`文件系统。比如官方镜像`ubuntu:18.04`就包含了完整的一套`Ubuntu18.04`最小系统的root文件系统。

Docker镜像是一个特殊的文件系统，除恶了提供容器运行时所需要的程序、库、资源、配置等文件外，还包含了一些为运行时准备的一些配置参数（如匿名卷、环境变量、用户等）。镜像**不包含**任何动态数据，其内容在构建之后也不会改变。

#### 分层存储

因为镜像包含操作系统完整的 `root` 文件系统，其体积往往是庞大的，因此在 Docker 设计时，就充分利用 [Union FS](https://en.wikipedia.org/wiki/Union_mount) 的技术，将其设计为分层存储的架构。所以严格来说，镜像并非是像一个 `ISO` 那样的打包文件，镜像只是一个虚拟的概念，其实际体现并非由一个文件组成，而是由一组文件系统组成，或者说，由多层文件系统联合组成。

镜像构建时，会一层层构建，前一层是后一层的基础。每一层构建完就不会再发生改变，后一层上的任何改变只发生在自己这一层。比如，删除前一层文件的操作，实际不是真的删除前一层的文件，而是仅在当前层标记为该文件已删除。在最终容器运行的时候，虽然不会看到这个文件，但是实际上该文件会一直跟随镜像。因此，在构建镜像的时候，需要额外小心，每一层尽量只包含该层需要添加的东西，任何额外的东西应该在该层构建结束前清理掉。

分层存储的特征还使得镜像的复用、定制变的更为容易。甚至可以用之前构建好的镜像作为基础层，然后进一步添加新的层，以定制自己所需的内容，构建新的镜像。

[![image-20220821121410473](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208211214598.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208211214598.png)

#### 镜像操作

指令的详细使用可以通过在末尾添加 –help来查看

> docker images
>
> docker rmi
>
> docker pull
>
> docker push
>
> docker save
>
> docker load

后续我会为大家详细介绍如何构建镜像。

------

### 容器

#### Docker容器

docker容器的命令

[![image-20220821122847191](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208211228278.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208211228278.png)

镜像（`Image`）和容器（`Container`）的关系，就像是面向对象程序设计中的 `类` 和 `实例` 一样，镜像是静态的定义，容器是镜像运行时的实体。容器可以被创建、启动、停止、删除、暂停等。

| Docker | 面向对象 |
| ------ | -------- |
| 容器   | 对象     |
| 镜像   | 类       |

容器的实质是进程，但与直接在宿主执行的进程不同，容器进程运行于属于自己的独立的 [命名空间](https://en.wikipedia.org/wiki/Linux_namespaces)。因此容器可以拥有自己的 `root` 文件系统、自己的网络配置、自己的进程空间，甚至自己的用户 ID 空间。容器内的进程是运行在一个隔离的环境里，使用起来，就好像是在一个独立于宿主的系统下操作一样。这种特性使得容器封装的应用比直接在宿主运行更加安全。看到这里也就不迷惑为什么有人会把虚拟机和Docker容器混淆了，前面我也介绍了他们之间的区别（虚拟机需要虚拟一套硬件出来）。

容器与镜像一样，也是使用分层存储，每一个容器运行时，是以镜像为基础层，在其上创建一个当前容器的存储层，我们可以称这个为容器运行时读写而准备的存储层为**容器存储层**

**容器存储层的生存周期和容器一样，容器消亡时，容器存储层也随之消亡。因此，任何保存于容器存储层的信息都会随容器删除而丢失。**

注意：按照《Docker最佳实践》的要求，容器不应该向其存储层内写入任何数据，容器存储层要保持无状态化。所有的写入操作，都应该使用**数据卷**、或者**绑定宿主目录**，在这些位置的读写会跳过容器存储层，直接对宿主（或网络存储）发生读写，其性能和稳定性更高。（毕竟容器宕掉了数据也没了，还是有一个稳定的地方存储数据安全有一点。）

**数据卷的生存周期独立于容器，容器消亡，数据卷不会消亡。因此，使用数据卷后，容器删除或者重新运行之后，数据却不会丢失。**

#### Docker Run命令常见参数

- –name：指定容器名称
- -p：指定端口映射
- -d：让容器后台运行

查看容器日志的命令：

- docker logs
- 添加 -f 参数可以持续查看日志

查看容器状态：

- docker ps

端口映射，将容器的端口映射宿主机

[![image-20220821124155858](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208211241926.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208211241926.png)

#### 注意：

对于不会运行的容器，可以取Docker Hub上查看。

[![image-20220821123628024](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208211236068.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208211236068.png)

------

### 仓库

#### Docker Registry

Docker Registry就是一个可以集中存储、分发镜像的服务。我将镜像构建完成之后，可以很容易的在当前宿主机上运行，但是，如果我需要其他服务器上使用该镜像，或者分享给别人使用时Docker Registry就很有用了。

**一个Docker Registry中可以有多个仓库（`Repository`）；每个仓库可以包含多个标签（`Tag`）；每个标签对应一个镜像。**

通常，**一个仓库会包含同一个软件不同版本的镜像，而标签就常用于对应该软件的各个版本。**我们可以通过 **`<仓库名>:<标签>`** 的格式来指定具体是这个软件哪个版本的镜像。**如果不给出标签，将以 `latest` 作为默认标签。**

以 [Ubuntu 镜像](https://hub.docker.com/_/ubuntu) 为例，`ubuntu` 是仓库的名字，其内包含有不同的版本标签，如，`16.04`, `18.04`。我们可以通过 `ubuntu:16.04`，或者 `ubuntu:18.04` 来具体指定所需哪个版本的镜像。如果忽略了标签，比如 `ubuntu`，那将视为 `ubuntu:latest`。

仓库名经常以 *两段式路径* 形式出现，比如 `jwilder/nginx-proxy`，前者往往意味着 Docker Registry 多用户环境下的用户名，后者则往往是对应的软件名。但这并非绝对，取决于所使用的具体 Docker Registry 的软件或服务。

**Docker Registry**分为公开服务和私有服务

------

#### Docker Registry 公开服务

Docker Registry 公开服务是开放给用户使用、允许用户管理镜像的 Registry 服务。一般这类公开服务允许用户免费上传、下载公开的镜像，并可能提供收费服务供用户管理私有镜像。

最常使用的 Registry 公开服务是官方的 [Docker Hub](https://hub.docker.com/)，这也是默认的 Registry，并拥有大量的高质量的 [官方镜像](https://hub.docker.com/search?q=&type=image&image_filter=official)。除此以外，还有 Red Hat 的 [Quay.io](https://quay.io/repository/)；Google 的 [Google Container Registry](https://cloud.google.com/container-registry/)，[Kubernetes](https://kubernetes.io/) 的镜像使用的就是这个服务；代码托管平台 [GitHub](https://github.com/) 推出的 [ghcr.io](https://docs.github.com/cn/packages/working-with-a-github-packages-registry/working-with-the-container-registry)。

由于某些原因，在国内访问这些服务可能会比较慢。国内的一些云服务商提供了针对 Docker Hub 的镜像服务（`Registry Mirror`），这些镜像服务被称为 **加速器**。常见的有 [阿里云加速器](https://www.aliyun.com/product/acr?source=5176.11533457&userCode=8lx5zmtu)、[DaoCloud 加速器](https://www.daocloud.io/mirror#accelerator-doc) 等。使用加速器会直接从国内的地址下载 Docker Hub 的镜像，比直接从 Docker Hub 下载速度会提高很多。在 [安装 Docker](https://docker-practice.github.io/zh-cn/install/mirror.html) 一节中有详细的配置方法。

国内也有一些云服务商提供类似于 Docker Hub 的公开服务。比如 [网易云镜像服务](https://c.163.com/hub#/m/library/)、[DaoCloud 镜像市场](https://hub.daocloud.io/)、[阿里云镜像库](https://www.aliyun.com/product/acr?source=5176.11533457&userCode=8lx5zmtu) 等。

------

#### 私有 Docker Registry

除了使用公开服务外，用户还可以在本地搭建私有 Docker Registry。Docker 官方提供了 [Docker Registry](https://hub.docker.com/_/registry/) 镜像，可以直接使用做为私有 Registry 服务。在 [私有仓库](https://docker-practice.github.io/zh-cn/repository/registry.html) 一节中，会有进一步的搭建私有 Registry 服务的讲解。

开源的 Docker Registry 镜像只提供了 [Docker Registry API](https://docs.docker.com/registry/spec/api/) 的服务端实现，足以支持 `docker` 命令，不影响使用。但不包含图形界面，以及镜像维护、用户管理、访问控制等高级功能。

除了官方的 Docker Registry 外，还有第三方软件实现了 Docker Registry API，甚至提供了用户界面以及一些高级功能。比如，[Harbor](https://github.com/goharbor/harbor) 和 [Sonatype Nexus](https://docker-practice.github.io/zh-cn/repository/nexus3_registry.html)。

------

------

## 使用Docker镜像

镜像是Docker的三大组件之一

Docker运行容器之前，需要本地存在对应的镜像，如果本地不存在该镜像，Docker会从镜像仓库中下载该镜像。

本部分会介绍关于使用Docker镜像的一些操作，包括：

> - 从仓库中获取镜像；
> - 管理本地主机上的镜像；
> - 介绍镜像实现的基本原理。

#### 获取镜像

在Docker Registry中我们提到了，[Docker Hub](https://hub.docker.com/search?q=&type=image) 上有大量的高质量的镜像可以用，这里我们就说一下怎么获取这些镜像。

从 Docker 镜像仓库获取镜像的命令是 `docker pull`。其命令格式为：

```BASH
docker pull [选项] [Docker Registry 地址[:端口号]/]仓库名[:标签]
```

具体**[选项]**可以通过`docker pull --help` 查看，这里详细介绍下镜像命名的格式。

- Docker镜像仓库地址：地址的格式一般是`<域名/IP>[:端口号]`。默认地址是Docker Hub（`docker.io`）。
- 仓库名：仓库名是两端式名称，即`<用户名>/<软件名>`。对于Docker Hub，如果不给出用户名，则默认`library`，也就是官方镜像。

如：

```BASH
$ docker pull ubuntu:18.04
18.04: Pulling from library/ubuntu
92dc2a97ff99: Pull complete
be13a9d27eb8: Pull complete
c8299583700a: Pull complete
Digest: sha256:4bc3ae6596938cb0d9e5ac51a1152ec9dcac2a1c50829c74abd9c4361e321b26
Status: Downloaded newer image for ubuntu:18.04
docker.io/library/ubuntu:18.04
```

上面的命令中没有给出 Docker 镜像仓库地址，因此将会从 Docker Hub （`docker.io`）获取镜像。而镜像名称是 `ubuntu:18.04`，因此将会获取官方镜像 `library/ubuntu` 仓库中标签为 `18.04` 的镜像。`docker pull` 命令的输出结果最后一行给出了镜像的完整名称，即： `docker.io/library/ubuntu:18.04`。

从下载过程中可以看到我们之前提及的分层存储的概念，镜像是由多层存储所构成。下载也是一层层的去下载，并非单一文件。下载过程中给出了每一层的 ID 的前 12 位。并且下载结束后，给出该镜像完整的 `sha256` 的摘要，以确保下载一致性。

在使用上面命令的时候，你可能会发现，你所看到的每层 ID 以及 `sha256` 的摘要和这里的不一样。这是因为官方镜像是一直在维护的，有任何新的 bug，或者版本更新，都会进行修复再以原来的标签发布，这样可以确保任何使用这个标签的用户可以获得更安全、更稳定的镜像。

*如果从 Docker Hub 下载镜像非常缓慢，可以参照 [镜像加速器](https://docker-practice.github.io/zh-cn/install/mirror.html) 一节配置加速器。*

#### 运行

有了镜像，我们就可以以这个镜像为基础启动并运行一个容器。以上面的`ubuntu:18.04`为例，如果我们打算能启动里面的`bash`并进行交互操作的话，可以执行下面的命令。

```BASH
$ docker run -it --rm ubuntu:18.04 bash

root@e7009c6ce357:/# cat /etc/os-release
NAME="Ubuntu"
VERSION="18.04.1 LTS (Bionic Beaver)"
ID=ubuntu
ID_LIKE=debian
PRETTY_NAME="Ubuntu 18.04.1 LTS"
VERSION_ID="18.04"
HOME_URL="https://www.ubuntu.com/"
SUPPORT_URL="https://help.ubuntu.com/"
BUG_REPORT_URL="https://bugs.launchpad.net/ubuntu/"
PRIVACY_POLICY_URL="https://www.ubuntu.com/legal/terms-and-policies/privacy-policy"
VERSION_CODENAME=bionic
UBUNTU_CODENAME=bionic
```

`docker run` 就是运行容器的命令，具体格式我们会在 [容器](https://docker-practice.github.io/zh-cn/container)介绍中进行详细讲解，我们这里简要的说明一下上面用到的参数。

- `-it`：这是两个参数，一个是 `-i`：交互式操作，一个是 `-t` 终端。我们这里打算进入 `bash` 执行一些命令并查看返回结果，因此我们需要交互式终端。
- `--rm`：这个参数是说容器退出后随之将其删除。默认情况下，为了排障需求，退出的容器并不会立即删除，除非手动 `docker rm`。我们这里只是随便执行个命令，看看结果，不需要排障和保留结果，因此使用 `--rm` 可以避免浪费空间。
- `ubuntu:18.04`：这是指用 `ubuntu:18.04` 镜像为基础来启动容器。
- `bash`：放在镜像名后的是 **命令**，这里我们希望有个交互式 Shell，因此用的是 `bash`。

通过`docker` `exec`进入容器后，我们就可以在Shell下操作，执行任何所需的命令。如上述代码

修改容器中文件的内容可以通过 cat查看 sed修改

```SH
sed -i 's#Welcome to nginx#传智教育欢迎您#g' index.html
sed -i 's#<head>#<head><meta charset="utf-8">#g' index.html
```

**一般不在容器内修改容器内文件**

可以通过`exit`退出这个容器。

## 数据卷

容器与数据耦合的问题，数据卷则完美解决了这些问题。

[![image-20220821163050315](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208211631714.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208211631714.png)

**数据卷（volume）**：是一个虚拟目录，指向宿主机文件系统中的某个目录。

[![image-20220821163214609](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208211632700.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208211632700.png)

数据卷操作的基本语法如下：

```SH
docker volume [command]
```

docker volume命令是数据卷操作，根据命令后跟随command来确定下一步操作：

- create 创建一个volume
- inspect 显示一个或多个volume的信息
- ls 列出所有的volume
- prune 删除未使用的volume
- rm 删除一个或多个指定的volume

### 挂载数据卷

我们在创建容器时，可以通过 -v 参数来挂载一个数据卷到某个容器目录

[![image-20220821164126063](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208211641130.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208211641130.png)

> 如果运行容器时volume不存在，会被自动创建出来

### 挂载宿主机目录

提示：目录挂载与数据卷挂载的语法是类似的：

- -v [宿主机目录]:[容器内目录]
- -v [宿主机文件]:[容器内文件]

**数据卷挂载的方式对比**

[![image-20220821170135582](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208211701670.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208211701670.png)

**案例展示：**

```SH
docker run --name nginx -d -p 80:80 -v /home/docker/nginx/html:/usr/share/nginx/html -v /home/docker/nginx/conf/nginx.conf:/etc/nginx/nginx.conf nginx
```

### 总结

1. docker run的命令中通过 -v 参数挂载文件或目录到容器中：
   1. -v volume名称:容器内目录
   2. -v 宿主机文件:容器内文件
   3. -v 宿主机目录:容器内目录
2. 数据卷挂载与目录直接挂载的区别：
   1. 数据卷挂载耦合度低，由docker来管理目录，但是目录较深，不好找
   2. 目录挂载耦合度高，需要我们自己管理目录，不过目录容易寻找查看

## Dockerfile自定义镜像

### 镜像结构

- 镜像是将应用程序及其需要的系统函数库、环境、配置、依赖打包而成。

[![image-20220821180045751](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208211800881.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208211800881.png)

**镜像是分层结构，每一层称为一个Layer**

- BaseImage层：包含基本的系统函数库、环境变量、文件系统
- Entrypoint：入口，是镜像中应用启动的命令
- 其他：在BaseImage基础上添加依赖、安装程序、完成整个应用的安装和配置。

### Dockerfile

Dockerfile就是一个文本文件，其中包含一个个的**指令(Instruction)**，用指令来说明要执行什么操作来构建镜像。每一个指令都会形成一层Layer。

[![image-20220821180547123](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208211805206.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208211805206.png)

更新详细语法说明，请参考官网文档： https://docs.docker.com/engine/reference/builder

- Dockerfile的第一行必须是FROM，从第一个基础镜像来构建

- **基础镜像可以是基本操作系统，如Ubuntu。也可以是其他人制作好的镜像**，例如：java:8-alpine

  > 在我们构建java应用的镜像时，一般需要指定操作系统，安装JDK，将jar包拷贝到那个目录，端口，镜像启动的命令。**指定操作系统和安装JDK是避免不了且完全重复的部分**，所以我们借助其他制作好的镜像就可以大大减少Dockerfile的编写量，实际得到的结果也是相同的。

## DockerCompose

- Docker Compose可以基于Compose文件帮助我们快速的部署分布式应用，而无需手动一个个创建和运行容器！
- Compose文件是一个文本文件，通过指令定义集群中的每个容器如何运行。

案例：

[![image-20220821182438898](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208211824956.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208211824956.png)

- DockerCompose的详细语法参考官网：https://docs.docker.com/compose/compose-file/

> 帮助我们快速部署分布式应用，无需一个个微服务去构建镜像和部署。

### 安装

```SH
sudo yum install docker-ce docker-ce-cli containerd.io docker-compose-plugin
```

这条命令会安装*最新版本*的 Docker Engine、containerd 和 Docker Compose。

安装docker-compose工具库，安装后才能使用docker-compose的相关命令

```sh
sudo yum install docker-compose
```

### Docker Compose的命令补全配置

首先安装bash-completion

```SH
sudo yum install -y bash-completion
```

然后执行

```SH
echo “199.232.68.133 raw.githubusercontent.com” >> /etc/hosts 
```

最后执行

```SH
sudo curl -L https://raw.githubusercontent.com/docker/compose/1.29.1/contrib/completion/bash/docker-compose -o /etc/bash_completion.d/docker-compose
source /etc/bash_completion.d/docker-compose
```

完成配置

## Docker镜像仓库

镜像仓库（ Docker Registry ）有公共的和私有的两种形式：

- 公共仓库：例如Docker官方的 Docker Hub，国内也有一些云服务商提供类似于 Docker Hub 的公开服务，比如 网易云镜像服务、DaoCloud 镜像服务、阿里云镜像服务等。
- 除了使用公开仓库外，用户还可以在本地搭建私有 Docker Registry。企业自己的镜像最好是采用私有Docker Registry来实现。

使用DockerCompose文件来部署Docker私有仓库

### 简化版镜像仓库

搭建方式比较简单，命令如下：

```BASH
docker run -d --restart=always --name registry -p 5000:5000 -v registry-data:/var/lib/registry registry	
```

命令中挂载了一个数据卷registry-data到容器内的/var/lib/reigstry目录，这是私有镜像库存放数据的目录

访问http://yourip:5000/v2/catalog可以查看当前私有镜像服务中包含的镜像

### 带有图形化界面的界面版本

Docker官方的Docker Registry是一个基础版本的Docker镜像仓库，具备仓库管理的完整功能，但是没有图形化界面。

使用DockerCompose部署带有图像界面的DockerRegistry，命令如下：

```YML
version: '3.3'
services:
  registry:
    image: registry
    volumes:
      - ./registry-data:/var/lib/registry
  ui: #ui界面并非docker官方提供的 而是有其他开发人员提供,这里使用dockercompose将两者组合起来
    image: joxit/docker-registry-ui:static
    ports:
      - 8080:80
    environment:
      - REGISTRY_TITLE=Devildyw私有仓库
      - REGISTRY_URL=http://registry:5000
    depends_on:
      - registry
```

#### 配置Docker 信任地址

我们的私服采用的是http协议，默认不被Docker信任，所以需要做一个配置：

```sh
#打开要修改的文件
#添加内容
"insecure-registries":["http://121.37.27.207:8080"]
#重加载
systemctl daemon-reload
#重启docker
systemctl restart docker
```

配置完成后，编写docker-compose.yml文件，将如上的内容填写进去保存后，执行

```SH
docker-compose up -d 
```

[![image-20220821195137434](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208211951542.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208211951542.png)

### 在私有镜像仓库推送或拉取镜像

推送镜像到私有镜像服务必须先tag，步骤如下：

1. 重新tag本地镜像，名称前缀为私有仓库地址：121.37.27.207:8080/

   ```SH
   docker tag nginx:latest 121.37.27.207:8080/nginx:1.0
   ```

2. 推送镜像

   ```SH
   docker push 121.37.27.207:8080/nginx:1.0
   ```

3. 拉取镜像

   ```SH
   docker pull 121.37.27.207:8080/nginx:1.0
   ```