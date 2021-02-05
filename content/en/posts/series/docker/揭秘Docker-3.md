---
title: "[Docker详解]-3:揭秘Dockerfile"
date: 2021-01-27 17:13:12
description: "详解Dockerfile"
tags:
-
series:
- DOCKER
categories:
- Frame

image: images/feature1/graph.png
---



### Dockerfile 指令详解



* Dockerfie 官方文档：https://docs.docker.com/engine/reference/builder/

* Dockerfile 最佳实践文档：https://docs.docker.com/develop/develop-images/dockerfile_best-practices/

* Docker 官方镜像 Dockerfile：https://github.com/docker-library/docs



#### COPY 复制文件



* COPY [--chown=<user>:<group>] <源路径>... <目标路径>
* COPY [--chown=<user>:<group>] ["<源路径1>",... "<目标路径>"]



COPY 指令将从构建上下文目录中 <源路径> 的文件/目录复制到新的一层的镜像内的 <目标路径> 位置。比如：

```bash
COPY package.json /usr/src/app/
```

*<目标路径>*可以是容器内的绝对路径，也可以是相对于工作目录的相对路径（工作目录可以用 WORKDIR 指令来指定）。目标路径不需要事先创建，如果目录不存在会在复制文件前先行创建缺失目录。





#### ADD 更高级的复制文件



ADD 指令和 COPY 的格式和性质基本一致。但是在 COPY 基础上增加了一些功能。

最适合使用 ADD 的场合，就是所提及的需要自动解压缩的场合。

因此在 COPY 和 ADD 指令中选择的时候，可以遵循这样的原则，所有的文件复制均使用 COPY 指令，仅在需要自动解压缩的场合使用 ADD。



#### CMD 容器启动命令



> Docker 不是虚拟机，容器就是进程。既然是进程，那么在启动容器的时候，需要指定所运行的程序及参数。CMD 指令就是用于指定默认的容器主进程的启动命令的。
>
> CMD 指令的首要目的在于为启动的容器指定默认要运行的程序，程序运行结束，容器也就结束；注意: CMD 指令指定的程序可被 docker run 命令行参数中指定要运行的程序所覆盖。

类似于 RUN 指令，用于运行程序，但二者运行的时间点不同；CMD 在docker run 时运行，而非docker build;



#### ENV 设置环境变量



* ENV <key> <value>
* ENV <key1>=<value1> <key2>=<value2>...



#### VOLUME 定义匿名卷



之前我们说过，容器运行时应该尽量保持容器存储层不发生写操作，对于数据库类需要保存动态数据的应用，其数据库文件应该保存于卷(volume)中，后面的章节我们会进一步介绍 Docker 卷的概念。为了防止运行时用户忘记将动态文件所保存目录挂载为卷，在 Dockerfile 中，我们可以事先指定某些目录挂载为匿名卷，这样在运行时如果用户不指定挂载，其应用也可以正常运行，不会向容器存储层写入大量数据。



```bash
VOLUME /data
```

这里的 /data 目录就会在运行时自动挂载为匿名卷，任何向 /data 中写入的信息都不会记录进容器存储层，从而保证了容器存储层的无状态化。当然，运行时可以覆盖这个挂载设置。比如：



#### WORKDIR 指定工作目录



使用 WORKDIR 指令可以来指定工作目录（或者称为当前目录），以后各层的当前目录就被改为指定的目录，如该目录不存在，WORKDIR 会帮你建立目录。



```bash
RUN cd /app
RUN echo "hello" > world.txt
```


如果将这个 Dockerfile 进行构建镜像运行后，会发现找不到 /app/world.txt 文件，或者其内容不是 hello。原因其实很简单，在 Shell 中，连续两行是同一个进程执行环境，因此前一个命令修改的内存状态，会直接影响后一个命令；而在 Dockerfile 中，这两行 RUN 命令的执行环境根本不同，是两个完全不同的容器。这是因为 Dockerfile 构建是分层存储的



> 之前说过每一个 RUN 都是启动一个容器、执行命令、然后提交存储层文件变更。第一层 RUN cd /app 的执行仅仅是当前进程的工作目录变更，一个内存上的变化而已，其结果不会造成任何文件变更。而到第二层的时候，启动的是一个全新的容器，跟第一层的容器更完全没关系，自然不可能继承前一层构建过程中的内存变化。
>
> 因此如果需要改变以后各层的工作目录的位置，那么应该使用 WORKDIR 指令。



```dock
WORKDIR /app

RUN echo "hello" > world.txt
```

如果你的 WORKDIR 指令使用的相对路径，那么所切换的路径与之前的 WORKDIR 有关：



```dockerfile
WORKDIR /a
WORKDIR b
WORKDIR c

RUN pwd
```

pwd 输出的结果为 /a/b/c。



#### ARG 构建参数

构建参数和 ENV 的效果一样，都是设置环境变量。所不同的是，ARG 所设置的构建环境的环境变量，在将来容器运行时是不会存在这些环境变量的。但是不要因此就使用 ARG 保存密码之类的信息，因为 docker history 还是可以看到所有值的。

Dockerfile 中的 ARG 指令是定义参数名称，以及定义其默认值。该默认值可以在构建命令 docker build 中用 --build-arg <参数名>=<值> 来覆盖。