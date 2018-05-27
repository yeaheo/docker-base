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

- 列出的镜像一般包括：仓库名、标签、ID、创建时间及镜像大小。
- **镜像 ID** 是镜像的唯一标识，但是一个镜像可以对应多个标签。
- 另外一个需要注意的问题是，`docker image ls` 列表中的镜像体积总和并非是所有镜像实际硬盘消耗。由于 Docker 镜像是多层存储结构，并且可以继承、复用，因此不同镜像可能会因为使用相同的基础镜像，从而拥有共同的层。由于 Docker 使用 Union FS，相同的层只需要保存一份即可，因此实际镜像硬盘占用空间很可能要比这个列表镜像大小的总和要小的多。

- 可以通过如下命令查看镜像、容器和数据卷所占用的空间大小：

  ```bash
  [root@test-node-3 ~]# docker system df
  TYPE                TOTAL               ACTIVE              SIZE                RECLAIMABLE
  Images              2                   0                   307.6 MB            307.6 MB (100%)
  Containers          0                   0                   0 B                 0 B
  Local Volumes       0                   0                   0 B                 0 B
  ```

#### 虚悬镜像
- 虚悬镜像可以这样理解，虚悬镜像其实是一种特殊的镜像，它的仓库名和标签皆为 `<none>` 。
- 查看虚悬镜像可以参考如下命令：
  
  ```bash
  docker image ls -f dangling=true
  ```
- 一般来说，虚悬镜像已经失去了存在的价值，是可以随意删除的，可以用下面的命令删除：

  ```bash
  docker image prune
  ```

#### 中间层镜像
- 为了加速镜像构建、重复利用资源，Docker 会利用 中间层镜像。所以在使用一段时间后，可能会看到一些依赖的中间层镜像。默认的 `docker image ls` 列表中只会显示顶层镜像，如果希望显示包括中间层镜像在内的所有镜像的话，需要加 `-a` 参数。
  
  ```bash
  [root@test-node-3 ~]# docker image ls -a
  REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
  docker.io/nginx     latest              ae513a47849c        3 weeks ago         109 MB
  docker.io/centos    latest              e934aafc2206        7 weeks ago         199 MB
  ```

  > 上面输出结果并没有中间层镜像,这是因为我们本地镜像较少，如果镜像较多的话可以看到好多无标签的镜像，与之前的虚悬镜像不同，这些无标签的镜像很多都是中间层镜像，是其它镜像所依赖的镜像。这些无标签镜像不应该删除，否则会导致上层镜像因为依赖丢失而出错。而且这些镜像一般都是复用的，也没必要删除。


#### 输出指定镜像信息
- 在大多数情况下，我们需要按照自己的想法列出我们需要的镜像信息，这个时候我们就要用到更加完善的功能了。

- **根据仓库名列出镜像**
  
  ```bash
  [root@test-node-3 ~]# docker image ls centos
  REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
  docker.io/centos    latest              e934aafc2206        7 weeks ago         199 MB
  ```
- 类似的还可以指定仓库名和标签，列出我们需要的镜像信息，只需在上述基础上加上标签信息即可，这里不再赘述。
- 我们也可以通过 `--filter` （简写成 `-f` ）参数列出我们需要的镜像，例如列出在 `centos:latest` 之后建立的镜像：
  
  ```bash
  [root@test-node-3 ~]# docker image ls --filter since=centos:latest
  REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
  docker.io/nginx     latest              ae513a47849c        3 weeks ago         109 MB
  ```
- 想查看某个位置之前的镜像也可以，只需要把 since 换成 before 即可。同时也可以通过 LABEL 来过滤。

- **以特定格式显示**
- 默认情况下，`docker image ls` 会输出一个完整的表格，但是我们并非所有时候都会需要这些内容。比如，刚才删除虚悬镜像的时候，我们需要利用 `docker image ls` 把所有的虚悬镜像的 ID 列出来，然后才可以交给 `docker image rm` 命令作为参数来删除指定的这些镜像，这个时候就用到了 `-q` 或 `--quiet` 参数。
- `--quiet` 或 `-q` 参数可以只列出镜像的 ID：
  
  ```bash
  [root@test-node-3 ~]# docker image ls -q
  ae513a47849c
  e934aafc2206
  ```

- 有时候我们可能只是对表格的结构不满意，希望自己组织列；或者不希望有标题，这样方便其它程序解析结果等，这就用到了 Go 的模板语法。
- 比如，下面的命令会直接列出镜像结果，并且只包含镜像ID和仓库名：

  ```bash
  [root@test-node-3 ~]# docker image ls --format "{{.ID}}: {{.Repository}}"
  ae513a47849c: docker.io/nginx
  e934aafc2206: docker.io/centos
  ```
- 有时候需要表格等距显示，并且有标题行，和默认一样，不过自己定义列：

  ```bash
  [root@test-node-3 ~]# docker image ls --format "table {{.ID}}\t {{.Repository}}\t {{.Tag}}"
  IMAGE ID            REPOSITORY          TAG
  ae513a47849c         docker.io/nginx     latest
  e934aafc2206         docker.io/centos    latest
  ```

### 删除镜像
- 如果要删除本地的镜像，可以使用 `docker image rm` 命令，其格式为：
  
  ```bash
  docker image rm [选项] <镜像1> [<镜像2> ...]
  ```

  > 删除镜像也可以用 `docker rmi` 命令，建议用 `docker image rm`

- **用 ID、镜像名、摘要删除镜像**
- 其中，<镜像> 可以是 镜像短 ID、镜像长 ID、镜像名 或者 镜像摘要。
- `docker image ls` 默认列出的就已经是短 ID 了，一般取前3个字符以上，只要足够区分于别的镜像就可以了。
  
  ```bash
  [root@test-node-3 ~]# docker image rm -f e93
  Untagged: docker.io/centos:latest
  Untagged: nginx:v2.0
  Deleted: sha256:e934aafc22064b7322c0250f1e32e5ce93b2d19b356f4537f5864bd102e8531f
  Deleted: sha256:43e653f84b79ba52711b0f726ff5a7fd1162ae9df4be76ca1de8370b8bbf9bb0
  ```
 
  > `-f` 选项表示强制删除。

- 我们也可以用镜像名，也就是 <仓库名>:<标签>，来删除镜像。

- **更精确的是使用 镜像摘要 删除镜像。**
  
  ```bash
  [root@test-node-3 ~]# docker image ls --digests 
  REPOSITORY          TAG                 DIGEST                                                                    IMAGE ID            CREATED             SIZE
  docker.io/nginx     latest              sha256:0fb320e2a1b1620b4905facb3447e3d84ad36da0b2c8aa8fe3a5a81d1187b884   ae513a47849c        3 weeks ago         109 MB

  [root@test-node-3 ~]# docker image rm docker.io/nginx@sha256:0fb320e2a1b1620b4905facb3447e3d84ad36da0b2c8aa8fe3a5a81d1187b884
  Untagged: docker.io/nginx@sha256:0fb320e2a1b1620b4905facb3447e3d84ad36da0b2c8aa8fe3a5a81d1187b884
  ```
- **结合 docker image ls 命令删除镜像**
- 有时候我们需要批量删除镜像，这个时候我们可以用 `docker image rm` 配合 `docker image ls` 来实现。具体使用方法参考如下：
  
  ```bash
  docker image rm $(docker image ls -q nginx)

  docker image rm $(docker image ls -q -f before=nginx:latest)
  ```

#### Untagged 和 Deleted
- 本地镜像的删除行为分为两类，一类是 Untagged，另一类是 Deleted。我们之前介绍过，镜像的唯一标识是其 ID 和摘要，而一个镜像可以有多个标签。
- 当我们使用上述命令删除镜像的时候，实际上是在删除某个标签的镜像。所以需要做的是先将满足要求的所有镜像标签都取消，这就是我们看到的 `Untagged` 。 一个镜像可以对应多个标签，因此当我们删除了所指定的标签后，可能还有别的标签指向了这个镜像，如果是这种情况，那么 Delete 行为就不会发生。所以并非所有的 `docker image rm` 都会产生删除镜像的行为，有可能仅仅是取消了某个标签而已。

- 如果镜像所有的标签都被取消了，该镜像很可能会失去了存在的意义，因此会触发删除行为。
- 镜像是多层存储结构，因此在删除的时候也是从上层向基础层方向依次进行判断删除。当有其他的容器或者镜像在依赖将要删除的镜像的时候，该镜像也不会被删除。







