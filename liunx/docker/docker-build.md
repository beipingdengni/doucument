## 基础操作

#### dockerfile

> 构建时使⽤.dockerignore ⽂件

```shell
FROM <image>:<tag> [as other_name]      # tag可选；不写默认是latest版
# as other_name是可选的，通常用于多阶段构建（有利于减少镜像大小）
# 使用是通过--from other_name使用,例如COPY --from other_name

LABEL author="tbp<test@qq.com>"
LABEL describe="test image"
# 或
LABEL author="tbp<test@qq.com>" describe="test image"
# 或
LABEL author="tbp<test@qq.com>" \
      describe="test image"

# 作者
MAINTAINER tbp <test@qq.com>

# 从构建主机复制文件到镜像中, <dest>如果不存在会自动创建，包含其父目录路径也会被创建
COPY <src> <dest>
COPY ["<src>", "<src>", ... "<dest>"]
# 拷贝一个文件
COPY testFile /opt/
# 拷贝一个目录
COPY testDir /opt/testDir # testDir下所有文件和目录都会被递归复制,  目标路径要写testDir，否则会复制到/opt下

# ADD：从构建宿主机复制文件到镜像中
ADD <src> <dest>
ADD ["<src>","<src>"... "<dest>"] # 如果是一个压缩文件(tar)，被被解压为一个目录，如果是通过URL下载一个文件不会被解压

# WORKDIR：设置工作目录,  类似于cd命令，为了改变当前的目录域
WORKDIR /opt   # 如果设置的目录不存在会自动创建，包括他的父目录
# 一个Dockerfile中WORKDIR可以出现多次，其路径也可以为相对路径,相对路径是基于前一个WORKDIR路径
# WORKDIR也可以调用ENV指定的变量

# ENV：设置镜像中的环境变量
# 一次设置一个
ENV <key> <value>
# 一次设置多个
ENV <key>=<value> <key1>=<value1> <key2>=<value2> .....
	#	使用环境变量的方式
	# $varname
	# ${varname}
	# ${varname:-default value}           # 设置一个默认值，如果varname未被设置，值为默认值
	# ${varname:+default value}           # 设置默认值；不管值存不存在都使用默认值

# USER：设置启动容器的用户
# 使用用户名
USER testuser
# 使用用户的UID
USER UID

# RUN：镜像构建时执行的命令
# 语法1，shell 形式
RUN command1 && command2
# 语法2，exec 形式
RUN ["executable","param1","[aram2]"]
# 示例
RUN echo 1 && echo 2 
RUN echo 1 && echo 2 \
    echo 3 && echo 4
RUN ["/bin/bash","-c","echo hello world"]
 # RUN 在下一次建构期间，会优先查找本地缓存，若不想使用缓存可以通过--no-cache解除
 # docker build --no-cache
 
# EXPOSE：为容器打开指定的监听端口以实现与外部通信
EXPOSE <port>/<protocol>  # 默认TCP协议，tcp/udp/http/https
EXPOSE 80
EXPOSE 80/http
EXPOSE 2379/tcp

# VOLUME：实现挂载功能，将宿主机目录挂载到容器中
VOLUME ["/data"]                    # [“/data”]可以是一个JsonArray ，也可以是多个值
VOLUME /var/log 
VOLUME /var/log /opt

# CMD：为容器设置默认启动命令或参数
# 语法1，shell形式
CMD command param1 param2 ...  # shell形式，默认/bin/sh -c ,进程在容器中的 PID != 1，这意味着该进程并不能接受到外部传入的停止信号docker stop
# 语法2，exec形式
CMD ["executable","param1","param2"] # 手动启动CMD ["/bin/sh","-c","executable","param1"...]
# 语法3,还是exec形式，不过仅设置参数
CMD ["param1","param2"] # 一般结合ENTRYPOINT指令使用

# ENTRYPOINT：用于为容器指定默认运行程序或命令
	# 与CMD类似，但存在区别，主要用于指定启动的父进程，PID=1, 
		# ENTRYPOINT设置默认命令不会被docker run命令行指定的参数覆盖，指定的命令行会被当做参数传递给ENTRYPOINT指定的程序。
		# docker run命令的 --entrypoint选项可以覆盖ENTRYPOINT指令指定的程序
		# 一个Dockerfile中可以有多个ENTRYPOINT，但只有最后一个生效
		# ENTRYPOINT主要用于启动父进程，后面跟的参数被当做子进程来启动
# 语法1，shell形式
ENTRYPOINT command
# 语法2，exec形式
ENTRYPOINT ["/bin/bash","param1","param2"]

=========================================================================
# ARG：指定环境变量用于构建过程
ARG name[=default value]
# 参考
ARG test_name
ARG nother_name=wzp
# docker build --build-arg test_name=test

# ONBUILD：为镜像添加触发器
ONBUILD <dockerfile_exec> <param1> <param2>
ONBUILD RUN mkdir mydir
# 该指令，对于使用该Dockerfile构建的镜像并不会生效，只有当其他Dockerfile以当前镜像作为基础镜像时被触发
# 例如：Dockfile A 构建了镜像A，Dockfile B中设置FROM A，此时构建镜像B是会运行ONBUILD设置的指令

# STOPSINGAL：设置停止时要发送给PID=1进程的信号（默认是10s）
	# 默认的停止信号为：SIGTERM，也可以通过docker run -s指定
	
# HEALTHCHECK：指定容器健康检查命令
	# 当在一个镜像指定了 HEALTHCHECK 指令后，用其启动容器，初始状态会为 starting，在 HEALTHCHECK 指令检查成功后变为 healthy，如果连续一定次数失败，则会变为 unhealthy
HEALTHCHECK [OPTIONS] CMD command
# 示例
HEALTHCHECK --interval=5s --timeout=3s \
    CMD curl -fs http://localhost/ || exit 1        # 如果执行不成功返回1
# 出现多次，只有最后一次生效
# OPTIONS选项
	# --interval=30：两次健康检查的间隔，默认为 30 秒；
	# --timeout=30：健康检查命令运行的超时时间，超过视为失败，默认30秒；
	# --retries=3：指定失败多少次视为unhealth，默认3次
# 返回值
	# 0：成功； 1：失败； 2：保留

# SHELL：指定shell形式的默认值
# SHELL 指令可以指定 RUN、ENTRYPOINT、CMD 指令的 shell，Linux 中默认为["/bin/sh", "-c"] ，Windows默认["CMD","/S","/C"]
SHELL ["/bin/bash","-c"]
SHELL ["powershell", "-command"]
# 示例，比如在Windows时，默认shell是["CMD","/S","/C"]
RUN powershell -command Execute-MyCmdlet -param1 "c:\foo.txt"
# docker调用的是cmd /S /C powershell -command Execute-MyCmdlet -param1 "c:\foo.txt"
RUN ["powershell", "-command", "Execute-MyCmdlet", "-param1 \"c:\\foo.txt\""]
# 这样虽然没有调用cmd.exe 但写起来比较麻烦，所以可以通过SHELL  ["powershell", "-command"] 这样省去前边的powershell -command
```

## demo 演示

```dockerfile
# 通过centos7的二进制包（假设是我自己做了系统裁剪移植）
FROM scratch
ADD centos-7-x86_64-docker.tar.xz /

RUN yum install -y wget && \
    yum clean all && \
    ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && \
    echo "Asia/Shanghai" > /etc/timezone

ENV JAVA_VERSION jdk1.8.0_271
ENV JAVA_HOME /usr/lib/${JAVA_VERSION}
ENV PATH ${JAVA_HOME}/bin:$PATH

# 涉及改变镜像大小的指令，尽量放到同一行，这样构建过程中的删除指令对减小体积才能生效
RUN wget --no-cookies --no-check-certificate --header "Cookie: gpw_e24=http%3A%2F%2Fwww.oracle.com%2F; oraclelicense=accept-securebackup-cookie" \
    http://download.oracle.com/otn-pub/java/jdk/8u271-b09/61ae65e088624f5aaa0b1d2d801acb16/jdk-8u271-linux-x64.tar.gz -O /usr/lib/jdk-8u271-linux-x64.tar.gz && \
    tar -xf /usr/lib/jdk-8u271-linux-x64.tar.gz -C /usr/lib/ && \
    rm -rf /usr/lib/jdk-8u271-linux-x64.tar.gz && \
    rm -rf ${JAVA_HOME}/src.zip \
           ${JAVA_HOME}/lib/visualvm \
           ${JAVA_HOME}/jre/lib/plugin.jar \
           ${JAVA_HOME}/jre/bin/javaws \
           ${JAVA_HOME}/jre/lib/desktop \
           ${JAVA_HOME}/jre/plugin \
           ${JAVA_HOME}/jre/lib/deploy* \
           ${JAVA_HOME}/jre/lib/amd64/libglass.so \
           ${JAVA_HOME}/jre/lib/amd64/libgstreamer-lite.so \
           ${JAVA_HOME}/jre/lib/amd64/libjavafx*.so \
           ${JAVA_HOME}/jre/lib/amd64/libjfx*.so

CMD ['/bin/bash']
```