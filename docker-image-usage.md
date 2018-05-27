## Docker - 镜像的使用和管理

### 获取镜像
- docker 默认从 Docker Hub 上拉取镜像，在国内从 Docker Hub 上拉取镜像一般速度会很慢，我们可以配置加速器来提高镜像下载速度，具体加速器配置参见：[配置 docker 镜像加速器](./docker-image-accelerator-installation.md)
- 从 Docker 镜像仓库获取镜像的命令是 `docker pull`。其命令格式为：
  
  ```bash
  docker pull [选项] [Docker Registry 地址[:端口号]/]仓库名[:标签]
  ```
- 具体选项可以通过 `docker pull --help` 查看。
  
  > 这里的仓库名是两段式名称，即 <用户名>/<软件名>。对于 Docker Hub，如果不给出用户名，则默认为 library，也就是官方镜像。也可以通过其他镜像仓库的地址下载镜像。

- 当我们需要下载其他镜像的时候可以先通过如下命令进行搜索然后再用 `docker pull` 进行下载，默认是在 Docker Hub 上进行搜索：
  
  ```bash
  docker search <镜像名称>
  
  例如：
  docker search centos
  ```

### 列出镜像
- 列出本地镜像可以使用 `docker image ls` 命令，参考如下：
  
  ```bash
  [root@test-node-3 ~]# docker image ls
  REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
  docker.io/nginx     latest              ae513a47849c        3 weeks ago         109 MB
  docker.io/centos    latest              e934aafc2206        7 weeks ago         199 MB
  ```
  > 其实也可以用 `docker images` 查看本地镜像，建议用 `docker image ls` 。

