# 使用dockerfile构建镜像
    配置指令
        ENV 指定环境变量,运行时候可以用--env key=val key1=val1进行覆盖
        FROM 指定所创建镜像的基础镜像
        ARG 定义创建镜像过程中使用的变量
        LABEL 为生成的镜像添加元数据标签信息
        EXPOSE 声明镜像内服务监听的端口，只是声明，启动的时候需要-p进行端口映射
        ENTRYPOINT 指定镜像的默认入口命令,可作为启动容器时候作为根命令执行,每个dockerfile只能有一个
        VOLUME 创建一个数据卷挂载点
        USER 指定运行容器时候的用户或uid
        WORKDIR 配置工作目录
                为后续的 RUN,CMD,ENTRYPOINT指令配置工作目录
        ONBUILD 创建子镜像时候，指定自动执行的操作指令
        STOPSIGNAL 指定退出信号量值
        HEALTHCHECK 配置健康检查
        SHELL 指定默认shell类型
    操作指令
        RUN   运行指定指令
            每条 RUN 指令将在当前镜像基础上执行指定命令,并提交为新的镜像层 。 当命令较长时
            可以使用\来换行
        CMD   启动容器时候指定默认执行的命令
            CMD [executable ",” paraml ” , " param2 '『]
            相当于执行 executable param1,param2 ,推荐方式
            每个 Dockerfile 只能有一条 CMD 命令 。 如果指定了多条命令,只有最后一条会被执行
        ADD   添加内容到镜像中
        COPY  复制内容到镜像中
              目标路径不存在时,会自动创建
              COPY 与 ADD 指令功能类似,当使用本地目录为源目录时,推荐使用 COPY

# golang 构建
    参考my-go