## 概念

docker 容器是一种特殊的进程

使用了各种Namespace修改进程视图和 Cgroups约束的一种特殊进程



### Namespace

Namespace技术实际上是修改应用进程看待整个计算机的“视图”， 即ta的“视线”被操作系统做了限制，只能看到某些特定的内容。

几种特殊的namespace

1. PID Namespace: 屏蔽其他进程pid，并设置进程pid=1

2. Mount Namespace: 让进程只看到当前的挂载消息， 利用mount挂载不同操作系统的文件(ubuntu, centos等)。

3. Network Namespace: 让进程只看到当前的网络设备和配置

一个进程，可以选择加入到某个进程已有的 Namespace，这也是docker exec 的实现原理

## Cgroups

cgroups 全称Linux Control Group， 它最主要的作用就是限制一个进程组能够使用的资源(cpu, 内存, 磁盘, 网络带宽等)上限。



## chroot 命令

chroot 命令作用就是change root file system， 即改变进程的根目录到指定的目录。这样使用ls / 则只能看到指定目录下的文件。Mount Namespace 正是基于对 chroot 的不断改良出来的。



## AuFS

AuFS 全称为Advance Union FileSystem， 容器内部使用的就是AuFS。

容器中的文件系统是一个分层架构，由上到下一共三层: 

1. 读写层: 初始化为空，每次对docker的修改都以增量的形式写入读写层
2. Init层: docker内置的一层，需要修改，但是不保存的文件放在这层，如: /etc/hosts、/etc/resolv.conf等信息
3. 只读层: 容器操作系统的基本文件系统就是放在这层，不可修改，因此该层对于整个操作系统相同容器镜像而言是共享的



文件由上至下查找，上层文件会覆盖下层文件。删除文件，则是在上层生成一个whiteout（.wh.filename）的文件，两层联合后会屏蔽whiteout文件，造成文件被删除的假象。因此即使是在容器中删除文件，容器size也会变大。(镜像保存后所有修改都会变成只读层)

## 标准

为了满足容器的一系列问题,并且标准化,成立了OCI(Open Contanier Initiative)提出了两个标准：

- 容器运行时标准：主要指定容器的运行状态和运行时需要的指令，docker公司给社区捐献了一个OCI容器的实现，就是runc。
- 容器镜像标准：主要是说明容器的镜像格式，一般是以OCI runtime filesystem bundle形式存在。

## low-level container runtime

low-level container runtime从原理上来讲是用linux namespace实现命名空间的隔离、资源的虚拟化和用cgroup来实现资源的使用控制。

- Create cgroup
- Run command(s) in cgroup
- Unshare to move to its own namespaces
- Clean up cgroup after command completes 

## high-level container runtime

- high-level container runtime负责容器镜像的传输和管理，解压缩镜像，然后传递到低级运行时以运行容器。高级运行时提供了守护程序应用程序和API，远程应用程序可使用它们来逻辑运行容器并对其进行监视，但是它们位于底层并委托低级运行时或其他高级运行时进行实际工作。
- Docker是一个包含了构建，打包，共享和运行容器的容器运行时。Docker将low-level和high-level分为了单独的项目。Docker现在包括dockerd守护程序，docker-containerd守护程序以及docker-runc。 docker-containerd和docker-runc是containerd和runc的vanilla版本。
- dockerd提供如构建映像之类的功能，而dockerd使用docker-containerd提供如映像管理和运行中的容器之类的功能。
- 控制链路: dockerd -> containerd -> runc 

## 容器

一个由 Namespace+Cgroups 构成的隔离环境，这一部分我们称为“容器运行时”（Container Runtime），是容器的动态视图。由“容器运行时”加上使用Mount Namespace挂载的联合文件系统(rootfs)，整合而成的就是docker容器。



## 镜像

一组联合挂载在 /var/lib/docker/aufs/mnt 上的 rootfs，这一部分我们称为“容器镜像”（Container Image），是容器的静态视图。对于开发者而言，我们并不关心容器运行时的差异，所以真正承载着容器信息进行传递的，是容器镜像。



## 总结

一个docker 容器是使用各种Namespace修改了视图的一个特殊进程， 所以其底层的linux的内核版本，系统时间是不能被改变的。为了防止某个容器（进程）抢占所有的系统资源(cpu, 内存等), 所以需要对其加上了Cgroups的限制。相比一个真正的虚拟机随意折腾，需要理解容器能做什么，不能做什么。   

