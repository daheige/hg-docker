# 删除已经退出的容器
     docker rm $(docker container ls -f "status=exited" -q)

# 查看所有的容器列表id
    docker container ls -aq

# 创建本地数据卷
        $ docker volume create -d local test
    查看数据卷信息
        $ docker volume inspect test
    绑定volume
        $ docker run -d -P --name web -v /webapp:/opt/webapp training/webapp python app.py

# 创建数据卷容器
        $ docker run -it -v /dbdata --name dbdata ubuntu
    运行容器，将数据卷绑定到容器上
        $ docker run -it --volumes-from dbdata --name db1 ubuntu
        docker run -it --volumes-from dbdata --name db2 ubuntu
        将两个容器运行的数据卷挂在同一个dbdata上
        在任意一个容器中创建一个文件，在db1,db2,dbdata中都可以看到
        root@fa13d30ebda2:/dbdata# touch test.md

        root@20899068911b:/dbdata# touch 1.txt
        root@20899068911b:/dbdata# ls
        1.txt  test.md
        root@20899068911b:/dbdata#

        对于--volumes-from参数可以一次性指定多个volume数据卷容器
        当删除了dbdata,db1,db2数据卷还在
        $ docker volume ls
        DRIVER              VOLUME NAME
        local               e59861d4449aeaaa78358c9

        将dbdata数据卷中的内容，迁移到本地~/mytest
        $ docker run --volumes-from dbdata -v $(pwd)/mytest:/backup --name worker ubuntu tar cvf /backup/backup.tar /dbdata
        tar: Removing leading `/' from member names
        /dbdata/
        /dbdata/test.md
        /dbdata/1.txt
        heige@daheige:~$ cd mytest/
        heige@daheige:~/mytest$ ll
        总用量 20
        drwxr-xr-x  2 heige heige  4096 7月  21 10:32 ./
        drwxr-xr-x 43 heige heige  4096 7月  21 10:31 ../
        -rw-r--r--  1 root  root  10240 7月  21 10:32 backup.tar
        创建一个容器worker然后指定volume是/backup目录，然后在容器中执行tar把/dbname的内容压缩到容器/backup
        而容器采用dbdata作为数据卷，就可以把/dbdata数据备份出来

        恢复
        先创建一个容器
            $ docker run -v /dbdata --name dbdata2 ubuntu /bin/bash
        恢复数据
        $ docker run --volumes-from dbdata2 -v $(pwd)/mytest:/backup busybox tar xvf /backup/backup.tar
        dbdata/
        dbdata/test.md
        dbdata/1.txt

# 清除无效的数据卷
    $ docker volume prune


#  端口映射访问容器
    -P 会随机启一个端口
    $ docker run -d -P training/webapp python app.py
    ff3a5c99916a362fe8b3c5f4358c4c00dd0b63b84a861c42a4e392e69230fa1b
    heige@daheige:~$ docker ps -l
    CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS                     NAMES
    ff3a5c99916a        training/webapp     "python app.py"     7 seconds ago       Up 4 seconds        0.0.0.0:32770->5000/tcp   youthful_lichterman
    查看发现32770->5000/tcp  32770已经映射到5000
    在浏览器中访问 http://localhost:32770/
    查看容器的日志 $ docker logs -f youthful_lichterman
    * Running on http://0.0.0.0:5000/ (Press CTRL+C to quit)
    172.17.0.1 - - [21/Jul/2019 02:49:49] "GET / HTTP/1.1" 200 -
    172.17.0.1 - - [21/Jul/2019 02:49:49] "GET /favicon.ico HTTP/1.1" 404 -
    172.17.0.1 - - [21/Jul/2019 02:49:55] "GET /fefe HTTP/1.1" 404 -
    172.17.0.1 - - [21/Jul/2019 02:49:58] "GET / HTTP/1.1" 200 -

    也可以使用-p将 hostPort:containPort进行映射
    $ docker run -d -p 5000:5000 training/webapp python app.py
    ab06e4c00743aeacc3488e593836afd03b7666c80f77e17e192c5a4ab5e1b518

    多个端口映射：
    多次使用-p标记可以绑定多个端口

    查看容器对应的端口映射
    $ docker port happy_shaw
    5000/tcp -> 0.0.0.0:5000

# 互联机制实现便捷互访
    1、自定义容器名称
        指定名称--name xxx
        $ docker run -d -p 5001:5000 --name web training/webapp python app.py
        3b70f2e707e6c16ac887fc163f85db4cab2cf94f6be2e4a3e2e192c76e1aa551
        查看容器状态
        $ docker ps -l
        $ docker inspect -f "{{ .Name }}" 3b70f2e707e6
        /web
    2、容器互联
        采用--link实现
        --link参数的格式为--link name: alias, 其中name是要链接的容器的名称 ,
        alias是别名

        建立容器数据库
        $ docker run -d --name db training/postgres #没有指定-p，没有暴露端口到外网
        删除之前的web
        docker rm -f web
        heige@daheige:~$ docker run -d -p 5002:5000 --name web --link db:db training/webapp python app.py
        8a7254f46625e37ab7e79f6fdfbe1643c2b28a7789f951f0738cec6967529610
        heige@daheige:~$ docker ps -l
        CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS                    NAMES
        8a7254f46625        training/webapp     "python app.py"     13 seconds ago      Up 10 seconds       0.0.0.0:5002->5000/tcp   web

        $ docker container ls
        CONTAINER ID        IMAGE               COMMAND                  CREATED              STATUS              PORTS                     NAMES
        8a7254f46625        training/webapp     "python app.py"          About a minute ago   Up About a minute   0.0.0.0:5002->5000/tcp    web
        245efd944744        training/postgres   "su postgres -c '/us…"   3 minutes ago        Up 3 minutes        5432/tcp                  db

        为容器公开连接信息
        1、通过env
        2、/etc/hosts
        $ docker run --rm --name web2 --link db:db training/webapp env
        PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
        HOSTNAME=c23c2323d4a5
        DB_PORT=tcp://172.17.0.5:5432
        DB_PORT_5432_TCP=tcp://172.17.0.5:5432
        DB_PORT_5432_TCP_ADDR=172.17.0.5
        DB_PORT_5432_TCP_PORT=5432
        DB_PORT_5432_TCP_PROTO=tcp
        DB_NAME=/web2/db
        DB_ENV_PG_VERSION=9.3
        HOME=/root

        其中DB_开头的，供web容器连接db时候所用
        除了环境变量,Docker还添加host信息到父容器的 /etc/host的文件。 下面是父容器web的host文件
        docker run -it --rm --link db:db training/webapp /bin/bash
        root@0f17cc19318b:/opt/webapp# cat /etc/hosts
        127.0.0.1	localhost
        ::1	localhost ip6-localhost ip6-loopback
        fe00::0	ip6-localnet
        ff00::0	ip6-mcastprefix
        ff02::1	ip6-allnodes
        ff02::2	ip6-allrouters
        172.17.0.5	db 245efd944744
        172.17.0.6	0f17cc19318b

        在容器中安装ping
        root@0f17cc19318b:/opt/webapp# apt-get install -yqq inetutils-ping
        root@0f17cc19318b:/opt/webapp# ping db
        PING db (172.17.0.5): 56 data bytes
        64 bytes from 172.17.0.5: icmp_seq=0 ttl=64 time=0.170 ms
        看到db解析到了172.17.0.5
        