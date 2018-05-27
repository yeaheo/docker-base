## Docker 容器三大概念

- Docker 容器有三个基本组件，分别是：

  ```bash
  镜像（image）
  容器（container）
  仓库（repository）
  ```

- 我们需要掌握这三个基本组件才能更好的利用 Docker 容器。

### Docker 镜像
- Docker 镜像其实就是一种特殊的文件系统。 操作系统分为内核和用户空间。对于 Linux 而言，内核启动后会挂载 `root` 文件系统为其提供用户空间支持。而 Docker 镜像就相当于是一个 `root` 文件系统。 例如官方镜像 `ubuntu:16.04` 就包含了完整的一套 `Ubuntu 16.04` 最小系统的 `root` 文件系统。 Docker 镜像除了提供容器运行时所需的程序、 库、 资源、 配置等文件外，还包含了一些为运行时准备的一些配置参数（如匿名卷、环境变量、用户等）。镜像不包含任何动态数据，其内容在构建之后也不会被改变。

#### Docker 镜像的分层存储
- 我们在简单介绍 Docker 容器的时候提到过，容器镜像是分层存储的，有些存储层可以被多个镜像同时复用。 Docker 镜像在设计的时候就依赖于 [Union FS](https://en.wikipedia.org/wiki/Union_mount) 技术，将其设计为分层存储的架构。镜像只是一个虚拟的概念，其实际体现并非由一个文件组成，而是由一组文件系统组成，也可以说，是由多层文件系统联合组成。

- 当我们在构建镜像的时候会一层层构建，前一层是后一层的基础。每一层构建完就不会再发生改变，后一层上的任何改变只发生在自己这一层。在构建镜像的时候，需要额外小心，每一层尽量只包含该层需要添加的东西，任何额外的东西应该在该层构建结束前清理掉。

- 分层存储的特征还使得镜像的复用、定制变的更为容易。甚至可以用之前构建好的镜像作为基础层，然后进一步添加新的层，以定制自己所需的内容，构建新的镜像。

- 关于镜像构建以后会持续更新，本文档只做简单介绍。

### Docker 容器
- Docker 容器就是 Docker 镜像的运行实例。用户可以通过 CLI（docker）或是 API 启动、停止、移动或删除容器。可以这么认为，对于应用软件，镜像是软件生命周期的构建和打包阶段，而容器则是启动和运行阶段。

- 容器的实质其实是进程，但与直接在宿主执行的进程不同，容器进程运行于属于自己的独立的命名空间。因此容器可以拥有自己的 root 文件系统、自己的网络配置、自己的进程空间，甚至自己的用户 ID 空间。

- 我们知道镜像使用的是分层存储架构，其实容器也是这样的。每一个容器运行时，是以镜像为基础层，在其上创建一个当前容器的存储层，我们可以称这个为容器运行时读写而准备的存储层为容器存储层。容器存储层的生存周期和容器一样，容器被删除后，容器存储层也被消除。因此，任何保存于容器存储层的信息都会随容器删除而丢失。所以一般容器不应该向其存储层内写入任何数据，容器存储层要保持无状态化。所有的文件写入操作，都应该使用 数据卷（Volume）、或者挂载到宿主机目录，在这些位置的读写会跳过容器存储层，直接对宿主（或网络存储）发生读写，其性能和稳定性更高。数据卷的生存周期独立于容器，容器消亡，数据卷不会消亡。因此，使用数据卷后，容器删除或者重新运行之后，数据却不会丢失。

### Docker 仓库
- Docker Registry 是存放 Docker 镜像的仓库，Registry 分私有和公有两种。

#### 公有 Docker Registry
- [Docker Hub](https://hub.docker.com/) 是默认的 Registry，由 Docker 公司维护，上面有数以万计的镜像，用户可以自由下载和使用。除此以外，还有 CoreOS 的 [Quay.io](https://quay.io/repository/)，CoreOS 相关的镜像存储在这里；Google 的 [Google Container Registry](https://cloud.google.com/container-registry/)，[Kubernetes](https://kubernetes.io/) 的镜像默认使用的就是这个服务。

- 由于某些原因，在国内访问这些服务可能会比较慢甚至根本访问不到。还好国内的一些云服务商提供了针对 Docker Hub 的镜像服务（Registry Mirror），这些镜像服务被称为加速器。常用的镜像加速器有 [阿里云镜像加速器](https://account.aliyun.com/login/login.htm?oauth_callback=https%3A%2F%2Fcr.console.aliyun.com%2F#/accelerator)、[DaoCloud 镜像加速器](https://www.daocloud.io/mirror#accelerator-doc)、[Docker Hub官方中国镜像加速器](https://docs.docker.com/registry/recipes/mirror/#use-case-the-china-registry-mirror)。使用加速器会直接从国内的地址下载 Docker Hub 的镜像，比直接从 Docker Hub 下载速度会提高很多。

- 国内一些云服务商也提供镜像仓库服务，常用的有 [阿里云镜像仓库](https://account.aliyun.com/login/login.htm?oauth_callback=https%3A%2F%2Fcr.console.aliyun.com%2F)、 [时速云镜像仓库](https://hub.tenxcloud.com/)、 [网易云镜像仓库](https://id.163yun.com/login?h=fc&referrer=https%3a%2f%2fc.163yun.com%2flogin%2fcallback%3fredirect%3d#/m/library/)、 [Daoloud 镜像仓库](https://hub.daocloud.io/)。

- 具体镜像加速器配置参见 [Docker 配置镜像加速器](./docker-image-accelerator-installation.md)

#### 私有 Docker Registry
- 除了使用公开服务外，用户还可以在本地搭建私有 Docker Registry。Docker 官方提供了 [Docker Registry](https://store.docker.com/images/registry/) 镜像，可以直接使用做为私有 Registry 服务。

- 开源的 Docker Registry 镜像只提供了 Docker Registry API 的服务端实现，足以支持 docker 命令，不影响使用。但不包含图形界面，以及镜像维护、用户管理、访问控制等高级功能。我们建议用 VMware 的产品 [vmware harbor](https://github.com/vmware/harbor),该产品具有比较友好的图形界面，而且支持 LDAP 认证。

- 利用 VMware Harbor 搭建私有 Docker 镜像仓库参见 [harbor 私有镜像仓库部署](https://github.com/yeaheo/kubernetes-deploy/blob/master/harbor-installation.md)
