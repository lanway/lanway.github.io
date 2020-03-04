### **Dockerfile书写规则**

------

- CMD 
  CMD指令用于指定一个容器启动时要运行的命令。当我们在使用docker 
  run命令时，会在完整的镜像之后指定容器启动时将要执行的命令，这个功能在Dockerfile文件中同样能够实现，一定要注意，该条指令指示的是在启动容器时将要执行的指令，和RUN命令的不同。语法格式采用的是docker
   exec使用的指令格式，比如

```
$ docker run --name "jpzhu/centos" -p localhost：8080:80  -it centos:7.2.1511 /bin/bash  #该条指令的含义是在启动容器时将要执行/bin/bash，相当于下面的Dockerfile文件11
```

```
#Dockerfile
FROM centos:7.2.1511
MAINTAINER jpzhu jpzhu.gm@gmail.com
CMD ["/bin/bash"]
EXPOSE 80 1234512345
```

```
$ docker build .  #使用上述Dockerfile构建镜像11
```

如果在启动容器时使用的命令是:

```
$ systemctl enable docker #此时在镜像中应该安装了docker，并且想要设置docker的开始自启动11
```

```
$ CMD ["systemctl","enable","docker"] #也就是说每个空格间隔的部分都套使用一个""将其设置为一个单元11
```

**需要注意的是：**要运行的命令是存放在一个数组结构中的。这将告诉[Docker](http://lib.csdn.net/base/docker)按指定的原样来运行该命令，当然也可以不使用数组来指定CMD命令，但是Docker官方一直推荐使用数组语法来设置要执行的命令。在Dockerfile中只能指定一条CMD指令，如果你指定了多条也只能有一条被执行。

- ENTRYPOINT 

  ​	ENTRYPOINT指令与CMD指令非常类似，也很容易与CMD指令混淆，这两个指令到底有什么样的区别呢？为什么要同时保留这两条指令呢？正如我们已经知道的那样，我们可以在使用docker
  run命令时覆盖dockerfile中使用的CMD指令，有时候我们希望容器能够按照我们想象的那样去工作，这时候使用CMD指令就不太合适了。而ENTRYPOINT指令提供的命令则不容易在启动容器时被覆盖，实际上，docker
   run命令行中指定的任何参数都会被当做参数再次传递给ENTRYPOINT指令中指定的命令。

- WORKDIR 

  ​	WORKDIR指令用来从镜像创建一个新容器时，在容器内部设置一个工作目录，ENTRYPOINT和CMD指定的程序会在这个目录中执行。我们可以使用该指令为Dockerfile中后续的一系列指令设置工作目录，也可以为最终的容器设置工作目录。

- ENV 

  ​	ENV指令用来在镜像的构建过程中设置环境变量，如ENV JAVA_HOME /usr/lib/jvm/bin，这和在[Linux](http://lib.csdn.net/base/linux)中设置环境变量，以及使用环境便令没什么区别。$JAVA_HOME指的是要引用/usr/lib/jvm/bin目录

- USER 

  ​	USER指令用来指定该镜像会以什么样的用户去运行。

- VOLUME 

  ​	VOLUME指令用来向基于镜像创建的容器添加卷。一个卷是可以存在于一个或者多个容器内的特定的目录，这个目录可以绕过联合文件系统，并提供如下共享数据或者对数据进行持久化的功能。

- ADD 

  ​	ADD指令用来将构建环境下（即dockerfile所在的目录下的文件）的文件和目录复制到镜像中，比如，在安装一个应用程序时，ADD命令需要源文件位置和目的文件位置两个参数。**注意：如果ADD在处理压缩文件时，同时会将压缩文件解压**。

- COPY 

  ​	COPY指令与ADD指令在功能上大体是相同的，都是为了将构建环境中的文件复制到镜像中，但是**COPY指令不会去做提取或者解压的操作**。上述两者的文件源路径必须是一个与当前构建环境相对的文件或者目录，本地文件都放到和Dockerfile同一个目录下，不能复制该目录之外的任何文件，因为构建环境将会上传到Docker守护进程，而复制是在Docker守护进程中进行的。

- ONBUILD 

  ​	ONBUILD指令能为镜像添加触发器，当一个镜像被用作其他镜像的基础镜像时，触发器会在构建过程中插入新指令，我们可以认为这些指令是紧跟在FROM之后指定的，触发器可以是任何的构建指令。ONBUILD不管卸载哪里，最后在处理时都会FORM之后被执行。