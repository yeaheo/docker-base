## Docker - 构建镜像

### 利用 commit 理解镜像构成

- 镜像是容器的基础，每次执行 docker run 的时候都会指定哪个镜像作为容器运行的基础。镜像是多层存储，每一层是在前一层的基础上进行的修改；而容器同样也是多层存储，是在以镜像为基础层，在其基础上加一层作为容器运行时的存储层。
- 当我们运行一个容器的时候（使用卷），我们做的任何文件修改都会被记录于容器存储层里。而 Docker 提供了一个 `docker commit` 命令，可以将容器的存储层保存下来成为镜像。换句话说，就是在原有镜像的基础上，再叠加上容器的存储层，并构成新的镜像。以后我们运行这个新镜像的时候，就会拥有原有容器最后的文件变化。
- `docker commit` 的语法格式为：
  
  ```bash
  docker commit [选项] <容器ID或容器名> [<仓库名>[:<标签>]]
  ```
- 例如，当我们修改了 nginx 镜像的时候可以用如下命令制作新镜像，这里只做参考：
  
  ```bash
  docker commit \
    --author "Lv Xiaoteng <helleo.cn@gmail.com>" \
    --message "修改了默认网页" \
    webserver \
    nginx:v2
  ```
- 其中 `--author` 是指定修改的作者，而 `--message` 则是记录本次修改的内容。这点和 `git` 版本控制相似，不过这里这些信息可以省略留空。

- 我们可以通过 `docker diff` 命令看到具体的改动，这里不再赘述。
- 实际状况下，我们制作镜像的时候不会用到 `docker commit` 而是用 dockerfile 的方式制作我们需要的镜像。 因为 `docker commit` 制作的镜像是在基本镜像的基础上直接保存容器存储层，这就造成了制作的镜像相当的臃肿，而且其他人也不知道镜像做了哪些更改，安装了哪些软件等等，可以说用 `docker commit` 制作的镜像是 **黑箱镜像** ，维护起来是相当的困难，所以我们不提倡用这种方式制作镜像，这里提到只是为了通过 `docker commit`了解镜像的构成。

### 利用 Dockerfile 制作镜像
- Dockerfile 其实是一种文本文件，其中包含了一条条指令，每个指令构建一层。因此每一条指令的内容，就是描述该层应当如何构建。
- Dockerfile 相关命令参见 [dockerfile-commands](./docker-dockerfile-commands.md)

