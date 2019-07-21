# ubuntu安装镜像
    卸载旧版本
        $ sudo apt-get remove docker \
    docker-engine \
    docker.io

    由于 apt 源使用 HTTPS 以确保软件下载过程中不被篡改。因此,我们首先需要
    添加使用 HTTPS 传输的软件包以及 CA 证书。
    $ sudo apt-get update
    $ sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    software-properties-common

    为了确认所下载软件包的合法性,需要添加软件源的 GPG 密钥。
    Ubuntu
    $ curl -fsSL https://mirrors.ustc.edu.cn/docker-ce/linux/ubuntu/
    gpg | sudo apt-key add -
    # 官方源
    # $ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | su
    do apt-key add -
    然后,我们需要向 source.list 中添加 Docker 软件源
    $ sudo add-apt-repository \
    "deb [arch=amd64] https://mirrors.ustc.edu.cn/docker-ce/linu
    x/ubuntu \
    $(lsb_release -cs) \
    stable"
    # 官方源
    # $ sudo add-apt-repository \
    # "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
    # $(lsb_release -cs) \
    # stable"

    安装 Docker CE
    更新 apt 软件包缓存,并安装 docker-ce :
    $ sudo apt-get update
    $ sudo apt-get install docker-ce
# centos7以下安装
    https://www.cnblogs.com/zhangzhen894095789/p/6641981.html?utm_source=itdadao&utm_medium=referral
# docker常用命令
    拉取image
        docker pull xxx
    展示image
        docker image ls
    展示虚悬image
        docker image ls -f dangling=true
    格式化展示image
        docker image ls --format "{{.Repository}}\t{{.Tag}}\t{{.ID}}\t{{.Size}}"

    删除镜像，删除本地镜像 $ docker image rm [选项] <镜像1> [<镜像2> ...]
        docker image rm ubuntu
    删除容器实例
        docker rm 1278cf0408f4
    查看摘要信息
        $ docker image ls --digests
    根据摘要删除image
        $ docker image rm ubuntu@sha256:7a47ccc3bbe8a451b500d2b53104868b46d60ee8f5b35a24b41a86077c650210
    通过-q过滤和rm组合删除image
        $ docker image rm $(docker image ls -q ubuntu)

# 关于镜像id,digests
    1. 镜像的唯一标识是其ID 和摘要,而一个镜像可以有多个标签。
    2. 定制镜像应该使用 Dockerfile 来完成

    3. 镜像是容器的基础,每次执行 docker run 的时候都会指定哪个镜像作为容器运
    行的基础。
    4. 镜像是多层存储,每一层是在前一层的基础上进行的修改;
    而容器同样也是多层存储,是在以镜像为基础层,在其基础上加一层作为
    容器运行时的存储层。

# 查看容器和停止容器
    heige@daheige:/etc/docker$ docker container ls
    CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                  NAMES
    4638aa78cdc0        8c9ca4d17702        "nginx -g 'daemon of…"   17 minutes ago      Up 17 minutes       0.0.0.0:8080->80/tcp   webserver
    heige@daheige:/etc/docker$ docker stop 4638aa78cdc0


# 运行一个webserver
    docker run --name webserver -d -p 8080:80 nginx
    端口映射8080:80 在容器中运行的port=80
    这条命令会用 nginx 镜像启动一个容器,name命名为 webserver ,并且映射了 8080
    端口,这样我们可以用浏览器去访问这个 nginx 服务器。
    http://localhost:8080/ 可以访问了

    列出容器
    $ docker container ls
    CONTAINER ID  IMAGE  COMMAND  CREATED  STATUS  PORTS  NAMES
    4638aa78cdc0        nginx               "nginx -g 'daemon of…"   About a minute ago   Up About a minute   0.0.0.0:8080->80/tcp   webserver
    
    现在,假设我们非常不喜欢这个欢迎页面,我们希望改成欢迎 Docker 的文字,我
    们可以使用 docker exec 命令进入容器,修改其内容。
    $ docker exec -it webserver bash
    root@4638aa78cdc0:/# echo "<h1>hello daheige</h1>" > /usr/share/nginx/html/index.html 
    root@4638aa78cdc0:/# exit
    exit
    当我们再次访问 localhost:8080
    我们以交互式终端方式进入 webserver 容器,并执行了 bash 命令,也就是获
    得一个可操作的 Shell。

    我们修改了容器的文件,也就是改动了容器的存储层。我们可以通过 docker
    diff 命令看到具体的改动。
    docker diff webserver
# 定制镜像
    Docker 提供了一个 docker commit 命令,可以将容器的存储层保存下来成为镜像。
    换句话说,就是在原有镜像的基础上,再叠利用 commit 理解镜像构成加上容器的存储层,并构成新的镜像。
    以后我们运行这个新镜像的时候,就会拥有
    原有容器最后的文件变化。
    定制镜像 docker commit
    $ docker commit --author "daheige <zhuwei313@hotmail.com>" --message "update nginx index.html" webserver nginx:v2
# 基于定制镜像运行一个容器
    $ docker run --name web2 -d -p 8082:80 nginx:v2 
    11bb8702f8d686bb48c1d839c6485692f0e2ab062cacac818ce97ace6dbe7428
    名字name为web2
    查看容器
    $ docker container ls
    CONTAINER ID        IMAGE               COMMAND                  CREATED              STATUS              PORTS                  NAMES
    11bb8702f8d6        nginx:v2            "nginx -g 'daemon of…"   About a minute ago   Up About a minute   0.0.0.0:8082->80/tcp   web2
    访问 http://localhost:8082/
    至此,我们第一次完成了定制镜像,使用的是 docker commit 命令,手动操作
    给旧的镜像添加了新的一层,形成新的镜像,对镜像多层存储应该有了更直观的感觉。

# 慎用 docker commit
    利用 commit 理解镜像构成
    使用 docker commit 命令虽然可以比较直观的帮助理解镜像分层存储的概念,
    但是实际环境中并不会这样使用。
    首先,如果仔细观察之前的 docker diff webserver 的结果,你会发现除了真
    正想要修改的 /usr/share/nginx/html/index.html 文件外,由于命令的执
    行,还有很多文件被改动或添加了。这还仅仅是最简单的操作,如果是安装软件
    包、编译构建,那会有大量的无关内容被添加进来,如果不小心清理,将会导致镜
    像极为臃肿。

    此外,使用 docker commit 意味着所有对镜像的操作都是黑箱操作,生成的镜
    像也被称为黑箱镜像,换句话说,就是除了制作镜像的人知道执行过什么命令、怎
    么生成的镜像,别人根本无从得知。而且,即使是这个制作镜像的人,过一段时间
    后也无法记清具体在操作的。虽然 docker diff 或许可以告诉得到一些线索,
    但是远远不到可以确保生成一致镜像的地步。这种黑箱镜像的维护工作是非常痛苦
    的。
    
    而且,回顾之前提及的镜像所使用的分层存储的概念,除当前层外,之前的每一层
    都是不会发生改变的,换句话说,任何修改的结果仅仅是在当前层进行标记、添
    加、修改,而不会改动上一层。如果使用 docker commit 制作镜像,以及后期
    修改的话,每一次修改都会让镜像更加臃肿一次,所删除的上一层的东西并不会丢
    失,会一直如影随形的跟着这个镜像,即使根本无法访问到。这会让镜像更加臃
    肿。