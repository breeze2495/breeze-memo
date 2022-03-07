## P0: 预习篇

### Docker发展历程

**像 Cloud Foundry 这样的 PaaS 项目，最核心的组件就是一套应用的打包和分发机制**。 Cloud Foundry 为每种主流编程语言都定义了一种打包格式，而“cf push”的作用

```shell
$ cf push "我的应用"
```

基本上等同于用户把应用的可执行文件和启动脚本打进一个压缩包内，上传到云上 Cloud Foundry 的存储中。接着，Cloud Foundry 会通过调度器选择一个可以运行这个应用的虚拟机，然后通知这个机器上的 Agent 把应用压缩包下载下来启动。

这时候关键来了，由于需要在一个虚拟机上启动很多个来自不同用户的应用，**Cloud Foundry 会调用操作系统的 Cgroups 和 Namespace 机制为每一个应用单独创建一个称作“沙盒”的隔离环境，然后在“沙盒”中启动这些应用进程**。这样，就实现了把多个用户的应用互不干涉地在虚拟机里批量地、自动地运行起来的目的。

这，**正是 PaaS 项目最核心的能力**。 而这些 Cloud Foundry 用来运行应用的隔离环境，或者说“沙盒”，就是所谓的“容器”。

**Docker 项目确实与 Cloud Foundry 的容器在大部分功能和实现原理上都是一样的**，可偏偏就是这剩下的一小部分不一样的功能，成了 Docker 项目接下来“呼风唤雨”的不二法宝。

**这个功能，就是 Docker 镜像。**

**“cf push”确实是能一键部署了，但是为了实现这个一键部署，用户为每个应用打包的工作可谓一波三折，费尽心机。**

**而 Docker 镜像解决的，恰恰就是打包这个根本性的问题。** 所谓 Docker 镜像，其实就是一个压缩包。但是这个压缩包里的内容，比 PaaS 的应用可执行文件 + 启停脚本的组合就要丰富多了。实际上，大多数 Docker 镜像是直接由一个完整操作系统的所有文件和目录构成的，所以这个压缩包里的内容跟你本地开发和测试环境用的操作系统是完全一样的。







## P1: 容器技术相关

> 容器本身没有价值，有价值的是“容器编排”。

### D1: 容器只是一种特殊的进程



**容器其实是一种沙盒技术**

顾名思义，沙盒就是能够像一个集装箱一样，把你的应用“装”起来的技术。这样，应用与应用之间，就因为有了边界而不至于相互干扰；而被装进集装箱的应用，也可以被方便地搬来搬去，这就是 PaaS 最理想的状态。



**边界的实现手段**

容器技术的**核心功能**，就是通过**约束和修改进程的动态表现**，从而为其**创造出一个“边界”**。

**Cgroups** 技术是用来制造约束的主要手段，而 **Namespace** 技术则是用来修改进程视图的主要方法。

```java
$ docker run -it busybox /bin/sh
/ #

//而 -it 参数告诉了 Docker 项目在启动容器后，需要给我们分配一个文本输入 / 输出环境，也就是 TTY，跟容器的标准输入相关联，这样我们就可以和这个 Docker 容器进行交互了。而 /bin/sh 就是我们要在 Docker 容器里运行的程序。
//上面这条指令翻译成人类的语言就是：请帮我启动一个容器，在容器里执行 /bin/sh，并且给我分配一个命令行终端跟这个容器交互。
```



**Namespace机制**

```java
/ # ps
PID  USER   TIME COMMAND
  1 root   0:00 /bin/sh
  10 root   0:00 ps
```

可以看到，我们在 Docker 里最开始执行的 /bin/sh，就是这个容器内部的第 1 号进程（PID=1），而这个容器里一共只有两个进程在运行。这就意味着，前面执行的 /bin/sh，以及我们刚刚执行的 ps，已经被 Docker 隔离在了一个跟宿主机完全不同的世界当中。

每当我们在宿主机上运行了一个 /bin/sh 程序，操作系统都会给它分配一个进程编号，比如 PID=100。这个编号是进程的唯一标识，就像员工的工牌一样。所以 PID=100，可以粗略地理解为这个 /bin/sh 是我们公司里的第 100 号员工，而第 1 号员工就自然是比尔 · 盖茨这样统领全局的人物。

而现在，我们要通过 Docker 把这个 /bin/sh 程序运行在一个容器当中。这时候，**Docker 就会在这个第 100 号员工入职时给他施一个“障眼法”，让他永远看不到前面的其他 99 个员工。这样，他就会错误地以为自己就是公司里的第 1 号员工**。这种机制，其实就是对被隔离应用的进程空间做了手脚，使得这些进程只能看到重新计算过的进程编号，比如 PID=1。**可实际上，他们在宿主机的操作系统里，还是原来的第 100 号进程。**

除了我们刚刚用到的 PID Namespace，Linux 操作系统还提供了 Mount、UTS、IPC、Network 和 User 这些 Namespace，用来对各种不同的进程上下文进行“**障眼法**”操作。

**因此，容器，其实是一种特殊的进程而已**



**虚拟机技术和docker技术 底层实现对比**

<img src="/Users/breeze/Library/Application Support/typora-user-images/image-20220127103252119.png" alt="image-20220127103252119" style="zoom:50%;" />

Hypervisor 的软件是虚拟机最主要的部分。它通过**硬件虚拟化功能**，模拟出了运行一个操作系统需要的各种硬件，比如 CPU、内存、I/O 设备等等。然后，它在这些虚拟的硬件上安装了一个新的操作系统，即 Guest OS。

用一个名为 Docker Engine 的软件替换了 Hypervisor。这也是为什么，很多人会把 Docker 项目称为“轻量级”虚拟化技术的原因，实际上就是把虚拟机的概念套在了容器上。

**在使用 Docker 的时候，并没有一个真正的“Docker 容器”运行在宿主机里面**。Docker 项目帮助用户启动的，还是原来的应用进程，只不过在创建这些进程时，Docker 为它们加上了各种各样的 Namespace 参数。这时，这些进程就会觉得自己是各自 PID Namespace 里的第 1 号进程，只能看到各自 Mount Namespace 里挂载的目录和文件，只能访问到各自 Network Namespace 里的网络设备，就仿佛运行在一个个“容器”里面，与世隔绝。



### D2: 隔离与限制

Namespace 技术实际上修改了应用进程看待整个计算机“视图”，即它的“视线”被操作系统做了限制，只能“看到”某些指定的内容。

因此，在上方的对比图里，我们应该**把 Docker 画在跟应用同级别并且靠边的位置**。这意味着，用户运行在容器里的应用进程，跟宿主机上的其他进程一样，都由宿主机操作系统统一管理，只不过这些被隔离的进程拥有额外设置过的 Namespace 参数。而 Docker 项目在这里扮演的角色，更多的是**旁路式的辅助和管理工作**。



**docker比虚拟机更受欢迎的原因**

使用虚拟化技术作为应用沙盒，就必须要由 Hypervisor 来负责创建虚拟机，这个**虚拟机是真实存在**的，并且它里面**必须运行一个完整的 Guest OS** 才能执行用户的应用进程。这就不可避免地带来了**额外的资源消耗和占用**。

而相比之下，**容器化后的用户应用，却依然还是一个宿主机上的普通进程**，这就意味着这些因为虚拟化而带来的性能损耗都是不存在的；而另一方面，使用 Namespace 作为隔离手段的容器**并不需要单独的 Guest OS**，这就使得容器额外的资源占用几乎可以忽略不计。

**“敏捷”和“高性能”是容器相较于虚拟机最大的优势，也是它能够在 PaaS 这种更细粒度的资源管理平台上大行其道的重要原因。**



**Namespace隔离机制的弊端**

基于 Linux Namespace 的隔离机制相比于虚拟化技术也有很多不足之处，其中最主要的问题就是：**隔离得不彻底**。

1. 首先，既然容器只是运行在宿主机上的一种特殊的进程，那么**多个容器之间使用的就还是同一个宿主机的操作系统内核**。
2. 其次，在 Linux 内核中，有**很多资源和对象是不能被 Namespace 化**的，最典型的例子就是：时间。
3. 此外，由于上述问题，尤其是**共享宿主机内核的事实，容器给应用暴露出来的攻击面是相当大**的，应用“越狱”的难度自然也比虚拟机低得多。



**容器的 限制 问题**

> 虽然容器内的第 1 号进程在“障眼法”的干扰下只能看到容器里的情况，但是宿主机上，它作为第 100 号进程与其他所有进程之间依然是平等的竞争关系。这就意味着，虽然第 100 号进程表面上被隔离了起来，但是它所能够使用到的资源（比如 CPU、内存），却是可以随时被宿主机上的其他进程（或者其他容器）占用的。当然，这个 100 号进程自己也可能把所有资源吃光。这些情况，显然都不是一个“沙盒”应该表现出来的合理行为。

**Linux Cgroups 就是 Linux 内核中用来为进程设置资源限制的一个重要功能**

- Linux Cgroups 的全称是 Linux Control Group。
  - 它最主要的作用，就是限制一个进程组能够使用的资源上限，包括 CPU、内存、磁盘、网络带宽等等。
  - 此外，Cgroups 还能够对进程进行优先级设置、审计，以及将进程挂起和恢复等操作。

- Linux Cgroups 的设计还是比较易用的，简单粗暴地理解呢，它就是一个子系统目录加上一组资源限制文件的组合。而对于 Docker 等 Linux 容器项目来说，它们只需要在每个子系统下面，为每个容器创建一个控制组（即创建一个新目录），然后在启动容器进程之后，把这个进程的 PID 填写到对应控制组的 tasks 文件中就可以了。



**一个正在运行的 Docker 容器，其实就是一个启用了多个 Linux Namespace 的应用进程，而这个进程能够使用的资源量，则受 Cgroups 配置的限制。**

这也是容器技术中一个非常重要的概念，即：**容器是一个“单进程”模型**。

由于一个容器的本质就是一个进程，用户的应用进程实际上就是容器里 PID=1 的进程，也是其他后续创建的所有进程的父进程。这就意味着，在一个容器中，你**没办法同时运行两个不同的应用，除非你能事先找到一个公共的 PID=1 的程序来充当两个不同应用的父进程**，这也是为什么很多人都会用 systemd 或者 supervisord 这样的软件来代替应用本身作为容器的启动进程。

但是，在后面分享容器设计模式时，我还会推荐其他更好的解决办法。这是因为容器本身的设计，就是**希望容器和应用能够同生命周期，这个概念对后续的容器编排非常重要**。否则，一旦出现类似于“容器是正常运行的，但是里面的应用早已经挂了”的情况，编排系统处理起来就非常麻烦了。



### D3: 容器镜像

#### rootfs(根文件系统)

 Mount Namespace 跟其他 Namespace 的使用略有不同的地方：

**它对容器进程视图的改变，一定是伴随着挂载操作（mount）才能生效。**

> 作为一个普通用户，我们希望的是一个更友好的情况：每当创建一个新容器时，我希望容器进程看到的文件系统就是一个独立的隔离环境，而不是继承自宿主机的文件系统。
>
> 怎么才能做到这一点呢？不难想到，我们可以在容器进程启动之前重新挂载它的整个根目录“/”。而由于 Mount Namespace 的存在，这个挂载对宿主机不可见，所以容器进程就可以在里面随便折腾了。
>
> 在 Linux 操作系统里，有一个名为 chroot 的命令可以帮助你在 shell 中方便地完成这个工作。顾名思义，它的作用就是帮你“change root file system”，即改变进程的根目录到你指定的位置。
>
> 实际上，Mount Namespace 正是基于对 chroot 的不断改良才被发明出来的，它也是 Linux 操作系统里的第一个 Namespace。

**而这个挂载在容器根目录上、用来为容器进程提供隔离后执行环境的文件系统，就是所谓的“容器镜像”。它还有一个更为专业的名字，叫作：rootfs（根文件系统）。**

所以，一个最常见的 rootfs，或者说容器镜像，会包括如下所示的一些目录和文件，比如 /bin，/etc，/proc 等等：



**现在，你应该可以理解，对 Docker 项目来说，它最核心的原理实际上就是为待创建的用户进程：**

1. 启用 Linux Namespace 配置；
2. 设置指定的 Cgroups 参数；
3. 切换进程的根目录（Change Root）。

**这样，一个完整的容器就诞生了。**

不过，Docker 项目在最后一步的切换上会优先使用 pivot_root 系统调用，如果系统不支持，才会使用 chroot。这两个系统调用虽然功能类似，但是也有细微的区别。

另外，**需要明确的是，rootfs 只是一个操作系统所包含的文件、配置和目录，并不包括操作系统内核。在 Linux 操作系统中，这两部分是分开存放的，操作系统只有在开机启动时才会加载指定版本的内核镜像。**

**实际上，同一台机器上的所有容器，共享宿主机操作系统的内核。**

> 这就意味着，如果你的应用程序需要配置内核参数、加载额外的内核模块，以及跟内核进行直接的交互，你就需要注意了：这些操作和依赖的对象，都是宿主机操作系统的内核，它对于该机器上的所有容器来说是一个“全局变量”，牵一发而动全身。这也是容器相比于虚拟机的主要缺陷之一：毕竟后者不仅有模拟出来的硬件机器充当沙盒，而且每个沙盒里还运行着一个完整的 Guest OS 给应用随便折腾。



**不过，正是由于 rootfs 的存在，容器才有了一个被反复宣传至今的重要特性：一致性。**

> **一致性：** 
>
> 由于云端与本地服务器环境不同，应用的打包过程，一直是使用PaaS最痛苦的步骤
>
> 有了容器（更准确的说是有个rootfs）之后，一致性的问题被解决了

**由于 rootfs 里打包的不只是应用，而是整个操作系统的文件和目录，也就意味着，应用以及它运行所需要的所有依赖，都被封装在了一起。**

**对一个应用来说，操作系统本身才是它运行所需要的最完整的“依赖库”。**

有了容器镜像“打包操作系统”的能力，这个最基础的依赖环境也终于变成了应用沙盒的一部分。这就赋予了容器所谓的一致性：无论在本地、云端，还是在一台任何地方的机器上，用户只需要解压打包好的容器镜像，那么这个应用运行所需要的完整的执行环境就被重现出来了。

**这种深入到操作系统级别的运行环境一致性，打通了应用在本地开发和远端执行环境之间难以逾越的鸿沟。**



**存在的问题** 

> 是否每开发或者升级一个现有应用，都要重复制作一次rootfs？
>
> - 场景：用 Ubuntu 操作系统的 ISO 做了一个 rootfs，然后又在里面安装了 Java 环境，用来部署我的 Java 应用。那么，我的另一个同事在发布他的 Java 应用时，显然希望能够直接使用我安装过 Java 环境的 rootfs，而不是重复这个流程
> - 一旦共用同一个rootfs的应用，需要修改这个rootfs，新旧rootfs就没有任何联系了，造成了极度的碎片化。

**以增量的方式做修改**

> Docker 在镜像的设计中，引入了层（layer）的概念。
>
> 也就是说，**用户制作镜像的每一步操作，都会生成一个层，也就是一个增量 rootfs。**

**联合文件系统(Union File System)** 

> 最主要的功能是将多个不同位置的目录联合挂载（union mount）到同一个目录下

```java
//比如，我现在有两个目录 A 和 B，它们分别有两个文件：

$ tree
.
├── A
│  ├── a
│  └── x
└── B
  ├── b
  └── x
  
//然后，我使用联合挂载的方式，将这两个目录挂载到一个公共的目录 C 上：

$ mkdir C
$ mount -t aufs -o dirs=./A:./B none ./C

//这时，我再查看目录 C 的内容，就能看到目录 A 和 B 下的文件被合并到了一起：

$ tree ./C
./C
├── a
├── b
└── x 
```

**docker的 AuFS : Advance UnionFS:**  它是对UnionFS的重写与改进 

**一个容器的rootfs由三部分组成：** 使用Ubuntu

<img src="/Users/breeze/Library/Application Support/typora-user-images/image-20220127142751345.png" alt="image-20220127142751345" style="zoom:50%;" />

**第一部分：只读层**

它是这个容器的 rootfs 最下面的五层，对应的正是 ubuntu:latest 镜像的五层。可以看到，它们的挂载方式都是只读的（ro+wh，即 readonly+whiteout）

**第二部分：可读写层**

它是这个容器的 rootfs 最上面的一层（6e3be5d2ecccae7cc），它的挂载方式为：rw，即 read write。在没有写入文件之前，这个目录是空的。而一旦在容器里做了写操作，你修改产生的内容就会以增量的方式出现在这个层中。

如果是删除操作，AuFS 会在可读写层创建一个 whiteout 文件，把只读层里的文件“遮挡”起来。

比如，你要删除只读层里一个名叫 foo 的文件，那么这个删除操作实际上是在可读写层创建了一个名叫.wh.foo 的文件。这样，当这两个层被联合挂载之后，foo 文件就会被.wh.foo 文件“遮挡”起来，“消失”了。这个功能，就是“ro+wh”的挂载方式，即只读 +whiteout 的含义。我喜欢把 whiteout 形象地翻译为：“白障”。

最上面这个**可读写层的作用，就是专门用来存放你修改 rootfs 后产生的增量，无论是增、删、改，都发生在这里**。而当我们使用完了这个被修改过的容器之后，还可以使用 docker commit 和 push 指令，保存这个被修改过的可读写层，并上传到 Docker Hub 上，供其他人使用；而与此同时，原先的只读层里的内容则不会有任何变化。这，就是增量 rootfs 的好处。

**第三部分：init层**

它是一个以“-init”结尾的层，夹在只读层和读写层之间。Init 层是 Docker 项目单独生成的一个内部层，专门用来存放 /etc/hosts、/etc/resolv.conf 等信息。

需要这样一层的原因是，这些文件本来属于只读的 Ubuntu 镜像的一部分，但是用户往往需要在启动容器时**写入一些指定的值比如 hostname**，所以就需要在可读写层对它们进行修改。可是，这些修改往往只对当前的容器有效，我们**并不希望执行 docker commit 时，把这些信息连同可读写层一起提交掉**。所以，Docker 做法是，在修改了这些文件之后，以一个单独的层挂载了出来。而用户执行 docker commit 只会提交可读写层，所以是不包含这些内容的。



### D4: k8s本质

<img src="/Users/breeze/Library/Application Support/typora-user-images/image-20220127172551343.png" alt="image-20220127172551343" style="zoom:30%;" />



#### Master节点（控制节点）

> **如何编排，管理，调度用户提交的作业？**
>
> 运行在大规模集群中的各种任务之间，实际上存在着各种各样的关系。这些关系的处理，才是作业编排和管理系统最困难的地方。

控制节点，即 Master 节点，由三个紧密协作的独立组件组合而成，它们分别是负责 API 服务的 kube-apiserver、负责调度的 kube-scheduler，以及负责容器编排的 kube-controller-manager。整个集群的持久化数据，则由 kube-apiserver 处理后保存在 Etcd 中。









#### Node节点（计算节点）

最核心的部分，则是一个叫作 kubelet 的组件。



##### Kubelet组件

- **CRI远程调用接口与容器进行交互**

kubelet 主要负责同容器运行时（比如 Docker 项目）打交道。而这个交互所依赖的，是一个称作 CRI（Container Runtime Interface）的远程调用接口，这个接口**定义了容器运行时的各项核心操作**，比如：启动一个容器需要的所有参数

- **Device Plugin插件管理GPU等主机物理设备**

kubelet 还通过 gRPC 协议同一个叫作 Device Plugin 的插件进行交互。这个插件，是 Kubernetes 项目**用来管理 GPU 等宿主机物理设备**的主要组件，也是基于 Kubernetes 项目进行机器学习训练、高性能作业支持等工作必须关注的功能

- **网络插件和存储插件为容器配置网络和持久化存储**

这两个插件与 kubelet 进行交互的接口，分别是 CNI（Container Networking Interface）和 CSI（Container Storage Interface）。







## P2: 容器编排与k8s作业管理

### D1: 为什么我们需要pod？

**“Pod存在的必要性”**

> Pod，是 Kubernetes 项目中**最小的 API 对象**。
>
> 如果换一个更专业的说法，我们可以这样描述：Pod，是 Kubernetes 项目的**原子调度单位**。
>

### 1.用于管理“进程组”



**容器的本质是一个进程**

**Pod扮演的是传统部署环境里面“虚拟机”的角色，这样的设计，是为了使用户从传统环境（虚拟机环境）向 Kubernetes（容器环境）的迁移，更加平滑**

**Kubernetes 项目所做的，其实就是将“进程组”的概念映射到了容器技术中，并使其成为了这个云计算“操作系统”里的“一等公民。“进程组”对应的也是 Linux 操作系统语境下的“线程组”**

> Kubernetes 项目之所以要这么做的原因，我在前面介绍 Kubernetes 和 Borg 的关系时曾经提到过：在 Borg 项目的开发和实践过程中，Google 公司的工程师们发现，他们部署的应用，往往都存在着类似于“进程和进程组”的关系。**更具体地说，就是这些应用之间有着密切的协作关系，使得它们必须部署在同一台机器上**。而如果事先没有“组”的概念，像这样的运维关系就会非常难以处理
>

以前面的 rsyslogd 为例子。已知 rsyslogd 由三个进程组成：一个 imklog 模块，一个 imuxsock 模块，一个 rsyslogd 自己的 main 函数主进程。这三个进程一定要运行在同一台机器上，否则，它们之间基于 Socket 的通信和文件交换，都会出现问题。现在，我要把 rsyslogd 这个应用给容器化，由于受限于容器的“单进程模型”，这三个模块必须被分别制作成三个不同的容器。而在这三个容器运行的时候，它们设置的内存配额都是 1 GB。

> 再次强调一下：容器的“**单进程模型**”，并不是指容器里只能运行“一个”进程，而是指容器没有管理多个进程的能力。

**成组调度存在的问题**

> 假设我们的 Kubernetes 集群上有两个节点：node-1 上有 3 GB 可用内存，node-2 有 2.5 GB 可用内存。
>
> 这时，假设我要用 Docker Swarm 来运行这个 rsyslogd 程序。为了能够让这三个容器都运行在同一台机器上，我就必须在另外两个容器上设置一个 affinity=main（与 main 容器有亲密性）的约束，即：它们俩必须和 main 容器运行在同一台机器上。
>
> 然后，我顺序执行：“docker run main”“docker run imklog”和“docker run imuxsock”，创建这三个容器。
>
> 这样，这三个容器都会进入 Swarm 的待调度队列。然后，main 容器和 imklog 容器都先后出队并被调度到了 node-2 上（这个情况是完全有可能的）。
>
> 可是，当 imuxsock 容器出队开始被调度时，Swarm 就有点懵了：node-2 上的可用资源只有 0.5 GB 了，并不足以运行 imuxsock 容器；可是，根据 affinity=main 的约束，imuxsock 容器又只能运行在 node-2 上。
>
> 在工业界和学术界，关于这个问题的讨论可谓旷日持久，也产生了很多可供选择的解决方案。比如，Mesos 中就有一个资源囤积（resource hoarding）的机制，会在所有设置了 Affinity 约束的任务都达到时，才开始对它们统一进行调度。而在 Google Omega 论文中，则提出了使用乐观调度处理冲突的方法，即：先不管这些冲突，而是通过精心设计的回滚机制在出现了冲突之后解决问题。

**但是，到了 Kubernetes 项目里，这样的问题就迎刃而解了：**

**Pod 是 Kubernetes 里的原子调度单位。这就意味着，Kubernetes 项目的调度器，是统一按照 Pod 而非容器的资源需求进行计算的。**

> 所以，像 imklog、imuxsock 和 main 函数主进程这样的三个容器，正是一个典型的由三个容器组成的 Pod。Kubernetes 项目在调度时，自然就会去选择可用内存等于 3 GB 的 node-1 节点进行绑定，而根本不会考虑 node-2。
>
> 像这样容器间的紧密协作，我们可以称为“超亲密关系”。这些具有“超亲密关系”容器的典型特征包括但不限于：互相之间会发生直接的文件交换、使用 localhost 或者 Socket 文件进行本地通信、会发生非常频繁的远程调用、需要共享某些 Linux Namespace（比如，一个容器要加入另一个容器的 Network Namespace）等等。



#### 2.“容器设计模式”

> **Pod 在 Kubernetes 项目里还有更重要的意义，那就是：容器设计模式**

**Pod的实现原理**

首先，关于 Pod 最重要的一个事实是：它只是一个**逻辑概念**。

也就是说，Kubernetes 真正处理的，还是宿主机操作系统上 Linux 容器的 Namespace 和 Cgroups，而并不存在一个所谓的 Pod 的边界或者隔离环境。

那么，Pod 又是怎么被“创建”出来的呢？

答案是：Pod，其实是一组共享了某些资源的容器。具体的说：Pod 里的所有容器，共享的是同一个 Network Namespace，并且可以声明共享同一个 Volume。

> 那这么来看的话，一个有 A、B 两个容器的 Pod，不就是等同于一个容器（容器 A）共享另外一个容器（容器 B）的网络和 Volume 的玩儿法么？
>
> 这好像通过 docker run --net --volumes-from 这样的命令就能实现，
>
> 比如：$ docker run --net=B --volumes-from=B --name=A image-A ...
>
> 但是，如果真这样做的话，容器 B 就必须比容器 A 先启动，这样一个 Pod 里的多个容器就**不是对等关系，而是拓扑关系**了。

**所以，在 Kubernetes 项目里，Pod 的实现需要使用一个中间容器，这个容器叫作 Infra 容器。**

> 在这个 Pod 中，Infra 容器永远都是第一个被创建的容器，而其他用户定义的容器，则通过 Join Network Namespace 的方式，与 Infra 容器关联在一起。
>
> Infra 容器“Hold 住”Network Namespace 后，用户容器就可以加入到 Infra 容器的 Network Namespace 当中了。所以，如果你查看这些容器在宿主机上的 Namespace 文件（这个 Namespace 文件的路径，我已经在前面的内容中介绍过），它们指向的值一定是完全一样的。
>
> 这也就意味着，对于 Pod 里的容器 A 和容器 B 来说：
>
> - 它们可以直接使用 localhost 进行通信；
> - 它们看到的网络设备跟 Infra 容器看到的完全一样；
> - 一个 Pod 只有一个 IP 地址，也就是这个 Pod 的 Network Namespace 对应的 IP 地址；
> - 当然，其他的所有网络资源，都是一个 Pod 一份，并且被该 Pod 中的所有容器共享；
> - Pod 的生命周期只跟 Infra 容器一致，而与容器 A 和 B 无关。
>
> 而对于同一个 Pod 里面的所有用户容器来说，它们的进出流量，也可以认为都是通过 Infra 容器完成的。这一点很重要，因为将来如果你要为 Kubernetes 开发一个网络插件时，应该重点考虑的是如何配置这个 Pod 的 Network Namespace，而不是每一个用户容器如何使用你的网络配置，这是没有意义的。

**有了这个设计之后，共享 Volume 就简单多了：Kubernetes 项目只要把所有 Volume 的定义都设计在 Pod 层级即可。**



**典型例子： WAR包与Tomcat服务器 -------  sidecar模式**

> pod中定义两个容器，一个是含有WAR包镜像的容器，另一个是含有标准tomcat镜像的容器；
>
> WAR包容器的类型是init container，会先启动。并且，Init Container 容器会按顺序逐一启动，而直到它们都启动并且退出了，用户容器才会启动。
>
> 分别挂载同一个名称的目录到自己的目录下
>
> - /app 目录，就挂载了一个名叫 app-volume 的 Volume。
> - Tomcat 容器，同样声明了挂载 app-volume 到自己的 webapps 目录下。
>
> 所以，等 Tomcat 容器启动时，它的 webapps 目录下就一定会存在 sample.war 文件：这个文件正是 WAR 包容器启动时拷贝到这个 Volume 里面的，而这个 Volume 是被这两个容器共享的。

**像这样，我们就用一种“组合”方式，解决了 WAR 包与 Tomcat 容器之间耦合关系的问题。**





### D2: 深入解析pod对象



#### 1.Pod一些重要字段

- **NodeSelector：**供用户将 Pod 与 Node 进行绑定

  ```yaml
  apiVersion: v1
  kind: Pod
  ...
  spec:
   nodeSelector:
     disktype: ssd
     
  #这样的一个配置，意味着这个 Pod 永远只能运行在携带了“disktype: ssd”标签（Label）的节点上；否则，它将调度失败。
  ```

  

- **NodeName：**一旦 Pod 的这个字段被赋值，Kubernetes 项目就会被认为这个 Pod 已经经过了调度，调度的结果就是赋值的节点名字。所以，这个字段一般由调度器负责设置，但用户也可以设置它来“骗过”调度器，当然这个做法一般是在测试或者调试的时候才会用到。

- **HostAliases：**定义了 Pod 的 hosts 文件（比如 /etc/hosts）里的内容

  ```yaml
  apiVersion: v1
  kind: Pod
  ...
  spec:
    hostAliases:
    - ip: "10.1.2.3"
      hostnames:
      - "foo.remote"
      - "bar.remote"
  ...
  
  #在这个 Pod 的 YAML 文件中，我设置了一组 IP 和 hostname 的数据。这样，这个 Pod 启动后，/etc/hosts 文件的内容将如下所示：
  
  cat /etc/hosts
  # Kubernetes-managed hosts file.
  127.0.0.1 localhost
  ...
  10.244.135.10 hostaliases-pod
  10.1.2.3 foo.remote
  10.1.2.3 bar.remote
  
  #其中，最下面两行记录，就是我通过 HostAliases 字段为 Pod 设置的。需要指出的是，在 Kubernetes 项目中，如果要设置 hosts 文件里的内容，一定要通过这种方法。否则，如果直接修改了 hosts 文件的话，在 Pod 被删除重建之后，kubelet 会自动覆盖掉被修改的内容。
  ```

  - **ImagePullPolicy：**定义了镜像拉取策略

    - ImagePullPolicy 的值默认是 Always，即每次创建 Pod 都重新拉取一次镜像。
    - 另外，当容器的镜像是类似于 nginx 或者 nginx:latest 这样的名字时，ImagePullPolicy 也会被认为 Always。
    - 而如果它的值被定义为 Never 或者 IfNotPresent，则意味着 Pod 永远不会主动拉取这个镜像，或者只在宿主机上不存在这个镜像时才拉取。

  - **Lifecycle：**它定义的是 Container Lifecycle Hooks。顾名思义，Container Lifecycle Hooks 的作用，是在容器状态发生变化时触发一系列“钩子”

    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: lifecycle-demo
    spec:
      containers:
      - name: lifecycle-demo-container
        image: nginx
        lifecycle:
          postStart:
            exec:
              command: ["/bin/sh", "-c", "echo Hello from the postStart handler > /usr/share/message"]
          preStop:
            exec:
              command: ["/usr/sbin/nginx","-s","quit"]
              
    #先说 postStart 吧。它指的是，在容器启动后，立刻执行一个指定的操作。
    #需要明确的是，postStart 定义的操作，虽然是在 Docker 容器 ENTRYPOINT 执行之后，但它并不严格保证顺序。也就是说，在 postStart 启动时，ENTRYPOINT 有可能还没有结束。当然，如果 postStart 执行超时或者错误，Kubernetes 会在该 Pod 的 Events 中报出该容器启动失败的错误信息，导致 Pod 也处于失败的状态。
    
    #而类似地，preStop 发生的时机，则是容器被杀死之前（比如，收到了 SIGKILL 信号）。而需要明确的是，preStop 操作的执行，是同步的。所以，它会阻塞当前的容器杀死流程，直到这个 Hook 定义操作完成之后，才允许容器被杀死，这跟 postStart 不一样。所以，在这个例子中，我们在容器成功启动之后，在 /usr/share/message 里写入了一句“欢迎信息”（即 postStart 定义的操作）。而在这个容器被删除之前，我们则先调用了 nginx 的退出指令（即 preStop 定义的操作），从而实现了容器的“优雅退出”。
    ```



#### 2.Pod的生命周期

主要体现在 Pod API 对象的 Status 部分，这是它除了 Metadata 和 Spec 之外的第三个重要字段。其中，pod.status.phase，就是 Pod 的当前状态，它有如下几种可能的情况：

- **Pending**。这个状态意味着，Pod 的 YAML 文件已经提交给了 Kubernetes，API 对象已经被创建并保存在 Etcd 当中。但是，这个 Pod 里有些容器因为某种原因而不能被顺利创建。比如，调度不成功。
- **Running**。这个状态下，Pod 已经调度成功，跟一个具体的节点绑定。它包含的容器都已经创建成功，并且至少有一个正在运行中。
- **Succeeded**。这个状态意味着，Pod 里的所有容器都正常运行完毕，并且已经退出了。这种情况在运行一次性任务时最为常见。
- **Failed**。这个状态下，Pod 里至少有一个容器以不正常的状态（非 0 的返回码）退出。这个状态的出现，意味着你得想办法 Debug 这个容器的应用，比如查看 Pod 的 Events 和日志。
- **Unknown**。这是一个异常状态，意味着 Pod 的状态不能持续地被 kubelet 汇报给 kube-apiserver，这很有可能是主从节点（Master 和 Kubelet）间的通信出现了问题。

> 更进一步地，Pod 对象的 Status 字段，还可以再细分出一组 Conditions。这些细分状态的值包括：PodScheduled、Ready、Initialized，以及 Unschedulable。它们主要用于描述造成当前 Status 的具体原因是什么。
>
> 比如，Pod 当前的 Status 是 Pending，对应的 Condition 是 Unschedulable，这就意味着它的调度出现了问题。而其中，Ready 这个细分状态非常值得我们关注：它意味着 Pod 不仅已经正常启动（Running 状态），而且已经可以对外提供服务了。这两者之间（Running 和 Ready）是有区别的



#### 3.Projected Volume

Kubernetes 支持的 Projected Volume 一共有四种：

- **Secret**

- **ConfigMap**

- **Downward API**

- **ServiceAccountToken**



##### Secret

> 帮你把 Pod 想要访问的加密数据，存放到 Etcd 中。然后，你就可以通过在 Pod 的容器里挂载 Volume 的方式，访问到这些 Secret 里保存的信息了。
>

```yaml
#最典型的场景：存放数据库认证信息

apiVersion: v1
kind: Pod
metadata:
  name: test-projected-volume 
spec:
  containers:
  - name: test-secret-volume
    image: busybox
    args:
    - sleep
    - "86400"
    volumeMounts:
    - name: mysql-cred
      mountPath: "/projected-volume"
      readOnly: true
  volumes:
  - name: mysql-cred
    projected:
      sources:
      - secret:
          name: user
      - secret:
          name: pass
          
#在这个 Pod 中，我定义了一个简单的容器。它声明挂载的 Volume，并不是常见的 emptyDir 或者 hostPath 类型，而是 projected 类型。而这个 Volume 的数据来源（sources），则是名为 user 和 pass 的 Secret 对象，分别对应的是数据库的用户名和密码。
 
#我们可以看到，保存在 Etcd 里的用户名和密码信息，已经以文件的形式出现在了容器的 Volume 目录里。而这个文件的名字，就是 kubectl create secret 指定的 Key，或者说是 Secret 对象的 data 字段指定的 Key。
#更重要的是，像这样通过挂载方式进入到容器里的 Secret，一旦其对应的 Etcd 里的数据被更新，这些 Volume 里的文件内容，同样也会被更新。其实，这是 kubelet 组件在定时维护这些 Volume。
```



##### ConfigMap

> 它与 Secret 的区别在于，ConfigMap 保存的是不需要加密的、应用所需的配置信息。而 ConfigMap 的用法几乎与 Secret 完全相同

```yaml
#比如，一个 Java 应用所需的配置文件（.properties 文件），就可以通过下面这样的方式保存在 ConfigMap 里：

# .properties文件的内容
$ cat example/ui.properties
color.good=purple
color.bad=yellow
allow.textmode=true
how.nice.to.look=fairlyNice

# 从.properties文件创建ConfigMap
$ kubectl create configmap ui-config --from-file=example/ui.properties

# 查看这个ConfigMap里保存的信息(data)
$ kubectl get configmaps ui-config -o yaml
apiVersion: v1
data:
  ui.properties: |
    color.good=purple
    color.bad=yellow
    allow.textmode=true
    how.nice.to.look=fairlyNice
kind: ConfigMap
metadata:
  name: ui-config
  ...
```



##### Downward API

> 让 Pod 里的容器能够直接获取到这个 Pod API 对象本身的信息

```yaml

apiVersion: v1
kind: Pod
metadata:
  name: test-downwardapi-volume
  labels:
    zone: us-est-coast
    cluster: test-cluster1
    rack: rack-22
spec:
  containers:
    - name: client-container
      image: k8s.gcr.io/busybox
      command: ["sh", "-c"]
      args:
      - while true; do
          if [[ -e /etc/podinfo/labels ]]; then
            echo -en '\n\n'; cat /etc/podinfo/labels; fi;
          sleep 5;
        done;
      volumeMounts:
        - name: podinfo
          mountPath: /etc/podinfo
          readOnly: false
  volumes:
    - name: podinfo
      projected:
        sources:
        - downwardAPI:
            items:
              - path: "labels"
                fieldRef:
                  fieldPath: metadata.labels
 #这个 Downward API Volume，则声明了要暴露 Pod 的 metadata.labels 信息给容器。通过这样的声明方式，当前 Pod 的 Labels 字段的值，就会被 Kubernetes 自动挂载成为容器里的 /etc/podinfo/labels 文件。
 
 #需要注意的是，Downward API 能够获取到的信息，一定是 Pod 里的容器进程启动之前就能够确定下来的信息。而如果你想要获取 Pod 容器运行后才会出现的信息，比如，容器进程的 PID，那就肯定不能使用 Downward API 了，而应该考虑在 Pod 里定义一个 sidecar 容器
```



##### ServiceAccountToken



> 相信你一定有过这样的想法：我现在有了一个 Pod，我能不能在这个 Pod 里安装一个 Kubernetes 的 Client，这样就可以从容器里直接访问并且操作这个 Kubernetes 的 API 了呢？
>
> 这当然是可以的。
>
> 不过，你首先要解决 **API Server 的授权问题**。
>
> **Service Account** 对象的作用，就是 Kubernetes 系统内置的一种“服务账户”，它是 Kubernetes 进行权限分配的对象。比如，Service Account A，可以只被允许对 Kubernetes API 进行 GET 操作，而 Service Account B，则可以有 Kubernetes API 的所有操作权限。
>
> 像这样的 Service Account 的授权信息和文件，实际上保存在它所绑定的一个特殊的 Secret 对象里的。这个特殊的 Secret 对象，就叫作 **ServiceAccountToken**。任何运行在 Kubernetes 集群上的应用，都必须使用这个 ServiceAccountToken 里保存的授权信息，也就是 Token，才可以合法地访问 API Server。
>
> 另外，为了方便使用，Kubernetes 已经为你提供了一个默认“服务账户”（default Service Account）。并且，任何一个运行在 Kubernetes 里的 Pod，都可以直接使用这个默认的 Service Account，而无需显示地声明挂载它。
>
> **这种把 Kubernetes 客户端以容器的方式运行在集群里，然后使用 default Service Account 自动授权的方式，被称作“InClusterConfig”，也是最推荐的进行 Kubernetes API 编程的授权方式。**



#### 4.容器健康检查和恢复机制

> 在 Kubernetes 中，你可以为 Pod 里的容器定义一个健康检查“探针”（Probe）。这样，kubelet 就会**根据这个 Probe 的返回值决定这个容器的状态**，而不是直接以容器镜像是否运行（来自 Docker 返回的信息）作为依据。这种机制，是生产环境中保证应用健康存活的重要手段。



**restartPolicy**

- Always：在任何情况下，只要容器不在运行状态，就自动重启容器；
- OnFailure: 只在容器 异常时才自动重启容器；
- Never: 从来不重启容器。

> 在实际使用时，我们需要根据应用运行的特性，合理设置这三种恢复策略。
>
> 比如，一个 Pod，它只计算 1+1=2，计算完成输出结果后退出，变成 Succeeded 状态。这时，你如果再用 restartPolicy=Always 强制重启这个 Pod 的容器，就没有任何意义了。
>
> 而如果你要关心这个容器退出后的上下文环境，比如容器退出后的日志、文件和目录，就需要将 restartPolicy 设置为 Never。因为一旦容器被自动重新创建，这些内容就有可能丢失掉了（被垃圾回收了）。
>
> 只要记住如下两个基本的设计原理即可：
>
> - 只要 Pod 的 restartPolicy 指定的策略允许重启异常的容器（比如：Always），那么这个 Pod 就会保持 Running 状态，并进行容器重启。否则，Pod 就会进入 Failed 状态 。
> - 对于包含多个容器的 Pod，只有它里面所有的容器都进入异常状态后，Pod 才会进入 Failed 状态。在此之前，Pod 都是 Running 状态。此时，Pod 的 READY 字段会显示正常容器的个数，比如：





### D3: 编排其实很简单：谈谈“控制器”模型

> Pod 对象，其实就是容器的升级版。它对容器进行了组合，添加了更多的属性和字段。这就好比给集装箱四面安装了吊环，使得 Kubernetes 这架“吊车”，可以更轻松地操作它。
>
> 而 Kubernetes 操作这些“集装箱”的逻辑，都由控制器（Controller）完成



**Kubernetes 项目的 pkg/controller 目录：**

```yaml
$ cd kubernetes/pkg/controller/
$ ls -d */              
deployment/             job/                    podautoscaler/          
cloud/                  disruption/             namespace/              
replicaset/             serviceaccount/         volume/
cronjob/                garbagecollector/       nodelifecycle/          replication/            statefulset/            daemon/
...
```

**Deployment**正是控制器中的一种，这些控制器之所以被统一放在 pkg/controller 目录下，就是因为它们都遵循 Kubernetes 项目中的一个通用编排模式，即：**控制循环（control loop）**。



**控制器模式**





#### Deployment

> Deployment 看似简单，但实际上，它实现了 Kubernetes 项目中一个非常重要的功能：Pod 的“**水平扩展 / 收缩**”（horizontal scaling out/in）。

举个例子，如果你更新了 Deployment 的 Pod 模板（比如，修改了容器的镜像），那么 Deployment 就需要遵循一种叫作“**滚动更新**”（rolling update）的方式，来升级现有的容器。

而这个能力的实现，依赖的是 Kubernetes 项目中的一个非常重要的概念（API 对象）：**ReplicaSet**。

ReplicaSet 的结构非常简单，我们可以通过这个 YAML 文件查看一下：

```yaml

apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: nginx-set
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.7.9

```

从这个 YAML 文件中，我们可以看到，**一个 ReplicaSet 对象，其实就是由副本数目的定义和一个 Pod 模板组成的**。

不难发现，它的定义其实是 Deployment 的一个子集。

更重要的是，**Deployment 控制器实际操纵的，正是这样的 ReplicaSet 对象，而不是 Pod 对象**。

对于一个 Deployment 所管理的 Pod，它的 ownerReference 是：ReplicaSet

![image-20220131144107827](/Users/breeze/Library/Application Support/typora-user-images/image-20220131144107827.png)

通过这张图，我们就很清楚地看到，一个定义了 replicas=3 的 Deployment，与它的 ReplicaSet，以及 Pod 的关系，实际上是一种“层层控制”的关系。

其中，ReplicaSet 负责通过“控制器模式”，保证系统中 Pod 的个数永远等于指定的个数（比如，3 个）。**这也正是 Deployment 只允许容器的 restartPolicy=Always 的主要原因：只有在容器能保证自己始终是 Running 状态的前提下，ReplicaSet 调整 Pod 的个数才有意义。**

而在此基础上，Deployment 同样通过“控制器模式”，来操作 ReplicaSet 的个数和属性，进而实现“水平扩展 / 收缩”和“滚动更新”这两个编排动作。其中，“水平扩展 / 收缩”非常容易实现，Deployment Controller 只需要修改它所控制的 ReplicaSet 的 Pod 副本个数就可以了。







### D4: 深入理解StateFulSet

>  Deployment 实际上**并不足以覆盖所有的应用编排问题**。
>
> 造成这个问题的根本原因，在于 Deployment 对应用做了一个简单化假设。**它认为，一个应用的所有 Pod，是完全一样的**。
>
> 所以，它们互相之间没有顺序，也无所谓运行在哪台宿主机上。需要的时候，Deployment 就可以通过 Pod 模板创建新的 Pod；不需要的时候，Deployment 就可以“杀掉”任意一个 Pod。
>
> 但是，**在实际的场景中，并不是所有的应用都可以满足这样的要求。**
>
> 尤其是分布式应用，它的**多个实例之间，往往有依赖关系，比如：主从关系、主备关系**。还有就是数据存储类应用，它的多个实例，往往都会在本地磁盘上保存一份数据。而这些实例一旦被杀掉，即便重建出来，实例与数据之间的对应关系也已经丢失，从而导致应用失败。
>
> 所以，这种实例之间有不对等关系，以及实例对外部数据有依赖关系的应用，就被称为“**有状态应用**”（Stateful Application）。
>
> 容器技术诞生后，大家很快发现，它用来封装“无状态应用”（Stateless Application），尤其是 Web 服务，非常好用。但是，一旦你想要用容器运行“有状态应用”，其困难程度就会直线上升。
>
> 而且，这个问题解决起来，单纯依靠容器技术本身已经无能为力，这也就导致了很长一段时间内，“有状态应用”几乎成了容器技术圈子的“忌讳”，大家一听到这个词，就纷纷摇头。
>
> 不过，Kubernetes 项目还是成为了“第一个吃螃蟹的人”。得益于“控制器模式”的设计思想，Kubernetes 项目很早就在 Deployment 的基础上，扩展出了对“有状态应用”的初步支持。这个编排功能，就是：**StatefulSet**。

StatefulSet 的设计其实非常容易理解。它**把真实世界里的应用状态，抽象为了两种情况**：

- **拓扑状态**。这种情况意味着，应用的多个实例之间不是完全对等的关系。这些应用实例，必须按照某些顺序启动，比如应用的主节点 A 要先于从节点 B 启动。而如果你把 A 和 B 两个 Pod 删除掉，它们再次被创建出来时也必须严格按照这个顺序才行。并且，新创建出来的 Pod，必须和原来 Pod 的网络标识一样，这样原先的访问者才能使用同样的方法，访问到这个新 Pod。
- **存储状态**。这种情况意味着，应用的多个实例分别绑定了不同的存储数据。对于这些应用实例来说，Pod A 第一次读取到的数据，和隔了十分钟之后再次读取到的数据，应该是同一份，哪怕在此期间 Pod A 被重新创建过。这种情况最典型的例子，就是一个数据库应用的多个存储实例。

所以，**StatefulSet 的核心功能，就是通过某种方式记录这些状态，然后在 Pod 被重新创建时，能够为新 Pod 恢复这些状态**。





#### 1.拓扑结构







#### 2.存储状态





### D5: 容器化守护进程DaemonSet

DaemonSet 的主要作用，是让你在 Kubernetes 集群里，运行一个 Daemon Pod。

 所以，这个 Pod 有如下三个特征：

- 这个 Pod 运行在 Kubernetes 集群里的每一个节点（Node）上；
- 每个节点上只有一个这样的 Pod 实例；
- 当有新的节点加入 Kubernetes 集群后，该 Pod 会自动地在新节点上被创建出来；而当旧节点被删除后，它上面的 Pod 也相应地会被回收掉。

这个机制听起来很简单，但 Daemon Pod 的意义确实是非常重要的。

几个例子：

- 各种网络插件的 Agent 组件，都必须运行在每一个节点上，用来处理这个节点上的容器网络；
- 各种存储插件的 Agent 组件，也必须运行在每一个节点上，用来在这个节点上挂载远程存储目录，操作容器的 Volume 目录；
- 各种监控组件和日志组件，也必须运行在每一个节点上，负责这个节点上的监控信息和日志搜集。

更重要的是，跟其他编排对象不一样，DaemonSet 开始运行的时机，很多时候比整个 Kubernetes 集群出现的时机都要早。



**DaemonSet 如何保证每个 Node 上有且只有一个被管理的 Pod ？**

> 显然，这是一个典型的“控制器模型”能够处理的问题。DaemonSet Controller，首先从 Etcd 里获取所有的 Node 列表，然后遍历所有的 Node。这时，它就可以很容易地去检查，当前这个 Node 上是不是有一个携带了 name=fluentd-elasticsearch 标签的 Pod 在运行。
>
> 而检查的结果，可能有这么三种情况：
>
> - 没有这种 Pod，那么就意味着要在这个 Node 上创建这样一个 Pod；
> - 有这种 Pod，但是数量大于 1，那就说明要把多余的 Pod 从这个 Node 上删除掉；
> - 正好只有一个这种 Pod，那说明这个节点是正常的。



**如何在指定的 Node 上创建新 Pod 呢？**

如果你已经熟悉了 Pod API 对象的话，那一定可以立刻说出答案：用 nodeSelector，选择 Node 的名字即可。

```yaml
nodeSelector:
    name: <Node名字>
```

不过，在 Kubernetes 项目里，nodeSelector 其实已经是一个将要被废弃的字段了。

因为，**现在有了一个新的、功能更完善的字段可以代替它，即：nodeAffinity**。我来举个例子：

```yaml

apiVersion: v1
kind: Pod
metadata:
  name: with-node-affinity
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: metadata.name
            operator: In
            values:
            - node-geektime
            
# spec.affinity 字段，是 Pod 里跟调度相关的一个字段
# 而在这里，我定义的 nodeAffinity 的含义是：requiredDuringSchedulingIgnoredDuringExecution：
# 1. 它的意思是说，这个 nodeAffinity 必须在每次调度的时候予以考虑。同时，这也意味着你可以设置在某些情况下不考虑这个 nodeAffinity；
# 2. 这个 Pod，将来只允许运行在“metadata.name”是“node-geektime”的节点上。

# 在这里，你应该注意到 nodeAffinity 的定义，可以支持更加丰富的语法，比如 operator: In（即：部分匹配；如果你定义 operator: Equal，就是完全匹配），这也正是 nodeAffinity 会取代 nodeSelector 的原因之一。
```

所以，**我们的 DaemonSet Controller 会在创建 Pod 的时候，自动在这个 Pod 的 API 对象里，加上这样一个 nodeAffinity 定义**。其中，需要绑定的节点名字，正是当前正在遍历的这个 Node。



**DaemonSet 还会给这个 Pod 自动加上另外一个与调度相关的字段，叫作 tolerations**。这个字段意味着这个 Pod，会“容忍”（Toleration）某些 Node 的“污点”（Taint）。

```yaml

apiVersion: v1
kind: Pod
metadata:
  name: with-toleration
spec:
  tolerations:
  - key: node.kubernetes.io/unschedulable
    operator: Exists
    effect: NoSchedule
```

这个 Toleration 的含义是：“容忍”所有被标记为 unschedulable“污点”的 Node；“容忍”的效果是允许调度。

而在正常情况下，被标记了 unschedulable“污点”的 Node，是不会有任何 Pod 被调度上去的（effect: NoSchedule）。可是，DaemonSet 自动地给被管理的 Pod 加上了这个特殊的 Toleration，就使得这些 Pod 可以忽略这个限制，继而保证每个节点上都会被调度一个 Pod。当然，如果这个节点有故障的话，这个 Pod 可能会启动失败，而 DaemonSet 则会始终尝试下去，直到 Pod 启动成功。



至此，通过上面这些内容，你应该能够明白，**DaemonSet 其实是一个非常简单的控制器**。

在它的控制循环中，只需要遍历所有节点，然后根据节点上是否有被管理 Pod 的情况，来决定是否要创建或者删除一个 Pod。

只不过，在创建每个 Pod 的时候，DaemonSet 会自动给这个 Pod 加上一个 nodeAffinity，从而保证这个 Pod 只会在指定节点上启动。同时，它还会自动给这个 Pod 加上一个 Toleration，从而忽略节点的 unschedulable“污点”。当然，你也可以在 Pod 模板里加上更多种类的 Toleration，从而利用 DaemonSet 达到自己的目的。



**ControllerRevision**

而我在前面的文章中讲解 Deployment 对象的时候，曾经提到过，**Deployment 管理这些版本，靠的是“一个版本对应一个 ReplicaSet 对象”**。

可是，DaemonSet 控制器操作的直接就是 Pod，不可能有 ReplicaSet 这样的对象参与其中。那么，**它的这些版本又是如何维护的呢？**

所谓，**一切皆对象**！

在 Kubernetes 项目中，任何你觉得需要记录下来的状态，都可以被用 API 对象的方式实现。当然，“版本”也不例外。Kubernetes v1.7 之后添加了一个 API 对象，名叫 **ControllerRevision，专门用来记录某种 Controller 对象的版本**。比如，你可以通过如下命令查看 fluentd-elasticsearch 对应的 ControllerRevision：

```yaml

$ kubectl get controllerrevision -n kube-system -l name=fluentd-elasticsearch
NAME                               CONTROLLER                             REVISION   AGE
fluentd-elasticsearch-64dc6799c9   daemonset.apps/fluentd-elasticsearch   2          1h

# 而如果你使用 kubectl describe 查看这个 ControllerRevision 对象：

$ kubectl describe controllerrevision fluentd-elasticsearch-64dc6799c9 -n kube-system
Name:         fluentd-elasticsearch-64dc6799c9
Namespace:    kube-system
Labels:       controller-revision-hash=2087235575
              name=fluentd-elasticsearch
Annotations:  deprecated.daemonset.template.generation=2
              kubernetes.io/change-cause=kubectl set image ds/fluentd-elasticsearch fluentd-elasticsearch=k8s.gcr.io/fluentd-elasticsearch:v2.2.0 --record=true --namespace=kube-system
API Version:  apps/v1
Data:
  Spec:
    Template:
      $ Patch:  replace
      Metadata:
        Creation Timestamp:  <nil>
        Labels:
          Name:  fluentd-elasticsearch
      Spec:
        Containers:
          Image:              k8s.gcr.io/fluentd-elasticsearch:v2.2.0
          Image Pull Policy:  IfNotPresent
          Name:               fluentd-elasticsearch
...
Revision:                  2
Events:                    <none>

# 就会看到，这个 ControllerRevision 对象，实际上是在 Data 字段保存了该版本对应的完整的 DaemonSet 的 API 对象。并且，在 Annotation 字段保存了创建这个对象所使用的 kubectl 命令。

# 接下来，我们可以尝试将这个 DaemonSet 回滚到 Revision=1 时的状态：

$ kubectl rollout undo daemonset fluentd-elasticsearch --to-revision=1 -n kube-system
daemonset.extensions/fluentd-elasticsearch rolled back
```





### D6: 撬动离线业务：Job 和 CronJob





### D7: 声明式API





### D8: 基于角色的权限控制: RBAC



### D9: Operatpor工作原理















## P3: k8s容器持久化存储













## P4: k8s容器网络



### D1: 浅谈容器网络



#### 不同namespace里的容器如何交互？



**一个 Linux 容器能看见的“网络栈”，实际上是被隔离在它自己的 Network Namespace 当中的。**

而所谓“**网络栈**”，就包括了：

- 网卡（Network Interface）
- 回环设备（Loopback Device）
- 路由表（Routing Table）
- iptables 规则。

对于一个进程来说，这些要素，其实就构成了它发起和响应网络请求的基本环境。



需要指出的是，作为一个容器，它可以**声明直接使用宿主机的网络栈**（–net=host），即：不开启 Network Namespace，比如：

```linux
$ docker run –d –net=host --name nginx-host nginx
```

在这种情况下，这个容器启动后，直接监听的就是宿主机的 80 端口。

像这样直接使用宿主机网络栈的方式，虽然可以为容器提供良好的网络性能，但也会不可避免地引入共享网络资源的问题，比如端口冲突。

所以，在大多数情况下，我们都**希望容器进程能使用自己 Network Namespace 里的网络栈**，即：拥有属于自己的 IP 地址和端口。



这时候，一个显而易见的问题就是：**这个被隔离的容器进程，该如何跟其他 Network Namespace 里的容器进程进行交互呢**？



在 Linux 中，能够起到虚拟交换机作用的网络设备，是**网桥（Bridge）**。它是一个工作在数据链路层（Data Link）的设备，主要功能是根据 MAC 地址学习来将数据包转发到网桥的不同端口（Port）上

而为了实现上述目的，Docker 项目会默认**在宿主机上创建一个名叫 docker0 的网桥**，凡是连接在 docker0 网桥上的容器，就可以通过它来进行通信。

可是，我们又该如何把这些容器“连接”到 docker0 网桥上呢？

这时候，我们就需要使用一种名叫 **Veth Pair** 的虚拟设备了。

**Veth Pair 设备的特点是：它被创建出来后，总是以两张虚拟网卡（Veth Peer）的形式成对出现的。**并且，从其中一个“网卡”发出的数据包，可以直接出现在与它对应的另一张“网卡”上，哪怕这两个“网卡”在不同的 Network Namespace 里。

这就使得 Veth Pair 常**常被用作连接不同 Network Namespace 的“网线”**。





**在默认情况下，被限制在 Network Namespace 里的容器进程，实际上是通过 Veth Pair 设备 + 宿主机网桥的方式，实现了跟同其他容器的数据交换。**

与之类似地，当你**在一台宿主机上，访问该宿主机上的容器的 IP 地址时**，这个请求的数据包，也是先根据路由规则到达 docker0 网桥，然后被转发到对应的 Veth Pair 设备，最后出现在容器里。

同样地，**当一个容器试图连接到另外一个宿主机时**，比如：ping 10.168.0.3，它发出的请求数据包，首先经过 docker0 网桥出现在宿主机上。然后根据宿主机的路由表里的直连路由规则（10.168.0.0/24 via eth0)），对 10.168.0.3 的访问请求就会交给宿主机的 eth0 处理。所以接下来，这个数据包就会经宿主机的 eth0 网卡转发到宿主机网络上，最终到达 10.168.0.3 对应的宿主机上。当然，这个过程的实现要求这两台宿主机本身是连通的。

**所以说，当你遇到容器连不通“外网”的时候，你都应该先试试 docker0 网桥能不能 ping 通，然后查看一下跟 docker0 和 Veth Pair 设备相关的 iptables 规则是不是有异常，往往就能够找到问题的答案了。**





### D2: 跨主通信



**如果在另外一台宿主机（比如：10.168.0.3）上，也有一个 Docker 容器。那么，我们的 nginx-1 容器又该如何访问它呢？**

这个问题，其实就是容器的“**跨主通信**”问题。

如果我们通过软件的方式，创建一个整个集群“公用”的网桥，然后把集群里的所有容器都连接到这个网桥上，不就可以相互通信了吗？

![image-20220207145449837](/Users/breeze/Library/Application Support/typora-user-images/image-20220207145449837.png)

可以看到，构建这种容器网络的核心在于：我们需要在已有的宿主机网络上，再通过软件构建一个覆盖在已有宿主机网络之上的、可以把所有容器连通在一起的虚拟网络。所以，这种技术就被称为：**Overlay Network（覆盖网络）**。

而这个 Overlay Network 本身，可以由每台宿主机上的一个“特殊网桥”共同组成。

比如，当 Node 1 上的 Container 1 要访问 Node 2 上的 Container 3 的时候，Node 1 上的“特殊网桥”在收到数据包之后，能够通过某种方式，把数据包发送到正确的宿主机，比如 Node 2 上。而 Node 2 上的“特殊网桥”在收到数据包后，也能够通过某种方式，把数据包转发给正确的容器，比如 Container 3。

甚至，每台宿主机上，都不需要有一个这种特殊的网桥，而仅仅通过某种方式配置宿主机的路由表，就能够把数据包转发到正确的宿主机上。





要理解容器“跨主通信”的原理，就一定要先从 **Flannel** 这个项目说起。

Flannel 项目是 CoreOS 公司主推的容器网络方案。事实上，Flannel 项目本身只是一个框架，真正为我们提供容器网络功能的，是 Flannel 的后端实现。

目前，**Flannel 支持三种后端实现**，分别是：

- VXLAN；
- host-gw；
- UDP；

这三种不同的后端实现，正代表了三种容器跨主网络的主流实现方法。 



#### UDP模式

UDP 模式，是 Flannel 项目最早支持的一种方式，却也是**性能最差**的一种方式。所以，这个模式目前已经被弃用。

不过，Flannel 之所以最先选择 UDP 模式，就是因为这种模式是**最直接、也是最容易理解**的容器跨主网络实现。



**举个例子：**我有两台宿主机。

> 宿主机 Node 1 上有一个容器 container-1，它的 IP 地址是 100.96.1.2，对应的 docker0 网桥的地址是：100.96.1.1/24。
>
> 宿主机 Node 2 上有一个容器 container-2，它的 IP 地址是 100.96.2.3，对应的 docker0 网桥的地址是：100.96.2.1/24。

我们现在的任务，就是**让 container-1 访问 container-2**。

这种情况下，container-1 容器里的进程发起的 IP 包，其源地址就是 100.96.1.2，目的地址就是 100.96.2.3。由于目的地址 100.96.2.3 并不在 Node 1 的 docker0 网桥的网段里，所以这个 **IP 包会被交给默认路由规则，通过容器的网关进入 docker0 网桥（如果是同一台宿主机上的容器间通信，走的是直连规则），从而出现在宿主机上**。

这时候，这个 IP 包的下一个目的地，就取决于宿主机上的路由规则了。此时，Flannel 已经在宿主机上创建出了一系列的路由规则，以 Node 1 为例，如下所示：

```yaml
# 在Node 1上
$ ip route
default via 10.168.0.1 dev eth0
100.96.0.0/16 dev flannel0  proto kernel  scope link  src 100.96.1.0
100.96.1.0/24 dev docker0  proto kernel  scope link  src 100.96.1.1
10.168.0.0/24 dev eth0  proto kernel  scope link  src 10.168.0.2
```

可以看到，由于我们的 IP 包的目的地址是 100.96.2.3，它匹配不到本机 docker0 网桥对应的 100.96.1.0/24 网段，只能匹配到第二条、也就是 100.96.0.0/16 对应的这条路由规则，从而进入到一个叫作 **flannel0** 的设备中。

而这个 **flannel0 设备**的类型就比较有意思了：它是一个 **TUN 设备（Tunnel 设备）**。

在 Linux 中，**TUN 设备是一种工作在三层（Network Layer）的虚拟网络设备**。TUN 设备的功能非常简单，即：**在操作系统内核和用户应用程序之间传递 IP 包**。

以 flannel0 设备为例：

- 像上面提到的情况，当操作系统将一个 IP 包发送给 flannel0 设备之后，flannel0 就会把这个 IP 包，交给创建这个设备的应用程序，也就是 Flannel 进程。这是一个从**内核态（Linux 操作系统）向用户态（Flannel 进程）的流动方向**。
- 反之，如果 Flannel 进程向 flannel0 设备发送了一个 IP 包，那么这个 IP 包就会出现在宿主机网络栈中，然后根据宿主机的路由表进行下一步处理。**这是一个从用户态向内核态的流动方向**。

所以，当 IP 包从容器经过 docker0 出现在宿主机，然后又根据路由表进入 flannel0 设备后，**宿主机上的 flanneld 进程（Flannel 项目在每个宿主机上的主进程）**，就会收到这个 IP 包。然后，flanneld 看到了这个 IP 包的目的地址，是 100.96.2.3，就把它发送给了 Node 2 宿主机。



**flanneld 又是如何知道这个 IP 地址对应的容器，是运行在 Node 2 上的呢？**

这里，就用到了 Flannel 项目里一个非常重要的概念：**子网（Subnet）**。

> **事实上，在由 Flannel 管理的容器网络里，一台宿主机上的所有容器，都属于该宿主机被分配的一个“子网”。**

在我们的例子中，Node 1 的子网是 100.96.1.0/24，container-1 的 IP 地址是 100.96.1.2。

Node 2 的子网是 100.96.2.0/24，container-2 的 IP 地址是 100.96.2.3。

而这些**子网与宿主机的对应关系，正是保存在 Etcd 当中**，如下所示：

```linux
$ etcdctl ls /coreos.com/network/subnets
/coreos.com/network/subnets/100.96.1.0-24
/coreos.com/network/subnets/100.96.2.0-24
/coreos.com/network/subnets/100.96.3.0-24
```

所以，flanneld 进程在处理由 flannel0 传入的 IP 包时，就可以根据目的 IP 的地址（比如 100.96.2.3），匹配到对应的子网（比如 100.96.2.0/24），从 Etcd 中找到这个子网对应的宿主机的 IP 地址是 10.168.0.3，如下所示：

```linux
$ etcdctl get /coreos.com/network/subnets/100.96.2.0-24
{"PublicIP":"10.168.0.3"}
```

而对于 flanneld 来说，只要 Node 1 和 Node 2 是互通的，那么 flanneld 作为 Node 1 上的一个普通进程，就一定可以通过上述 IP 地址（10.168.0.3）访问到 Node 2，这没有任何问题。

所以说，flanneld 在收到 container-1 发给 container-2 的 IP 包之后，就会把这个 IP 包直接封装在一个 UDP 包里，然后发送给 Node 2。不难理解，这个 UDP 包的源地址，就是 flanneld 所在的 Node 1 的地址，而目的地址，则是 container-2 所在的宿主机 Node 2 的地址。

当然，这个请求得以完成的原因是，**每台宿主机上的 flanneld，都监听着一个 8285 端口**，所以 flanneld 只要把 UDP 包发往 Node 2 的 8285 端口即可。

通过这样一个普通的、宿主机之间的 UDP 通信，**一个 UDP 包就从 Node 1 到达了 Node 2**。而 Node 2 上监听 8285 端口的进程也是 flanneld，所以这时候，**flanneld 就可以从这个 UDP 包里解析出封装在里面的、container-1 发来的原 IP 包**。

而接下来 flanneld 的工作就非常简单了：**flanneld 会直接把这个 IP 包发送给它所管理的 TUN 设备，即 flannel0 设备**。根据我前面讲解的 TUN 设备的原理，这正是一个从用户态向内核态的流动方向（Flannel 进程向 TUN 设备发送数据包），所以 **Linux 内核网络栈就会负责处理这个 IP 包，具体的处理方法，就是通过本机的路由表来寻找这个 IP 包的下一步流向**。而 Node 2 上的路由表，跟 Node 1 非常类似，如下所示：

```yaml
# 在Node 2上
$ ip route
default via 10.168.0.1 dev eth0
100.96.0.0/16 dev flannel0  proto kernel  scope link  src 100.96.2.0
100.96.2.0/24 dev docker0  proto kernel  scope link  src 100.96.2.1
10.168.0.0/24 dev eth0  proto kernel  scope link  src 10.168.0.3
```

由于这个 IP 包的目的地址是 100.96.2.3，它跟第三条、也就是 100.96.2.0/24 网段对应的路由规则匹配更加精确。所以，Linux 内核就会按照这条路由规则，把这个 IP 包转发给 docker0 网桥。

接下来的流程，就如同我在上一篇文章中那样，docker0 网桥会扮演二层交换机的角色，将数据包发送给正确的端口，进而通过 Veth Pair 设备进入到 container-2 的 Network Namespace 里。而 container-2 返回给 container-1 的数据包，则会经过与上述过程完全相反的路径回到 container-1 中。

> 上述流程要正确工作还有一个重要的前提，那就是 docker0 网桥的地址范围必须是 Flannel 为宿主机分配的子网。这个很容易实现，以 Node 1 为例，你只需要给它上面的 Docker Daemon 启动时配置如下所示的 bip 参数即可：
>
> ```
> $ FLANNEL_SUBNET=100.96.1.1/24
> $ dockerd --bip=$FLANNEL_SUBNET ...
> ```



**以上，就是基于 Flannel UDP 模式的跨主通信的基本原理了。**

![image-20220207160623576](/Users/breeze/Library/Application Support/typora-user-images/image-20220207160623576.png)

可以看到，**Flannel UDP 模式提供的其实是一个三层的 Overlay 网络**，即：它首先对发出端的 IP 包进行 UDP 封装，然后在接收端进行解封装拿到原始的 IP 包，进而把这个 IP 包转发给目标容器。这就好比，Flannel 在不同宿主机上的两个容器之间打通了一条“隧道”，使得这两个容器可以直接使用 IP 地址进行通信，而无需关心容器和宿主机的分布情况。



**UDP模式存在的严重性能问题**

相比于两台宿主机之间的直接通信，基于 Flannel UDP 模式的容器通信多了一个额外的步骤，即 flanneld 的处理过程。而这个过程，由于使用到了 flannel0 这个 TUN 设备，仅在发出 IP 包的过程中，就需要经过三次用户态与内核态之间的数据拷贝，如下所示：

<img src="/Users/breeze/Library/Application Support/typora-user-images/image-20220207160947508.png" alt="image-20220207160947508" style="zoom:50%;" />

我们可以看到：

**第一次**，用户态的容器进程发出的 IP 包经过 docker0 网桥进入内核态；

**第二次**，IP 包根据路由表进入 TUN（flannel0）设备，从而回到用户态的 flanneld 进程；

**第三次**，flanneld 进行 UDP 封包之后重新进入内核态，将 UDP 包通过宿主机的 eth0 发出去。

此外，我们还可以看到，Flannel 进行 UDP 封装（Encapsulation）和解封装（Decapsulation）的过程，也都是在用户态完成的。在 Linux 操作系统中，上述这些上下文切换和用户态操作的代价其实是比较高的，这也正是造成 Flannel UDP 模式性能不好的主要原因。

所以说，我们**在进行系统级编程的时候，有一个非常重要的优化原则，就是要减少用户态到内核态的切换次数，并且把核心的处理逻辑都放在内核态进行**。这也是为什么，Flannel 后来支持的**VXLAN** 模式，逐渐成为了主流的容器网络方案的原因。



#### VXLAN模式

VXLAN，即 Virtual Extensible LAN（虚拟可扩展局域网），是 Linux 内核本身就支持的一种网络虚似化技术。所以说，**VXLAN 可以完全在内核态实现上述封装和解封装的工作，从而通过与前面相似的“隧道”机制，构建出覆盖网络（Overlay Network）**。

VXLAN 的覆盖网络的**设计思想**是：**在现有的三层网络之上，“覆盖”一层虚拟的、由内核 VXLAN 模块负责维护的二层网络，使得连接在这个 VXLAN 二层网络上的“主机”（虚拟机或者容器都可以）之间，可以像在同一个局域网（LAN）里那样自由通信。**

当然，实际上，这些“主机”可能分布在不同的宿主机上，甚至是分布在不同的物理机房里。

而为了能够在二层网络上打通“隧道”，**VXLAN 会在宿主机上设置一个特殊的网络设备作为“隧道”的两端。这个设备就叫作 VTEP，即：VXLAN Tunnel End Point（虚拟隧道端点）**。

而 VTEP 设备的作用，其实跟前面的 flanneld 进程非常相似。只不过，它进行封装和解封装的对象，是二层数据帧（Ethernet frame）；而且这个工作的执行流程，**全部是在内核里完成的（因为 VXLAN 本身就是 Linux 内核中的一个模块）**。上述基于 VTEP 设备进行“隧道”通信的流程，我也为你总结成了一幅图，如下所示：

![image-20220207204301335](/Users/breeze/Library/Application Support/typora-user-images/image-20220207204301335.png)

图中**每台宿主机上名叫 flannel.1 的设备，就是 VXLAN 所需的 VTEP 设备，它既有 IP 地址，也有 MAC 地址。**



现在，我们的 **container-1 的 IP 地址是 10.1.15.2，要访问的 container-2 的 IP 地址是 10.1.16.3**。

那么，与前面 UDP 模式的流程类似，当 container-1 发出请求之后，这个目的地址是 10.1.16.3 的 IP 包，**会先出现在 docker0 网桥，然后被路由到本机 flannel.1 设备进行处理**。

也就是说，来到了“隧道”的入口。为了方便叙述，我接下来会把这个 IP 包称为“原始 IP 包”。为了能够将“原始 IP 包”封装并且发送到正确的宿主机，VXLAN 就需要找到这条“隧道”的出口，即：目的宿主机的 VTEP 设备。

**而这个VTEP设备的信息，正是每台宿主机上的 flanneld 进程负责维护的**。

比如，当 Node 2 启动并加入 Flannel 网络之后，在 Node 1（以及所有其他节点）上，flanneld 就会添加一条如下所示的**路由规则**：

```shell

$ route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
...
10.1.16.0       10.1.16.0       255.255.255.0   UG    0      0        0 flannel.1
```

这条规则的意思是：凡是发往 10.1.16.0/24 网段的 IP 包，都需要经过 flannel.1 设备发出，并且，它最后被发往的网关地址是：10.1.16.0。

从图 3 的 Flannel VXLAN 模式的流程图中我们可以看到，10.1.16.0 正是 Node 2 上的 VTEP 设备（也就是 flannel.1 设备）的 IP 地址。

为了方便叙述，接下来我会把 Node 1 和 Node 2 上的 flannel.1 设备分别称为“源 VTEP 设备”和“目的 VTEP 设备”。而这些 VTEP 设备之间，就需要想办法组成一个虚拟的二层网络，即：通过二层数据帧进行通信。

所以在我们的例子中，**“源 VTEP 设备”收到“原始 IP 包”后，就要想办法把“原始 IP 包”加上一个目的 MAC 地址，封装成一个二层数据帧，然后发送给“目的 VTEP 设备”**（当然，这么做还是因为这个 IP 包的目的地址不是本机）。

这里需要解决的问题就是：**“目的 VTEP 设备”的 MAC 地址是什么？**

此时，根据前面的路由记录，我们已经知道了“目的 VTEP 设备”的 IP 地址。而要**根据三层 IP 地址查询对应的二层 MAC 地址，这正是 ARP（Address Resolution Protocol ）表的功能**。而**这里要用到的 ARP 记录，也是 flanneld 进程在 Node 2 节点启动时，自动添加在 Node 1 上的**。我们可以通过 ip 命令看到它，如下所示：

```shell

# 在Node 1上
$ ip neigh show dev flannel.1
10.1.16.0 lladdr 5e:f8:4f:00:e3:37 PERMANENT

#	这条记录的意思非常明确，即：IP 地址 10.1.16.0，对应的 MAC 地址是 5e:f8:4f:00:e3:37。

# 可以看到，最新版本的 Flannel 并不依赖 L3 MISS 事件和 ARP 学习，而会在每台节点启动时把它的 VTEP 设备对应的 ARP 记录，直接下放到其他每台宿主机上。	
```



有了这个“目的 VTEP 设备”的 MAC 地址，Linux 内核就可以开始二层封包工作了。这个二层帧的格式，如下所示：

<img src="/Users/breeze/Library/Application Support/typora-user-images/image-20220208185954547.png" alt="image-20220208185954547" style="zoom:50%;" />

可以看到，Linux 内核会**把“目的 VTEP 设备”的 MAC 地址，填写在图中的 Inner Ethernet Header 字段，得到一个二层数据帧**。

需要注意的是，上述封包过程只是加一个二层头，不会改变“原始 IP 包”的内容。所以图中的 Inner IP Header 字段，依然是 container-2 的 IP 地址，即 10.1.16.3。

但是，上面提到的这些 **VTEP 设备的 MAC 地址，对于宿主机网络来说并没有什么实际意义**。所以上面封装出来的这个数据帧，并不能在我们的宿主机二层网络里传输。为了方便叙述，我们把它称为**“内部数据帧”（Inner Ethernet Frame）**。

所以接下来，Linux 内核还需要再把“内部数据帧”进一步封装成为宿主机网络里的一个普通的数据帧，好让它“载着”“内部数据帧”，通过宿主机的 eth0 网卡进行传输。我们把这次要封装出来的、宿主机对应的数据帧称为**“外部数据帧”（Outer Ethernet Frame）**。

为了实现这个“搭便车”的机制，**Linux 内核会在“内部数据帧”前面，加上一个特殊的 VXLAN 头，用来表示这个“乘客”实际上是一个 VXLAN 要使用的数据帧**。

而这个 VXLAN 头里有一个**重要的标志叫作 VNI**，它是 VTEP 设备识别某个数据帧是不是应该归自己处理的重要标识。而在 Flannel 中，VNI 的默认值是 1，这也是为何，宿主机上的 VTEP 设备都叫作 flannel.1 的原因，这里的“1”，其实就是 VNI 的值。

**然后，Linux 内核会把这个数据帧封装进一个 UDP 包里发出去。**

所以，跟 UDP 模式类似，在宿主机看来，它会以为自己的 flannel.1 设备只是在向另外一台宿主机的 flannel.1 设备，发起了一次普通的 UDP 链接。它哪里会知道，这个 UDP 包里面，其实是一个完整的二层数据帧。这是不是跟特洛伊木马的故事非常像呢？



不过，不要忘了，**一个 flannel.1 设备只知道另一端的 flannel.1 设备的 MAC 地址，却不知道对应的宿主机地址是什么。**

**也就是说，这个 UDP 包该发给哪台宿主机呢？**



在这种场景下，**flannel.1 设备实际上要扮演一个“网桥”的角色，在二层网络进行 UDP 包的转发**。而在 Linux 内核里面，“网桥”设备进行转发的依据，来自于一个叫作 **FDB（Forwarding Database）的转发数据库**。不难想到，这个 flannel.1“网桥”对应的 FDB 信息，也是 flanneld 进程负责维护的。它的内容可以通过 bridge fdb 命令查看到，如下所示：

```shell

# 在Node 1上，使用“目的VTEP设备”的MAC地址进行查询
$ bridge fdb show flannel.1 | grep 5e:f8:4f:00:e3:37
5e:f8:4f:00:e3:37 dev flannel.1 dst 10.168.0.3 self permanent
```

可以看到，在上面这条 FDB 记录里，指定了这样一条规则，即：

发往我们前面提到的“目的 VTEP 设备”（MAC 地址是 5e:f8:4f:00:e3:37）的二层数据帧，应该通过 flannel.1 设备，发往 IP 地址为 10.168.0.3 的主机。显然，这台主机正是 Node 2，UDP 包要发往的目的地就找到了。



所以**接下来的流程，就是一个正常的、宿主机网络上的封包工作**。



我们知道，UDP 包是一个四层数据包，所以 **Linux 内核会在它前面加上一个 IP 头**，即原理图中的 Outer IP Header，组成一个 IP 包。

并且，在这个 IP 头里，会**填上前面通过 FDB 查询出来的目的主机的 IP 地址**，即 Node 2 的 IP 地址 10.168.0.3。

然后，Linux 内核再在这个 IP 包前面**加上二层数据帧头，即原理图中的 Outer Ethernet Header，并把 Node 2 的 MAC 地址填进去**。这个 MAC 地址本身，是 Node 1 的 ARP 表要学习的内容，无需 Flannel 维护。

这时候，我们封装出来的“**外部数据帧**”的格式，如下所示：

<img src="/Users/breeze/Library/Application Support/typora-user-images/image-20220208191131350.png" alt="image-20220208191131350" style="zoom:50%;" />

这样，封包工作就宣告完成了。

接下来，Node 1 上的 flannel.1 设备就可以把这个数据帧从 Node 1 的 eth0 网卡发出去。

显然，这个帧会经过宿主机网络来到 Node 2 的 eth0 网卡。

这时候，Node 2 的内核网络栈会发现这个数据帧里有 VXLAN Header，并且 VNI=1。所以 Linux 内核会对它进行拆包，拿到里面的内部数据帧，然后根据 VNI 的值，把它交给 Node 2 上的 flannel.1 设备。而 flannel.1 设备则会进一步拆包，取出“原始 IP 包”。

接下来就回到了上一篇文章中的单机容器网络的处理流程。

最终，IP 包就进入到了 container-2 容器的 Network Namespace 里。

以上，就是 **Flannel VXLAN 模式**的具体工作原理了。





### D3: k8s网络模型和CNI网络插件

在上一篇文章中，以 Flannel 项目为例，讲解了容器跨主机网络的两种实现方法：**UDP 和 VXLAN**。

不难看到，这些例子有一个共性，那就是**用户的容器都连接在 docker0 网桥上**。而**网络插件则在宿主机上创建了一个特殊的设备（UDP 模式创建的是 TUN 设备，VXLAN 模式创建的则是 VTEP 设备）**，**docker0 与这个设备之间，通过 IP 转发（路由表）进行协作**。

然后，**网络插件**真正要做的事情，则是通过某种方法，**把不同宿主机上的特殊设备连通，从而达到容器跨主机通信的目的**。



#### CNI网桥

实际上，上面这个流程，也正是 Kubernetes 对容器网络的主要处理方法。只不过，Kubernetes 是通过一个叫作 CNI 的接口，维护了一个单独的网桥来代替 docker0。这个网桥的名字就叫作：**CNI 网桥**，它在宿主机上的设备名称默认是：**cni0**。

以 Flannel 的 VXLAN 模式为例，在 Kubernetes 环境里，它的工作方式跟我们在上一篇文章中没有任何不同。只不过，docker0 网桥被替换成了 CNI 网桥而已，如下所示：

<img src="/Users/breeze/Library/Application Support/typora-user-images/image-20220208192012026.png" alt="image-20220208192012026" style="zoom:50%;" />

在这里，Kubernetes 为 Flannel 分配的子网范围是 10.244.0.0/16。这个参数可以在部署的时候指定，比如：

```shell

$ kubeadm init --pod-network-cidr=10.244.0.0/16
```

也可以在部署完成后，通过修改 kube-controller-manager 的配置文件来指定。

这时候，假设 Infra-container-1 要访问 Infra-container-2（也就是 Pod-1 要访问 Pod-2），这个 IP 包的源地址就是 10.244.0.2，目的 IP 地址是 10.244.1.3。

而此时，Infra-container-1 里的 eth0 设备，同样是以 Veth Pair 的方式连接在 Node 1 的 cni0 网桥上。所以这个 IP 包就会经过 cni0 网桥出现在宿主机上。

此时，Node 1 上的路由表，如下所示：    

```shell

# 在Node 1上
$ route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
...
10.244.0.0      0.0.0.0         255.255.255.0   U     0      0        0 cni0
10.244.1.0      10.244.1.0      255.255.255.0   UG    0      0        0 flannel.1
172.17.0.0      0.0.0.0         255.255.0.0     U     0      0        0 docker0
```

因为我们的 IP 包的目的 IP 地址是 10.244.1.3，所以它只能匹配到第二条规则，也就是 10.244.1.0 对应的这条路由规则。

可以看到，这条规则指定了本机的 flannel.1 设备进行处理。并且，flannel.1 在处理完后，要将 IP 包转发到的网关（Gateway），正是“隧道”另一端的 VTEP 设备，也就是 Node 2 的 flannel.1 设备。

所以，接下来的流程，就跟上一篇文章中介绍过的 Flannel VXLAN 模式完全一样了。

**需要注意的是，CNI 网桥只是接管所有 CNI 插件负责的、即 Kubernetes 创建的容器（Pod）**。而此时，如果你用 docker run 单独启动一个容器，那么 Docker 项目还是会把这个容器连接到 docker0 网桥上。所以这个容器的 IP 地址，一定是属于 docker0 网桥的 172.17.0.0/16 网段。

Kubernetes 之所以要设置这样一个与 docker0 网桥功能几乎一样的 CNI 网桥，

主要原因包括两个方面：

- 一方面，Kubernetes 项目**并没有使用 Docker 的网络模型（CNM）**，所以它并不希望、也不具备配置 docker0 网桥的能力；
- 另一方面，这还**与 Kubernetes 如何配置 Pod，也就是 Infra 容器的 Network Namespace 密切相关**。

我们知道，Kubernetes 创建一个 Pod 的第一步，就是创建并启动一个 Infra 容器，用来“hold”住这个 Pod 的 Network Namespace。

所以，**CNI 的设计思想**，就是：**Kubernetes 在启动 Infra 容器之后，就可以直接调用 CNI 网络插件，为这个 Infra 容器的 Network Namespace，配置符合预期的网络栈**。

> 一个 Network Namespace 的网络栈包括：网卡（Network Interface）、回环设备（Loopback Device）、路由表（Routing Table）和 iptables 规则。



#### 这个网络栈的配置工作又是如何完成的呢？

> 为了回答这个问题，我们就需要从 **CNI 插件的部署和实现方式**谈起了。



我们在部署 Kubernetes 的时候，有一个步骤是**安装 kubernetes-cni 包**，它的**目的就是在宿主机上安装 CNI 插件所需的基础可执行文件**。

在安装完成后，你可以在宿主机的 **/opt/cni/bin** 目录下看到它们，如下所示：

```shell

$ ls -al /opt/cni/bin/
total 73088
-rwxr-xr-x 1 root root  3890407 Aug 17  2017 bridge
-rwxr-xr-x 1 root root  9921982 Aug 17  2017 dhcp
-rwxr-xr-x 1 root root  2814104 Aug 17  2017 flannel
-rwxr-xr-x 1 root root  2991965 Aug 17  2017 host-local
-rwxr-xr-x 1 root root  3475802 Aug 17  2017 ipvlan
-rwxr-xr-x 1 root root  3026388 Aug 17  2017 loopback
-rwxr-xr-x 1 root root  3520724 Aug 17  2017 macvlan
-rwxr-xr-x 1 root root  3470464 Aug 17  2017 portmap
-rwxr-xr-x 1 root root  3877986 Aug 17  2017 ptp
-rwxr-xr-x 1 root root  2605279 Aug 17  2017 sample
-rwxr-xr-x 1 root root  2808402 Aug 17  2017 tuning
-rwxr-xr-x 1 root root  3475750 Aug 17  2017 vlan
```

这些 CNI 的基础可执行文件，按照功能可以分为三类：

- 第一类，叫作 **Main 插件**，它是<u>用来创建具体网络设备的二进制文件</u>。比如，bridge（网桥设备）、ipvlan、loopback（lo 设备）、macvlan、ptp（Veth Pair 设备），以及 vlan。我在前面提到过的 Flannel、Weave 等项目，都属于“网桥”类型的 CNI 插件。所以在具体的实现中，它们往往会调用 bridge 这个二进制文件。这个流程，我马上就会详细介绍到。
- 第二类，叫作 **IPAM（IP Address Management）插件**，它是<u>负责分配 IP 地址的二进制文件</u>。比如，dhcp，这个文件会向 DHCP 服务器发起请求；host-local，则会使用预先配置的 IP 地址段来进行分配。
- 第三类，是由 **CNI 社区维护的内置 CNI 插件**。比如：flannel，就是专门为 Flannel 项目提供的 CNI 插件；tuning，是一个通过 sysctl 调整网络设备参数的二进制文件；portmap，是一个通过 iptables 配置端口映射的二进制文件；bandwidth，是一个使用 Token Bucket Filter (TBF) 来进行限流的二进制文件。



从这些二进制文件中，我们可以看到，如果要实现一个给 Kubernetes 用的容器网络方案，其实需要做两部分工作，

以 Flannel 项目为例：

- 首先，**实现这个网络方案本身**。这一部分需要编写的，其实就是 flanneld 进程里的主要逻辑。比如，创建和配置 flannel.1 设备、配置宿主机路由、配置 ARP 和 FDB 表里的信息等等。
- 然后，**实现该网络方案对应的 CNI 插件**。这一部分主要需要做的，就是配置 Infra 容器里面的网络栈，并把它连接在 CNI 网桥上。

由于 Flannel 项目对应的 CNI 插件已经被内置了，所以它无需再单独安装。而对于 Weave、Calico 等其他项目来说，我们就必须在安装插件的时候，把对应的 CNI 插件的可执行文件放在 /opt/cni/bin/ 目录下。

> 实际上，对于 Weave、Calico 这样的网络方案来说，它们的 DaemonSet 只需要挂载宿主机的 /opt/cni/bin/，就可以实现插件可执行文件的安装了。

接下来，你就需要在宿主机上安装 flanneld（网络方案本身）。而在这个过程中，flanneld 启动后会在每台宿主机上生成它对应的 CNI 配置文件（它其实是一个 ConfigMap），从而告诉 Kubernetes，这个集群要使用 Flannel 作为容器网络方案。这个 CNI 配置文件的内容如下所示：

```shell

$ cat /etc/cni/net.d/10-flannel.conflist 
{
  "name": "cbr0",
  "plugins": [
    {
      "type": "flannel",
      "delegate": {
        "hairpinMode": true,
        "isDefaultGateway": true
      }
    },
    {
      "type": "portmap",
      "capabilities": {
        "portMappings": true
      }
    }
  ]
}
```

需要注意的是，**在 Kubernetes 中，处理容器网络相关的逻辑并不会在 kubelet 主干代码里执行，而是会在具体的 CRI（Container Runtime Interface，容器运行时接口）实现里完成**。

对于 Docker 项目来说，它的 CRI 实现叫作 **dockershim**，你可以在 kubelet 的代码里找到它。所以，接下来 dockershim 会加载上述的 CNI 配置文件。

需要注意，Kubernetes 目前不支持多个 CNI 插件混用。如果你在 CNI 配置目录（/etc/cni/net.d）里放置了多个 CNI 配置文件的话，dockershim 只会加载按字母顺序排序的第一个插件。但另一方面，CNI 允许你在一个 CNI 配置文件里，通过 plugins 字段，定义多个插件进行协作。比如，在我们上面这个例子里，Flannel 项目就指定了 flannel 和 portmap 这两个插件。

这时候，**dockershim 会把这个 CNI 配置文件加载起来，并且把列表里的第一个插件、也就是 flannel 插件，设置为默认插件**。而在<u>后面的执行过程中，flannel 和 portmap 插件会按照定义顺序被调用，从而依次完成“配置容器网络”和“配置端口映射”这两步操作。</u>



#### CNI插件工作原理

接下来，我就来为你讲解一下这样一个 **CNI 插件的工作原理**。

当 kubelet 组件需要创建 Pod 的时候，它第一个创建的一定是 Infra 容器。所以在这一步，dockershim 就会先调用 Docker API 创建并启动 Infra 容器，紧接着执行一个叫作 **SetUpPod** 的方法。这个方法的作用就是：<u>为 CNI 插件准备参数，然后调用 CNI 插件为 Infra 容器配置网络</u>。

这里要调用的 CNI 插件，就是 /opt/cni/bin/flannel；而调用它所需要的参数，分为两部分。

**第一部分，是由 dockershim 设置的一组 CNI 环境变量。**

其中，最重要的环境变量参数叫作：**CNI_COMMAND**。它的取值只有两种：**ADD 和 DEL**。

这个 ADD 和 DEL 操作，就是 CNI 插件唯一需要实现的两个方法。

- ADD 操作的含义是：把容器添加到 CNI 网络里；
- DEL 操作的含义则是：把容器从 CNI 网络里移除掉。

而对于网桥类型的 CNI 插件来说，这两个操作意味着把容器以 Veth Pair 的方式“插”到 CNI 网桥上，或者从网桥上“拔”掉。



接下来，我以 ADD 操作为重点进行讲解。CNI 的 ADD 操作需要的参数包括：容器里网卡的名字 eth0（CNI_IFNAME）、Pod 的 Network Namespace 文件的路径（CNI_NETNS）、容器的 ID（CNI_CONTAINERID）等。这些参数都属于上述环境变量里的内容。

其中，Pod（Infra 容器）的 Network Namespace 文件的路径，我在前面讲解容器基础的时候提到过，即：/proc/< 容器进程的 PID>/ns/net。

除此之外，在 CNI 环境变量里，还有一个叫作 CNI_ARGS 的参数。通过这个参数，CRI 实现（比如 dockershim）就可以以 Key-Value 的格式，传递自定义信息给网络插件。这是用户将来自定义 CNI 协议的一个重要方法。



**第二部分，则是 dockershim 从 CNI 配置文件里加载到的、默认插件的配置信息。**

这个配置信息在 CNI 中被叫作 Network Configuration，它的完整定义你可以参考这个文档。dockershim 会把 Network Configuration 以 JSON 数据的格式，通过标准输入（stdin）的方式传递给 Flannel CNI 插件。而有了这两部分参数，Flannel CNI 插件实现 ADD 操作的过程就非常简单了。不过，需要注意的是，Flannel 的 CNI 配置文件（ /etc/cni/net.d/10-flannel.conflist）里有这么一个字段，叫作 delegate：

```shell

...
     "delegate": {
        "hairpinMode": true,
        "isDefaultGateway": true
      }
```

Delegate 字段的意思是，这个 CNI 插件并不会自己做事儿，而是会调用 Delegate 指定的某种 CNI 内置插件来完成。对于 Flannel 来说，它调用的 Delegate 插件，就是前面介绍到的 CNI bridge 插件。

所以说，dockershim 对 Flannel CNI 插件的调用，其实就是走了个过场。

Flannel CNI 插件唯一需要做的，就是对 dockershim 传来的 Network Configuration 进行补充。比如，将 Delegate 的 Type 字段设置为 bridge，将 Delegate 的 IPAM 字段设置为 host-local 等。经过 Flannel CNI 插件补充后的、完整的 Delegate 字段如下所示：

```shell

{
    "hairpinMode":true,
    "ipMasq":false,
    "ipam":{
        "routes":[
            {
                "dst":"10.244.0.0/16"
            }
        ],
        "subnet":"10.244.1.0/24",
        "type":"host-local"
    },
    "isDefaultGateway":true,
    "isGateway":true,
    "mtu":1410,
    "name":"cbr0",
    "type":"bridge"
}
```













## P5: k8s作业调度与资源管理

> 作为一个容器集群编排与管理项目，Kubernetes 为用户提供的基础设施能力，
>
> 不仅包括了**应用定义和描述**的部分，
>
> 还包括了**对应用的资源管理和调度的处理。**



### D1: k8s的资源模型与资源管理

作为 Kubernetes 的资源管理与调度部分的基础，我们要从它的**资源模型**开始说起。

##### 内存和CPU资源限额

我在前面的文章中已经提到过，在 Kubernetes 里，Pod 是最小的原子调度单位。这也就意味着，所有**跟调度和资源管理相关的属性都应该是属于 Pod 对象的字段**。而这其中最重要的部分，就是 **Pod 的 CPU 和内存配置**，如下所示：

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: frontend
spec:
  containers:
  - name: db
    image: mysql
    env:
    - name: MYSQL_ROOT_PASSWORD
      value: "password"
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "128Mi"
        cpu: "500m"
  - name: wp
    image: wordpress
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "128Mi"
        cpu: "500m"
```

在 Kubernetes 中，像 **CPU** 这样的资源被称作“**可压缩资源**”（compressible resources）。它的**典型特点**是，当可压缩资源不足时，Pod 只会“饥饿”，但不会退出。

而像**内存**这样的资源，则被称作“**不可压缩资源**（incompressible resources）。当不可压缩资源不足时，Pod 就会因为 OOM（Out-Of-Memory）被内核杀掉。

而由于 Pod 可以由多个 Container 组成，所以 **CPU 和内存资源的限额，是要配置在每个 Container 的定义上**的。这样，Pod 整体的资源配置，就由这些 Container 的配置值累加得到。

其中，Kubernetes 里为 CPU 设置的单位是“CPU 的个数”。比如，cpu=1 指的就是，这个 Pod 的 CPU 限额是 1 个 CPU。当然，具体“1 个 CPU”在宿主机上如何解释，是 1 个 CPU 核心，还是 1 个 vCPU，还是 1 个 CPU 的超线程（Hyperthread），完全取决于宿主机的 CPU 实现方式。Kubernetes 只负责保证 Pod 能够使用到“1 个 CPU”的计算能力。此外，Kubernetes 允许你将 CPU 限额设置为分数，比如在我们的例子里，CPU limits 的值就是 500m。所谓 500m，指的就是 500 millicpu，也就是 0.5 个 CPU 的意思。这样，这个 Pod 就会被分配到 1 个 CPU 一半的计算能力。

当然，你也可以直接把这个配置写成 cpu=0.5。但**在实际使用时，我还是推荐你使用 500m 的写法，毕竟这才是 Kubernetes 内部通用的 CPU 表示方式。**

而对于内存资源来说，它的单位自然就是 bytes。Kubernetes 支持你使用 Ei、Pi、Ti、Gi、Mi、Ki（或者 E、P、T、G、M、K）的方式来作为 bytes 的值。比如，在我们的例子里，Memory requests 的值就是 64MiB (2 的 26 次方 bytes) 。这里要注意区分 MiB（mebibyte）和 MB（megabyte）的区别。

> 备注：1Mi=1024**1024；1M=1000*1000
>

此外，不难看到，Kubernetes 里 Pod 的 CPU 和内存资源，实际上还要分为 **limits 和 requests** 两种情况，如下所示：

```yaml
spec.containers[].resources.limits.cpu
spec.containers[].resources.limits.memory
spec.containers[].resources.requests.cpu
spec.containers[].resources.requests.memory
```

这两者的区别其实非常简单：

- 在**调度**的时候，kube-scheduler 只会按照 **requests** 的值进行计算。
- 而在真正设置 Cgroups 限制的时候，kubelet 则会按照 limits 的值来进行设置。

更确切地说，当你指定了 requests.cpu=250m 之后，相当于将 Cgroups 的 **cpu.shares** 的值设置为 (250/1000) * 1024。而当你没有设置 requests.cpu 的时候，cpu.shares 默认则是 1024。

这样，**Kubernetes 就通过 cpu.shares 完成了对 CPU 时间的按比例分配**。

而如果你指定了 limits.cpu=500m 之后，则相当于将 Cgroups 的 **cpu.cfs_quota_us** 的值设置为 (500/1000) * 100ms，而 cpu.cfs_period_us 的值始终是 100ms。

这样，**Kubernetes 就为你设置了这个容器只能用到 CPU 的 50%**。

而对于内存来说，当你指定了 limits.memory=128Mi 之后，相当于将 Cgroups 的 memory.**limit_in_bytes** 设置为 128 * 1024 * 1024。而需要注意的是，在调度的时候，调度器只会使用 **requests.memory**=64Mi 来进行判断。



**Kubernetes 这种对 CPU 和内存资源限额的设计，实际上参考了 Borg 论文中对“动态资源边界”的定义**，既：容器化作业在提交时所设置的资源边界，并不一定是调度系统所必须严格遵守的，这是因为在实际场景中，大多数作业使用到的资源其实远小于它所请求的资源限额。

**基于这种假设**，Borg 在作业被提交后，会**主动减小**它的资源限额配置，以便容纳更多的作业、提升资源利用率。

而当作业资源使用量增加到一定阈值时，Borg 会通过“**快速恢复**”过程，还原作业原始的资源限额，防止出现异常情况。

而 Kubernetes 的 **requests+limits** 的做法，其实就是上述思路的一个简化版：用户在提交 Pod 时，可以声明一个相对较小的 requests 值供调度器使用，而 Kubernetes 真正设置给容器 Cgroups 的，则是相对较大的 limits 值。不难看到，这跟 Borg 的思路相通的。



##### QoS模型

在理解了 Kubernetes 资源模型的设计之后，我再来和你谈谈 Kubernetes 里的 **QoS 模型**。

在 Kubernetes 中，不同的 requests 和 limits 的设置方式，其实会**将这个 Pod 划分到不同的 QoS 级别**当中。

当 Pod 里的每一个 Container 都同时设置了 requests 和 limits，

- 并且 requests 和 limits 值**相等**的时候，这个 Pod 就属于 **Guaranteed** 类别，如下所示：

  ```yaml
  apiVersion: v1
  kind: Pod
  metadata:
    name: qos-demo
    namespace: qos-example
  spec:
    containers:
    - name: qos-demo-ctr
      image: nginx
      resources:
        limits:
          memory: "200Mi"
          cpu: "700m"
        requests:
          memory: "200Mi"
          cpu: "700m"
  ```

  当这个 Pod 创建之后，它的 **qosClass 字段**就会被 Kubernetes 自动设置为 **Guaranteed**。

  需要注意的是，当 Pod 仅设置了 limits 没有设置 requests 的时候，Kubernetes 会自动为它设置与 limits 相同的 requests 值，所以，这也属于 Guaranteed 情况。

- 而当 Pod 不满足 Guaranteed 的条件，但**至少有一个 Container 设置了 requests**。那么这个 Pod 就会被划分到 **Burstable** 类别。比如下面这个例子：

  ```yaml
  apiVersion: v1
  kind: Pod
  metadata:
    name: qos-demo-2
    namespace: qos-example
  spec:
    containers:
    - name: qos-demo-2-ctr
      image: nginx
      resources:
        limits
          memory: "200Mi"
        requests:
          memory: "100Mi"
  ```

- 而如果一个 Pod **既没有设置 requests，也没有设置 limits**，那么它的 QoS 类别就是 **BestEffort**。比如下面这个例子：

  ```yaml
  apiVersion: v1
  kind: Pod
  metadata:
    name: qos-demo-3
    namespace: qos-example
  spec:
    containers:
    - name: qos-demo-3-ctr
      image: nginx
  ```

  

**QoS 划分的主要应用场景**，是<u>当宿主机资源紧张的时候，kubelet 对 Pod 进行 **Eviction（即资源回收）**时需要用到的</u>。

具体地说，当 Kubernetes 所管理的<u>宿主机上不可压缩资源短缺时，就有可能触发 Eviction</u>。比如，可用内存（memory.available）、可用的宿主机磁盘空间（nodefs.available），以及容器运行时镜像存储空间（imagefs.available）等等。目前，Kubernetes 为你设置的 Eviction 的默认阈值如下所示：

```shell
memory.available<100Mi
nodefs.available<10%
nodefs.inodesFree<5%
imagefs.available<15%
```

当然，上述各个触发条件在 kubelet 里都是可配置的。比如下面这个例子：

```shell
kubelet --eviction-hard=imagefs.available<10%,memory.available<500Mi,nodefs.available<5%,nodefs.inodesFree<5% --eviction-soft=imagefs.available<30%,nodefs.available<10% --eviction-soft-grace-period=imagefs.available=2m,nodefs.available=2m --eviction-max-pod-grace-period=600
```

在这个配置中，你可以看到 Eviction 在 Kubernetes 里其实分为 **Soft 和 Hard** 两种模式。

其中，Soft Eviction 允许你为 Eviction 过程设置一段“优雅时间”，比如上面例子里的 imagefs.available=2m，就意味着当 imagefs 不足的阈值达到 2 分钟之后，kubelet 才会开始 Eviction 的过程。

而 Hard Eviction 模式下，Eviction 过程就会在阈值达到之后立刻开始。

> Kubernetes 计算 Eviction 阈值的数据来源，主要依赖于从 Cgroups 读取到的值，以及使用 cAdvisor 监控到的数据。
>

当宿主机的 Eviction 阈值达到后，就会进入 **MemoryPressure** 或者 **DiskPressure** 状态，从而避免新的 Pod 被调度到这台宿主机上。

而当 Eviction 发生的时候，kubelet 具体会挑选哪些 Pod 进行删除操作，就需要参考这些 Pod 的 QoS 类别了。

- 首先，自然是 BestEffort 类别的 Pod。
- 其次，是属于 Burstable 类别、并且发生“饥饿”的资源使用量已经超出了 requests 的 Pod。
- 最后，才是 Guaranteed 类别。并且，Kubernetes 会保证只有当 Guaranteed 类别的 Pod 的资源使用量超过了其 limits 的限制，或者宿主机本身正处于 Memory Pressure 状态时，Guaranteed 的 Pod 才可能被选中进行 Eviction 操作。

当然，对于同 QoS 类别的 Pod 来说，Kubernetes 还会根据 Pod 的优先级来进行进一步地排序和选择。



##### Cpuset

在理解了 Kubernetes 里的 QoS 类别的设计之后，我再来为你讲解一下Kubernetes 里一个非常有用的特性：**cpuset 的设置**。

我们知道，在使用容器的时候，你可以**通过设置 cpuset 把容器绑定到某个 CPU 的核上**，而不是像 cpushare 那样共享 CPU 的计算能力。这种情况下，由于操作系统在 **CPU 之间进行上下文切换的次数大大减少**，容器里应用的性能会得到大幅提升。

事实上，cpuset 方式，是生产环境里部署在线应用类型的 Pod 时，非常常用的一种方式。

可是，这样的需求在 Kubernetes 里又该**如何实现**呢？

- **首先，你的 Pod 必须是 Guaranteed 的 QoS 类型；**
- **然后，你只需要将 Pod 的 CPU 资源的 requests 和 limits 设置为同一个相等的整数值即可。**

例如：

```yaml
spec:
  containers:
  - name: nginx
    image: nginx
    resources:
      limits:
        memory: "200Mi"
        cpu: "2"
      requests:
        memory: "200Mi"
        cpu: "2"
```

这时候，该 Pod 就会被绑定在 2 个独占的 CPU 核上。当然，具体是哪两个 CPU 核，是由 kubelet 为你分配的。

以上，就是 Kubernetes 的**资源模型和 QoS** 类别相关的主要内容。



**总结**

正是基于上述讲述，在实际的使用中，我**强烈建议你将 DaemonSet 的 Pod 都设置为 Guaranteed 的 QoS 类型**。否则，一旦 DaemonSet 的 Pod 被回收，它又会立即在原宿主机上被重建出来，这就使得前面资源回收的动作，完全没有意义了。



### D2: k8s默认调度器

在 Kubernetes 项目中，**默认调度器的主要职责，就是为一个新创建出来的 Pod，寻找一个最合适的节点（Node）**。

> 而这里“**最合适**”的含义，包括两层：
>
> - 从集群所有的节点中，根据调度算法挑选出所有**可以运行**该 Pod 的节点；
> - 从第一步的结果中，再根据调度算法挑选一个**最符合条件**的节点作为最终结果

所以在具体的调度流程中，默认调度器会首先调用一组叫作 **Predicate** 的调度算法，来检查每个 Node。

然后，再调用一组叫作 **Priority** 的调度算法，来给上一步得到的结果里的每个 Node **打分**。最终的调度结果，就是得分最高的那个 Node。

而我在前面的文章中曾经介绍过，**调度器对一个 Pod 调度成功，实际上就是将它的 spec.nodeName 字段填上调度结果的节点名字。**

在 Kubernetes 中，上述调度机制的工作原理，可以用如下所示的一幅示意图来表示。

![image-20220210110909140](/Users/breeze/Library/Application Support/typora-user-images/image-20220210110909140.png)

可以看到，**Kubernetes 的调度器的核心，实际上就是两个相互独立的控制循环。**



**第一个控制循环，我们可以称之为 Informer Path。**

它的主要目的，是启动一系列 Informer，用来监听（Watch）Etcd 中 Pod、Node、Service 等与调度相关的 API 对象的变化。

比如，当一个<u>待调度 Pod（即：它的 nodeName 字段是空的）被创建出来之后，调度器就会通过 Pod Informer 的 Handler，将这个待调度 Pod 添加进调度队列。</u>

在默认情况下，Kubernetes 的调度队列是一个 **PriorityQueue（优先级队列）**，并且当某些集群信息发生变化的时候，调度器还会对调度队列里的内容进行一些特殊操作。这里的设计，主要是出于调度优先级和抢占的考虑，我会在后面的文章中再详细介绍这部分内容。

此外，Kubernetes 的默认调度器**还要负责对调度器缓存（即：scheduler cache）进行更新**。

事实上，Kubernetes 调度部分进行**性能优化**的一个**最根本原则**，就是**尽最大可能将集群信息 Cache 化**，以便从根本上提高 Predicate 和 Priority 调度算法的执行效率。



**第二个控制循环，是调度器负责 Pod 调度的主循环，我们可以称之为 Scheduling Path。**

Scheduling Path 的主要逻辑，就是不断地从调度队列里出队一个 Pod。然后，调用 **Predicates** 算法进行“过滤”。这一步“过滤”得到的一组 Node，就是所有可以运行这个 Pod 的宿主机列表。当然，Predicates 算法需要的 Node 信息，都是从 Scheduler Cache 里直接拿到的，这是调度器保证算法执行效率的主要手段之一。

接下来，调度器就会再调用 **Priorities** 算法为上述列表里的 Node 打分，分数从 0 到 10。得分最高的 Node，就会作为这次调度的结果。调度算法执行完成后，调度器就需要将 Pod 对象的 nodeName 字段的值，修改为上述 Node 的名字。这个步骤在 Kubernetes 里面被称作 **Bind**。

但是，<u>为了不在关键调度路径里远程访问 APIServer</u>，Kubernetes 的默认调度器在 Bind 阶段，只会更新 Scheduler Cache 里的 Pod 和 Node 的信息。这种基于“乐观”假设的 API 对象更新方式，在 Kubernetes 里被称作 **Assume**。

Assume 之后，调度器才会创建一个 **Goroutine** 来异步地向 APIServer 发起更新 Pod 的请求，来真正完成 Bind 操作。如果这次异步的 Bind 过程失败了，其实也没有太大关系，等 Scheduler Cache 同步之后一切就会恢复正常。

当然，正是由于上述 Kubernetes 调度器的“乐观”绑定的设计，当一个新的 Pod 完成调度需要在某个节点上运行起来之前，该节点上的 kubelet 还会通过一个叫作 **Admit** 的操作来再次验证该 Pod 是否确实能够运行在该节点上。

这一步 Admit 操作，实际上就是把一组叫作 **GeneralPredicates** 的、最基本的调度算法，比如：“资源是否可用”“端口是否冲突”等再执行一遍，作为 kubelet 端的**二次确认**。

**除了上述的“Cache 化”和“乐观绑定”，Kubernetes 默认调度器还有一个重要的设计，那就是“无锁化”。**

在 Scheduling Path 上，调度器会启动多个 Goroutine **以节点为粒度并发**执行 Predicates 算法，从而提高这一阶段的执行效率。而与之类似的，Priorities 算法也会以 **MapReduce 的方式并行计算**然后再进行汇总。

而在这些所有需要并发的路径上，**调度器会避免设置任何全局的竞争资源**，从而免去了使用锁进行同步带来的巨大的性能损耗。

所以，在这种思想的指导下，如果你再去查看一下前面的调度器原理图，你就会发现，**Kubernetes 调度器只有对调度队列和 Scheduler Cache 进行操作时，才需要加锁**。而这两部分操作，都不在 Scheduling Path 的算法执行路径上。

当然，Kubernetes 调度器的上述设计思想，也是在集群规模不断增长的演进过程中逐步实现的。尤其是 “Cache 化”，这个变化其实是最近几年 Kubernetes 调度器性能得以提升的一个关键演化。

不过，随着 Kubernetes 项目发展到今天，它的默认调度器也已经来到了一个关键的十字路口。事实上，Kubernetes 现今发展的主旋律，是整个开源项目的“民主化”。也就是说，**Kubernetes 下一步发展的方向，是组件的轻量化、接口化和插件化**。所以，我们才有了 CRI、CNI、CSI、CRD、Aggregated APIServer、Initializer、Device Plugin 等各个层级的可扩展能力。

可是，默认调度器，却成了 Kubernetes 项目里最后一个没有对外暴露出良好定义过的、可扩展接口的组件。

当然，这是有一定的历史原因的。在过去几年，Kubernetes 发展的重点，都是以功能性需求的实现和完善为核心。在这个过程中，它的很多决策，还是以优先服务公有云的需求为主，而性能和规模则居于相对次要的位置。

**而现在，随着 Kubernetes 项目逐步趋于稳定，越来越多的用户开始把 Kubernetes 用在规模更大、业务更加复杂的私有集群当中。很多以前的 Mesos 用户，也开始尝试使用 Kubernetes 来替代其原有架构。在这些场景下，对默认调度器进行扩展和重新实现，就成了社区对 Kubernetes 项目最主要的一个诉求。**

所以，Kubernetes 的默认调度器，是目前这个项目里为数不多的、正在经历大量重构的核心组件之一。这些正在进行的重构的目的，一方面是将默认调度器里大量的“技术债”清理干净；另一方面，就是为默认调度器的**可扩展性**设计进行铺垫。

而 Kubernetes **默认调度器的可扩展性设计**，可以用如下所示的一幅示意图来描述：

![image-20220210141305833](/Users/breeze/Library/Application Support/typora-user-images/image-20220210141305833.png)

可以看到，默认调度器的可扩展机制，在 Kubernetes 里面叫作 **Scheduler Framework**。

顾名思义，**这个设计的主要目的，就是在调度器生命周期的各个关键点上，为用户暴露出可以进行扩展和实现的接口，从而实现由用户自定义调度器的能力**。

上图中，每一个绿色的箭头都是一个可以插入自定义逻辑的接口。

比如，上面的 Queue 部分，就意味着你可以在这一部分提供一个自己的调度队列的实现，从而控制每个 Pod 开始被调度（出队）的时机。而 Predicates 部分，则意味着你可以提供自己的过滤算法实现，根据自己的需求，来决定选择哪些机器。

**需要注意的是，上述这些可插拔式逻辑，都是标准的 Go 语言插件机制（Go plugin 机制）**，也就是说，你需要在编译的时候选择把哪些插件编译进去。

有了上述设计之后，扩展和自定义 Kubernetes 的默认调度器就变成了一件非常容易实现的事情。这也意味着默认调度器在后面的发展过程中，必然不会在现在的实现上再添加太多的功能，反而还会对现在的实现进行精简，最终成为 Scheduler Framework 的一个最小实现。

而调度领域更多的创新和工程工作，就可以交给整个社区来完成了。这个思路，是完全符合我在前面提到的 Kubernetes 的“民主化”设计的。

不过，这样的 Scheduler Framework 也有一个**不小的问题**，那就是一旦这些<u>插入点的接口设计不合理，就会导致整个生态没办法很好地把这个插件机制使用起来</u>。而与此同时，这些<u>接口本身的变更又是一个费时费力</u>的过程，一旦把控不好，就很可能会把社区推向另一个极端，即：Scheduler Framework 没法实际落地，大家只好都再次 fork kube-scheduler。



### D3: k8s默认调度器调度策略



##### Predicates

Predicates 在调度过程中的作用，可以理解为 **Filter**，即：它按照调度策略，从当前集群的所有节点中，“过滤”出一系列符合条件的节点。这些节点，都是可以运行待调度 Pod 的宿主机。

而在 Kubernetes 中，默认的**调度策略有如下三种。**



**第一种类型，叫作 GeneralPredicates。**

顾名思义，这一组过滤规则，**负责的是最基础的调度策略**。

比如，PodFitsResources 计算的就是宿主机的 CPU 和内存资源等是否够用。当然，我在前面已经提到过，PodFitsResources 检查的只是 Pod 的 requests 字段。

需要注意的是，Kubernetes 的调度器并没有为 GPU 等硬件资源定义具体的资源类型，而是统一用一种名叫 Extended Resource 的、Key-Value 格式的扩展字段来描述的。比如下面这个例子：

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: extended-resource-demo
spec:
  containers:
  - name: extended-resource-demo-ctr
    image: nginx
    resources:
      requests:
        alpha.kubernetes.io/nvidia-gpu: 2
      limits:
        alpha.kubernetes.io/nvidia-gpu: 2
```

可以看到，我们这个 Pod 通过alpha.kubernetes.io/nvidia-gpu=2这样的定义方式，声明使用了两个 NVIDIA 类型的 GPU。

而在 **PodFitsResources** 里面，调度器其实并不知道这个字段 Key 的含义是 GPU，而是直接使用后面的 Value 进行计算。当然，在 Node 的 Capacity 字段里，你也得相应地加上这台宿主机上 GPU 的总数，比如：alpha.kubernetes.io/nvidia-gpu=4。这些流程，我在后面讲解 Device Plugin 的时候会详细介绍。

而 **PodFitsHost** 检查的是，宿主机的名字是否跟 Pod 的 spec.nodeName 一致。

**PodFitsHostPorts** 检查的是，Pod 申请的宿主机端口（spec.nodePort）是不是跟已经被使用的端口有冲突。

**PodMatchNodeSelector** 检查的是，Pod 的 nodeSelector 或者 nodeAffinity 指定的节点，是否与待考察节点匹配，等等。

可以看到，像上面这样一组 GeneralPredicates，正是 Kubernetes 考察一个 Pod 能不能运行在一个 Node 上最基本的过滤条件。所以，GeneralPredicates 也会被其他组件（比如 kubelet）直接调用。我在上一篇文章中已经提到过，kubelet 在启动 Pod 前，会执行一个 Admit 操作来进行二次确认。这里二次确认的规则，就是执行一遍 GeneralPredicates。



**第二种类型，是与 Volume 相关的过滤规则。**

这一组过滤规则，负责的是**跟容器持久化 Volume 相关的调度策略**。

其中，**NoDiskConflict** 检查的条件，是多个 Pod 声明挂载的持久化 Volume 是否有冲突。比如，AWS EBS 类型的 Volume，是不允许被两个 Pod 同时使用的。所以，当一个名叫 A 的 EBS Volume 已经被挂载在了某个节点上时，另一个同样声明使用这个 A Volume 的 Pod，就不能被调度到这个节点上了。

而 **MaxPDVolumeCountPredicate** 检查的条件，则是一个节点上某种类型的持久化 Volume 是不是已经超过了一定数目，如果是的话，那么声明使用该类型持久化 Volume 的 Pod 就不能再调度到这个节点了。

而 **VolumeZonePredicate**，则是检查持久化 Volume 的 Zone（高可用域）标签，是否与待考察节点的 Zone 标签相匹配。

此外，这里还有一个叫作 **VolumeBindingPredicate** 的规则。它负责检查的，是该 Pod 对应的 PV 的 nodeAffinity 字段，是否跟某个节点的标签相匹配。

在前面的第 29 篇文章《PV、PVC 体系是不是多此一举？从本地持久化卷谈起》中，我曾经为你讲解过，**Local Persistent Volume**（本地持久化卷），必须使用 nodeAffinity 来跟某个具体的节点绑定。这其实也就意味着，在 Predicates 阶段，Kubernetes 就必须能够根据 Pod 的 Volume 属性来进行调度。此外，如果该 Pod 的 PVC 还没有跟具体的 PV 绑定的话，调度器还要负责检查所有待绑定 PV，当有可用的 PV 存在并且该 PV 的 nodeAffinity 与待考察节点一致时，这条规则才会返回“成功”。比如下面这个例子：

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: example-local-pv
spec:
  capacity:
    storage: 500Gi
  accessModes:
  - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: local-storage
  local:
    path: /mnt/disks/vol1
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - key: kubernetes.io/hostname
          operator: In
          values:
          - my-node
```

可以看到，这个 PV 对应的持久化目录，只会出现在名叫 my-node 的宿主机上。所以，任何一个通过 PVC 使用这个 PV 的 Pod，都必须被调度到 my-node 上才可以正常工作。VolumeBindingPredicate，正是调度器里完成这个决策的位置。



**第三种类型，是宿主机相关的过滤规则。**

这一组规则，主要**考察待调度 Pod 是否满足 Node 本身的某些条件**。比如，PodToleratesNodeTaints，负责检查的就是我们前面经常用到的 Node 的“污点”机制。只有当 Pod 的 Toleration 字段与 Node 的 Taint 字段能够匹配的时候，这个 Pod 才能被调度到该节点上。

而 NodeMemoryPressurePredicate，检查的是当前节点的内存是不是已经不够充足，如果是的话，那么待调度 Pod 就不能被调度到该节点上。



**第四种类型，是 Pod 相关的过滤规则。**

这一组规则，跟 GeneralPredicates 大多数是重合的。而比较特殊的，是 PodAffinityPredicate。这个规则的作用，是检查待调度 Pod 与 Node 上的已有 Pod 之间的亲密（affinity）和反亲密（anti-affinity）关系。比如下面这个例子：

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: with-pod-antiaffinity
spec:
  affinity:
    podAntiAffinity: 
      requiredDuringSchedulingIgnoredDuringExecution: 
      - weight: 100  
        podAffinityTerm:
          labelSelector:
            matchExpressions:
            - key: security 
              operator: In 
              values:
              - S2
          topologyKey: kubernetes.io/hostname
  containers:
  - name: with-pod-affinity
    image: docker.io/ocpqe/hello-pod
```

这个例子里的 podAntiAffinity 规则，就指定了这个 Pod 不希望跟任何携带了 security=S2 标签的 Pod 存在于同一个 Node 上。需要注意的是，PodAffinityPredicate 是有作用域的，比如上面这条规则，就仅对携带了 Key 是kubernetes.io/hostname标签的 Node 有效。这正是 topologyKey 这个关键词的作用。而与 podAntiAffinity 相反的，就是 podAffinity，比如下面这个例子：

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: with-pod-affinity
spec:
  affinity:
    podAffinity: 
      requiredDuringSchedulingIgnoredDuringExecution: 
      - labelSelector:
          matchExpressions:
          - key: security 
            operator: In 
            values:
            - S1 
        topologyKey: failure-domain.beta.kubernetes.io/zone
  containers:
  - name: with-pod-affinity
    image: docker.io/ocpqe/hello-pod
```

这个例子里的 Pod，就只会被调度到已经有携带了 security=S1 标签的 Pod 运行的 Node 上。而这条规则的作用域，则是所有携带 Key 是failure-domain.beta.kubernetes.io/zone标签的 Node。

此外，上面这两个例子里的 **requiredDuringSchedulingIgnoredDuringExecution** 字段的含义是：这条规则必须在 Pod 调度时进行检查（requiredDuringScheduling）；但是如果是已经在运行的 Pod 发生变化，比如 Label 被修改，造成了该 Pod 不再适合运行在这个 Node 上的时候，Kubernetes 不会进行主动修正（IgnoredDuringExecution）。

上面这四种类型的 Predicates，就构成了调度器确定一个 Node 可以运行待调度 Pod 的基本策略。

**在具体执行的时候， 当开始调度一个 Pod 时，Kubernetes 调度器会同时启动 16 个 Goroutine，来并发地为集群里的所有 Node 计算 Predicates，最后返回可以运行这个 Pod 的宿主机列表。**

需要注意的是，在为每个 Node 执行 Predicates 时，调度器会按照固定的顺序来进行检查。这个顺序，是按照 Predicates 本身的含义来确定的。比如，宿主机相关的 Predicates 会被放在相对靠前的位置进行检查。要不然的话，在一台资源已经严重不足的宿主机上，上来就开始计算 PodAffinityPredicate，是没有实际意义的。





##### Priorities

在 Predicates 阶段完成了节点的“过滤”之后，Priorities 阶段的工作就是**为这些节点打分**。这里打分的范围是 0-10 分，得分最高的节点就是最后被 Pod 绑定的最佳节点。

Priorities 里最常用到的一个打分规则，是 **LeastRequestedPriority**。它的计算方法，可以简单地总结为如下所示的公式：

```apl
score = (cpu((capacity-sum(requested))10/capacity) + memory((capacity-sum(requested))10/capacity))/2
```

可以看到，这个算法实际上就是在**选择空闲资源（CPU 和 Memory）最多的宿主机**。而与 LeastRequestedPriority 一起发挥作用的，还有 BalancedResourceAllocation。它的计算公式如下所示：

```apl
score = 10 - variance(cpuFraction,memoryFraction,volumeFraction)*10
```

其中，每种资源的 Fraction 的定义是 ：Pod 请求的资源 / 节点上的可用资源。

而 **variance** 算法的作用，则是计算每两种资源 Fraction 之间的“距离”。而最后选择的，则是资源 Fraction 差距最小的节点。所以说，BalancedResourceAllocation 选择的，其实是调度完成后，所有节点里**各种**资源分配最均衡的那个节点，从而避免一个节点上 CPU 被大量分配、而 Memory 大量剩余的情况。

此外，还有 NodeAffinityPriority、TaintTolerationPriority 和 InterPodAffinityPriority 这三种 Priority。顾名思义，它们与前面的 PodMatchNodeSelector、PodToleratesNodeTaints 和 PodAffinityPredicate 这三个 Predicate 的含义和计算方法是类似的。但是作为 Priority，一个 Node 满足上述规则的字段数目越多，它的得分就会越高。

在默认 Priorities 里，还有一个叫作 ImageLocalityPriority 的策略。它是在 Kubernetes v1.12 里新开启的调度规则，即：如果待调度 Pod 需要使用的镜像很大，并且已经存在于某些 Node 上，那么这些 Node 的得分就会比较高。当然，为了避免这个算法引发调度堆叠，调度器在计算得分的时候还会根据镜像的分布进行优化，即：如果大镜像分布的节点数目很少，那么这些节点的权重就会被调低，从而“对冲”掉引起调度堆叠的风险。

以上，就是 Kubernetes 调度器的 Predicates 和 Priorities 里默认调度规则的主要工作原理了。

**在实际的执行过程中，调度器里关于集群和 Pod 的信息都已经缓存化，所以这些算法的执行过程还是比较快的。**

此外，对于比较复杂的调度算法来说，比如 PodAffinityPredicate，它们在计算的时候不只关注待调度 Pod 和待考察 Node，还需要关注整个集群的信息，比如，遍历所有节点，读取它们的 Labels。这时候，Kubernetes 调度器会在为每个待调度 Pod 执行该调度算法之前，先将算法需要的集群信息初步计算一遍，然后缓存起来。这样，在真正执行该算法的时候，调度器只需要读取缓存信息进行计算即可，从而避免了为每个 Node 计算 Predicates 的时候反复获取和计算整个集群的信息。



需要注意的是，除了本篇讲述的这些规则，Kubernetes 调度器里其实还有一些默认不会开启的策略。你可以通过为 kube-scheduler 指定一个配置文件或者创建一个 ConfigMap ，来配置哪些规则需要开启、哪些规则需要关闭。并且，你可以通过为 Priorities 设置权重，来控制调度器的调度行为。



### D4: k8s默认调度器的优先级与抢占

**首先需要明确的是，优先级和抢占机制，解决的是 Pod 调度失败时该怎么办的问题**。

正常情况下，当一个 Pod 调度失败后，它就会被暂时“搁置”起来，直到 Pod 被更新，或者集群状态发生变化，调度器才会对这个 Pod 进行重新调度。

但在有时候，我们希望的是这样一个场景。当一个高优先级的 Pod 调度失败后，该 Pod 并不会被“搁置”，而是会“挤走”某个 Node 上的一些低优先级的 Pod 。这样就可以保证这个高优先级 Pod 的调度成功。

这个特性，其实也是一直以来就存在于 Borg 以及 Mesos 等项目里的一个基本功能。而在 Kubernetes 里，优先级和抢占机制是在 1.10 版本后才逐步可用的。

要使用这个机制，你首先需要在 Kubernetes 里提交一个 **PriorityClass** 的定义，如下所示：

```yaml
apiVersion: scheduling.k8s.io/v1beta1
kind: PriorityClass
metadata:
  name: high-priority
value: 1000000
globalDefault: false
description: "This priority class should be used for high priority service pods only."
```

上面这个 YAML 文件，定义的是一个名叫 high-priority 的 PriorityClass，其中 value 的值是 1000000 （一百万）。

**Kubernetes 规定，优先级是一个 32 bit 的整数，最大值不超过 1000000000（10 亿，1 billion），并且值越大代表优先级越高。**而超出 10 亿的值，其实是被 Kubernetes 保留下来分配给系统 Pod 使用的。显然，这样做的目的，就是保证系统 Pod 不会被用户抢占掉。

而一旦上述 YAML 文件里的 globalDefault 被设置为 true 的话，那就意味着这个 PriorityClass 的值会成为系统的默认值。而如果这个值是 false，就表示我们只希望声明使用该 PriorityClass 的 Pod 拥有值为 1000000 的优先级，而对于没有声明 PriorityClass 的 Pod 来说，它们的优先级就是 0。

在创建了 PriorityClass 对象之后，Pod 就可以声明使用它了，如下所示：

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    env: test
spec:
  containers:
  - name: nginx
    image: nginx
    imagePullPolicy: IfNotPresent
  priorityClassName: high-priority
```

可以看到，这个 Pod 通过 priorityClassName 字段，声明了要使用名叫 high-priority 的 PriorityClass。当这个 Pod 被提交给 Kubernetes 之后，Kubernetes 的 PriorityAdmissionController 就会自动将这个 Pod 的 spec.priority 字段设置为 1000000。

而我在前面的文章中曾为你介绍过，调度器里维护着一个调度队列。所以，当 Pod 拥有了优先级之后，高优先级的 Pod 就可能会比低优先级的 Pod 提前出队，从而尽早完成调度过程。**这个过程，就是“优先级”这个概念在 Kubernetes 里的主要体现。**



为了方便叙述，我接下来会把待调度的高优先级 Pod 称为“抢占者”（Preemptor）。

当上述抢占过程发生时，**抢占者并不会立刻被调度到被抢占的 Node 上**。事实上，调度器只会将抢占者的 **spec.nominatedNodeName** 字段，设置为被抢占的 Node 的名字。然后，抢占者会重新进入下一个调度周期，然后在新的调度周期里来决定是不是要运行在被抢占的节点上。这当然也就意味着，即使在下一个调度周期，调度器也不会保证抢占者一定会运行在被抢占的节点上。

这样设计的一个重要原因是，调度器只会通过标准的 DELETE API 来删除被抢占的 Pod，所以，这些 Pod 必然是有一定的“优雅退出”时间（默认是 30s）的。而在这段时间里，其他的节点也是有可能变成可调度的，或者直接有新的节点被添加到这个集群中来。

所以，鉴于优雅退出期间，集群的可调度性可能会发生的变化，把抢占者交给下一个调度周期再处理，是一个非常合理的选择。

而在抢占者等待被调度的过程中，如果有其他更高优先级的 Pod 也要抢占同一个节点，那么调度器就会清空原抢占者的 spec.nominatedNodeName 字段，从而允许更高优先级的抢占者执行抢占，并且，这也就使得原抢占者本身，也有机会去重新抢占其他节点。

这些，都是设置 **nominatedNodeName** 字段的主要目的。



**那么，Kubernetes 调度器里的抢占机制，又是如何设计的呢？**

我在前面已经提到过，抢占发生的原因，一定是一个高优先级的 Pod 调度失败。这一次，我们还是称这个 Pod 为“抢占者”，称被抢占的 Pod 为“牺牲者”（victims）。

**而 Kubernetes 调度器实现抢占算法的一个最重要的设计，就是在调度队列的实现里，使用了两个不同的队列。**

**第一个队列，叫作 activeQ**。凡是在 activeQ 里的 Pod，都是下一个调度周期需要调度的对象。所以，当你在 Kubernetes 集群里新创建一个 Pod 的时候，调度器会将这个 Pod 入队到 activeQ 里面。而我在前面提到过的、调度器不断从队列里出队（Pop）一个 Pod 进行调度，实际上都是从 activeQ 里出队的。

**第二个队列，叫作 unschedulableQ**，专门用来存放调度失败的 Pod。而这里的一个关键点就在于，当一个 unschedulableQ 里的 Pod 被更新之后，调度器会自动把这个 Pod 移动到 activeQ 里，从而给这些调度失败的 Pod “重新做人”的机会。

现在，回到我们的抢占者调度失败这个时间点上来。

调度失败之后，抢占者就会被放进 unschedulableQ 里面。



然后，这次失败事件就会触发**调度器为抢占者寻找牺牲者的流程**。

**第一步**，调度器会检查这次失败事件的原因，来确认抢占是不是可以帮助抢占者找到一个新节点。这是因为有很多 Predicates 的失败是不能通过抢占来解决的。比如，PodFitsHost 算法（负责的是，检查 Pod 的 nodeSelector 与 Node 的名字是否匹配），这种情况下，除非 Node 的名字发生变化，否则你即使删除再多的 Pod，抢占者也不可能调度成功。

**第二步**，如果确定抢占可以发生，那么调度器就会**把自己缓存的所有节点信息复制一份**，然后**使用这个副本来模拟抢占过程**。

这里的抢占过程很容易理解。调度器会检查缓存副本里的每一个节点，然后从该节点上最低优先级的 Pod 开始，逐一“删除”这些 Pod。而每删除一个低优先级 Pod，调度器都会检查一下抢占者是否能够运行在该 Node 上。一旦可以运行，调度器就记录下这个 Node 的名字和被删除 Pod 的列表，这就是一次抢占过程的结果了。

当遍历完所有的节点之后，调度器会在上述模拟产生的所有**抢占结果里做一个选择，找出最佳结果**。**而这一步的判断原则，就是尽量减少抢占对整个系统的影响**。比如，需要抢占的 Pod 越少越好，需要抢占的 Pod 的优先级越低越好，等等。



在**得到了最佳的抢占结果之后**，这个结果里的 Node，就是即将被抢占的 Node；被删除的 Pod 列表，就是牺牲者。所以接下来，调度器就可以**真正开始抢占的操作**了，这个过程，可以分为三步。

**第一步**，调度器会检查牺牲者列表，清理这些 Pod 所携带的 nominatedNodeName 字段。

**第二步**，调度器会把抢占者的 nominatedNodeName，设置为被抢占的 Node 的名字。

**第三步**，调度器会开启一个 Goroutine，同步地删除牺牲者。

而第二步对抢占者 Pod 的更新操作，就会触发到我前面提到的“重新做人”的流程，从而让抢占者在下一个调度周期重新进入调度流程。



所以接下来，**调度器就会通过正常的调度流程把抢占者调度成功**。这也是为什么，我前面会说调度器并不保证抢占的结果：在这个正常的调度流程里，是一切皆有可能的。

不过，对于任意一个待调度 Pod 来说，因为有上述抢占者的存在，它的调度过程，其实是有一些特殊情况需要特殊处理的。

具体来说，在为某一对 Pod 和 Node 执行 Predicates 算法的时候，如果待检查的 Node 是一个即将被抢占的节点，即：调度队列里有 nominatedNodeName 字段值是该 Node 名字的 Pod 存在（可以称之为：“潜在的抢占者”）。那么，**调度器就会对这个 Node ，将同样的 Predicates 算法运行两遍。**

**第一遍**， 调度器会假设上述“潜在的抢占者”已经运行在这个节点上，然后执行 Predicates 算法；

**第二遍**， 调度器会正常执行 Predicates 算法，即：不考虑任何“潜在的抢占者”。

而只有这两遍 Predicates 算法都能通过时，这个 Pod 和 Node 才会被认为是可以绑定（bind）的。

不难想到，这里需要**执行第一遍 Predicates 算法的原因**，是由于 InterPodAntiAffinity 规则的存在。

由于 InterPodAntiAffinity 规则关心待考察节点上所有 Pod 之间的互斥关系，所以我们在执行调度算法时必须考虑，如果抢占者已经存在于待考察 Node 上时，待调度 Pod 还能不能调度成功。

当然，这也就意味着，我们在这一步只需要考虑那些优先级等于或者大于待调度 Pod 的抢占者。毕竟对于其他较低优先级 Pod 来说，待调度 Pod 总是可以通过抢占运行在待考察 Node 上。

而我们需要**执行第二遍 Predicates 算法的原因**，则是因为“潜在的抢占者”最后不一定会运行在待考察的 Node 上。关于这一点，我在前面已经讲解过了：Kubernetes 调度器并不保证抢占者一定会运行在当初选定的被抢占的 Node 上。

以上，就是 Kubernetes 默认调度器里优先级和抢占机制的实现原理了。





### D5: k8s GPU管理与Device Plugin机制

从 2016 年开始，Kubernetes 社区就不断收到来自不同渠道的大量诉求，希望能够在 Kubernetes 集群上运行 TensorFlow 等机器学习框架所创建的训练（Training）和服务（Serving）任务。而这些诉求中，除了前面我为你讲解过的 Job、Operator 等离线作业管理需要用到的编排概念之外，还有一个亟待实现的功能，就是**对 GPU 等硬件加速设备管理的支持**。

对于云的用户来说，在 GPU 的支持上，他们**最基本的诉求**其实非常简单：我只要在 Pod 的 YAML 里面，声明某容器需要的 GPU 个数，那么 Kubernetes 为我创建的容器里就应该出现对应的 GPU 设备，以及它对应的驱动目录。

以 NVIDIA 的 GPU 设备为例，上面的需求就意味着当用户的容器被创建之后，这个容器里必须出现如下两部分设备和目录：

- GPU 设备，比如 /dev/nvidia0；
- GPU 驱动目录，比如 /usr/local/nvidia/*。

其中，GPU 设备路径，正是该容器启动时的 Devices 参数；而驱动目录，则是该容器启动时的 Volume 参数。

所以，在 Kubernetes 的 GPU 支持的实现里，kubelet 实际上就是将上述两部分内容，设置在了创建该容器的 CRI （Container Runtime Interface）参数里面。这样，等到该容器启动之后，对应的容器里就会出现 GPU 设备和驱动的路径了。

不过，Kubernetes 在 Pod 的 API 对象里，并没有为 GPU 专门设置一个资源类型字段，而是使用了一种叫作 **Extended Resource**（ER）的特殊字段来负责传递 GPU 的信息。比如下面这个例子：

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: cuda-vector-add
spec:
  restartPolicy: OnFailure
  containers:
    - name: cuda-vector-add
      image: "k8s.gcr.io/cuda-vector-add:v0.1"
      resources:
        limits:
          nvidia.com/gpu: 1
          
# 可以看到，在上述 Pod 的 limits 字段里，这个资源的名称是nvidia.com/gpu，它的值是 1。
# 也就是说，这个 Pod 声明了自己要使用一个 NVIDIA 类型的 GPU。
```

而在 kube-scheduler 里面，它其实并不关心这个字段的具体含义，只会在计算的时候，一律将调度器里保存的该类型资源的可用量，直接减去 Pod 声明的数值即可。所以说，Extended Resource，其实是 Kubernetes 为用户设置的一种对自定义资源的支持。

当然，为了能够让调度器知道这个自定义类型的资源在每台宿主机上的可用量，宿主机节点本身，就必须能够向 API Server 汇报该类型资源的可用数量。在 Kubernetes 里，各种类型的资源可用量，其实是 Node 对象 Status 字段的内容，比如下面这个例子：

```yaml
apiVersion: v1
kind: Node
metadata:
  name: node-1
...
Status:
  Capacity:
   cpu:  2
   memory:  2049008Ki
```

而为了能够在上述 Status 字段里添加自定义资源的数据，你就必须使用 PATCH API 来对该 Node 对象进行更新，加上你的自定义资源的数量。这个 PATCH 操作，可以简单地使用 curl 命令来发起，如下所示：

```shell
# 启动 Kubernetes 的客户端 proxy，这样你就可以直接使用 curl 来跟 Kubernetes  的API Server 进行交互了
$ kubectl proxy

# 执行 PACTH 操作
$ curl --header "Content-Type: application/json-patch+json" \
--request PATCH \
--data '[{"op": "add", "path": "/status/capacity/nvidia.com/gpu", "value": "1"}]' \
http://localhost:8001/api/v1/nodes/<your-node-name>/status
```

PATCH 操作完成后，你就可以看到 Node 的 Status 变成了如下所示的内容：

```yaml
apiVersion: v1
kind: Node
...
Status:
  Capacity:
   cpu:  2
   memory:  2049008Ki
   nvidia.com/gpu: 1
```

这样在调度器里，它就能够在缓存里记录下 node-1 上的nvidia.com/gpu类型的资源的数量是 1。

当然，在 Kubernetes 的 GPU 支持方案里，你并不需要真正去做上述关于 Extended Resource 的这些操作。**在 Kubernetes 中，对所有硬件加速设备进行管理的功能，都是由一种叫作 Device Plugin 的插件来负责的。**这其中，当然也就包括了对该硬件的 Extended Resource 进行汇报的逻辑。

Kubernetes 的 **Device Plugin** 机制，我可以用如下所示的一幅示意图来和你解释清楚。

![image-20220212151009594](/Users/breeze/Library/Application Support/typora-user-images/image-20220212151009594.png)

我们先从这幅示意图的右侧开始看起。

首先，对于每一种硬件设备，都需要有它所对应的 Device Plugin 进行管理，这些 Device Plugin，都通过 gRPC 的方式，同 kubelet 连接起来。以 NVIDIA GPU 为例，它对应的插件叫作NVIDIA GPU device plugin。

这个 Device Plugin 会通过一个叫作 **ListAndWatch** 的 API，定期向 kubelet 汇报该 Node 上 GPU 的列表。

比如，在我们的例子里，一共有三个 GPU（GPU0、GPU1 和 GPU2）。这样，kubelet 在拿到这个列表之后，就可以直接在它向 APIServer 发送的心跳里，以 Extended Resource 的方式，加上这些 GPU 的数量，比如nvidia.com/gpu=3。所以说，用户在这里是不需要关心 GPU 信息向上的汇报流程的。需要注意的是，

ListAndWatch 向上汇报的信息，<u>只有本机上 GPU 的 ID 列表，而不会有任何关于 GPU 设备本身的信息</u>。而且 kubelet 在向 API Server 汇报的时候，只会汇报该 GPU 对应的 Extended Resource 的数量。当然，kubelet 本身，会将这个 GPU 的 ID 列表保存在自己的内存里，并通过 ListAndWatch API 定时更新。

而当一个 Pod 想要使用一个 GPU 的时候，它只需要像我在本文一开始给出的例子一样，在 Pod 的 limits 字段声明nvidia.com/gpu: 1。那么接下来，Kubernetes 的调度器就会从它的缓存里，寻找 GPU 数量满足条件的 Node，然后将缓存里的 GPU 数量减 1，完成 Pod 与 Node 的绑定。

这个调度成功后的 Pod 信息，自然就会被对应的 kubelet 拿来进行容器操作。而当 kubelet 发现这个 Pod 的容器请求一个 GPU 的时候，kubelet 就会从自己持有的 GPU 列表里，为这个容器分配一个 GPU。此时，kubelet 就会向本机的 Device Plugin 发起一个 **Allocate()** 请求。这个请求携带的参数，正是即将分配给该容器的设备 ID 列表。

当 Device Plugin 收到 Allocate 请求之后，它就会根据 kubelet 传递过来的设备 ID，从 Device Plugin 里找到这些设备对应的设备路径和驱动目录。当然，这些信息，正是 Device Plugin 周期性的从本机查询到的。比如，在 NVIDIA Device Plugin 的实现里，它会定期访问 nvidia-docker 插件，从而获取到本机的 GPU 信息。

而被分配 GPU 对应的设备路径和驱动目录信息被返回给 kubelet 之后，kubelet 就完成了为一个容器分配 GPU 的操作。接下来，kubelet 会把这些信息追加在创建该容器所对应的 CRI 请求当中。这样，当这个 CRI 请求发给 Docker 之后，Docker 为你创建出来的容器里，就会出现这个 GPU 设备，并把它所需要的驱动目录挂载进去。

至此，**Kubernetes 为一个 Pod 分配一个 GPU 的流程就完成了。**



对于其他类型硬件来说，要想在 Kubernetes 所管理的容器里使用这些硬件的话，也需要遵循上述 Device Plugin 的流程来实现如下所示的 Allocate 和 ListAndWatch API：

```shell
  service DevicePlugin {
        // ListAndWatch returns a stream of List of Devices
        // Whenever a Device state change or a Device disappears, ListAndWatch
        // returns the new list
        rpc ListAndWatch(Empty) returns (stream ListAndWatchResponse) {}
        // Allocate is called during container creation so that the Device
        // Plugin can run device specific operations and instruct Kubelet
        // of the steps to make the Device available in the container
        rpc Allocate(AllocateRequest) returns (AllocateResponse) {}
  }
```

目前，Kubernetes 社区里已经实现了很多硬件插件，比如FPGA、SRIOV、RDMA等等。



**总结**

在本篇文章中，详细讲述了 Kubernetes 对 GPU 的管理方式，以及它所需要使用的 Device Plugin 机制。

需要指出的是，Device Plugin 的设计，长期以来都是以 Google Cloud 的用户需求为主导的，所以，它的整套工作机制和流程上，实际上跟学术界和工业界的真实场景还有着不小的差异。

这里最大的问题在于，GPU 等硬件设备的调度工作，实际上是由 kubelet 完成的。即，kubelet 会负责从它所持有的硬件设备列表中，为容器挑选一个硬件设备，然后调用 Device Plugin 的 Allocate API 来完成这个分配操作。可以看到，在整条链路中，调度器扮演的角色，仅仅是为 Pod 寻找到可用的、支持这种硬件设备的节点而已。

这就使得，**Kubernetes 里对硬件设备的管理，只能处理“设备个数”这唯一一种情况。**一旦你的设备是异构的、不能简单地用“数目”去描述具体使用需求的时候，比如，“<u>我的 Pod 想要运行在计算能力最强的那个 GPU 上”，Device Plugin 就完全不能处理了</u>。

更不用说，在很多场景下，我们其实希望在调度器进行调度的时候，就可以根据整个集群里的某种硬件设备的全局分布，做出一个最佳的调度选择。

此外，上述 Device Plugin 的设计，也使得 Kubernetes 里，缺乏一种能够对 Device 进行描述的 API 对象。这就使得如果你的硬件设备本身的属性比较复杂，并且 Pod 也关心这些硬件的属性的话，那么 Device Plugin 也是完全没有办法支持的。

更为棘手的是，在 Device Plugin 的设计和实现中，Google 的工程师们一直不太愿意为 Allocate 和 ListAndWatch API 添加可扩展性的参数。这就使得，当你确实需要处理一些比较复杂的硬件设备使用需求时，是没有办法通过扩展 Device Plugin 的 API 来实现的。

针对这些问题，RedHat 在社区里曾经大力推进过 ResourceClass的设计，试图将硬件设备的管理功能上浮到 API 层和调度层。但是，由于各方势力的反对，这个提议最后不了了之了。

所以说，**目前 Kubernetes 本身的 Device Plugin 的设计，实际上能覆盖的场景是非常单一的，属于“可用”但是“不好用”的状态。**并且， Device Plugin 的 API 本身的可扩展性也不是很好。这也就解释了为什么像 NVIDIA 这样的硬件厂商，实际上并没有完全基于上游的 Kubernetes 代码来实现自己的 GPU 解决方案，而是做了一定的改动，也就是 fork。这，实属不得已而为之。











## P6: k8s容器运行时

在前面的文章中，讲解了关于 Kubernetes 调度和资源管理相关的内容。实际上，在调度这一步完成后，**Kubernetes 就需要负责将这个调度成功的 Pod，在宿主机上创建出来，并把它所定义的各个容器启动起来**。这些，都是 **kubelet 这个核心组件的主要功能**。

在接下来三篇文章中，将深入到 kubelet 里面，详细剖析一 **Kubernetes 对容器运行时的管理能力。**

### D1: 幕后英雄 SIG-Node 与 CRI

在 Kubernetes 社区里，与 kubelet 以及容器运行时管理相关的内容，都属于 SIG-Node 的范畴。

SIG-Node 以及 kubelet，其实是 Kubernetes 整套体系里非常核心的一个部分。 毕竟，它们才是 Kubernetes 这样一个容器编排与管理系统，跟容器打交道的主要“场所”。

而 **kubelet** 这个组件本身，也是 Kubernetes 里面第二个**不可被替代**的组件（第一个不可被替代的组件当然是 kube-apiserver）。也就是说，**无论如何，我都不太建议你对 kubelet 的代码进行大量的改动。保持 kubelet 跟上游基本一致的重要性，就跟保持 kube-apiserver 跟上游一致是一个道理。**

当然， kubelet 本身，也是按照“控制器”模式来工作的。它实际的工作原理，可以用如下所示的一幅示意图来表示清楚。

<img src="/Users/breeze/Library/Application Support/typora-user-images/image-20220209100446924.png" alt="image-20220209100446924" style="zoom: 25%;" />

可以看到，kubelet 的工作核心，就是一个控制循环，即：SyncLoop（图中的大圆圈）。

而**驱动这个控制循环运行的事件**，包括四种：

- **Pod 更新事件；**
- **Pod 生命周期变化；**
- **kubelet 本身设置的执行周期；**
- **定时的清理事件。**

所以，跟其他控制器类似，kubelet 启动的时候，**要做的第一件事情，就是设置 Listers，也就是注册它所关心的各种事件的 Informer**。这些 Informer，就是 SyncLoop 需要处理的数据的来源。

此外，kubelet 还**负责维护着很多很多其他的子控制循环**（也就是图中的小圆圈）。这些控制循环的名字，一般被称作某某 Manager，比如 Volume Manager、Image Manager、Node Status Manager 等等。

不难想到，<u>这些控制循环的责任，就是通过控制器模式，完成 kubelet 的某项具体职责</u>。

- 比如 Node Status Manager，就负责响应 Node 的状态变化，然后将 Node 的状态收集起来，并通过 Heartbeat 的方式上报给 APIServer。
- 再比如 CPU Manager，就负责维护该 Node 的 CPU 核的信息，以便在 Pod 通过 cpuset 的方式请求 CPU 核的时候，能够正确地管理 CPU 核的使用量和可用量。



那么这个 **SyncLoop，又是如何根据 Pod 对象的变化，来进行容器操作的呢？**

实际上，kubelet 也是通过 **Watch 机制**，监听了与自己相关的 Pod 对象的变化。当然，这个 Watch 的过滤条件是该 Pod 的 nodeName 字段与自己相同。kubelet 会把这些 Pod 的信息缓存在自己的内存里。

而<u>当一个 Pod 完成调度、与一个 Node 绑定起来之后， 这个 Pod 的变化就会触发 kubelet 在控制循环里注册的 Handler</u>，也就是上图中的 **HandlePods** 部分。此时，通过检查该 Pod 在 kubelet 内存里的状态，kubelet 就能够判断出这是一个新调度过来的 Pod，从而触发 Handler 里 ADD 事件对应的处理逻辑。

在具体的处理过程当中，kubelet 会**启动一个名叫 Pod Update Worker 的、单独的 Goroutine 来完成对 Pod 的处理工作**。

比如，如果是 **ADD 事件**的话，kubelet 就会为这个新的 Pod 生成对应的 Pod Status，检查 Pod 所声明使用的 Volume 是不是已经准备好。然后，调用下层的容器运行时（比如 Docker），开始创建这个 Pod 所定义的容器。

而如果是 **UPDATE** 事件的话，kubelet 就会根据 Pod 对象具体的变更情况，调用下层容器运行时进行容器的重建工作。

在这里需要注意的是，<u>kubelet 调用下层容器运行时的执行过程，并不会直接调用 Docker 的 API，而是通过一组叫作 **CRI**（Container Runtime Interface，容器运行时接口）的 **gRPC** 接口来间接执行的</u>。

**Kubernetes 项目之所以要在 kubelet 中引入这样一层单独的抽象，当然是为了对 Kubernetes 屏蔽下层容器运行时的差异。**

实际上，<u>对于 1.6 版本之前的 Kubernetes 来说，它就是直接调用 Docker 的 API 来创建和管理容器的</u>。

> 但是，正如我在本专栏开始介绍容器背景的时候提到过的，Docker 项目风靡全球后不久，CoreOS 公司就推出了 rkt 项目来与 Docker 正面竞争。
>
> 在这种背景下，Kubernetes 项目的默认容器运行时，自然也就成了两家公司角逐的重要战场。毋庸置疑，Docker 项目必然是 Kubernetes 项目最依赖的容器运行时。但凭借与 Google 公司非同一般的关系，CoreOS 公司还是在 2016 年成功地将对 rkt 容器的支持，直接添加进了 kubelet 的主干代码里。
>
> 不过，这个“赶鸭子上架”的举动，并没有为 rkt 项目带来更多的用户，反而给 kubelet 的维护人员，带来了巨大的负担。不难想象，在这种情况下， kubelet 任何一次重要功能的更新，都不得不考虑 Docker 和 rkt 这两种容器运行时的处理场景，然后分别更新 Docker 和 rkt 两部分代码。更让人为难的是，由于 rkt 项目实在太小众，kubelet 团队所有与 rkt 相关的代码修改，都必须依赖于 CoreOS 的员工才能做到。这不仅拖慢了 kubelet 的开发周期，也给项目的稳定性带来了巨大的隐患。
>
> 与此同时，在 2016 年，Kata Containers 项目的前身 runV 项目也开始逐渐成熟，这种**基于虚拟化技术的强隔离容器**，与 Kubernetes 和 Linux 容器项目之间具有良好的互补关系。所以，在 Kubernetes 上游，对虚拟化容器的支持很快就被提上了日程。
>
> 不过，虽然**虚拟化容器运行时有各种优点，但它与 Linux 容器截然不同的实现方式，使得它跟 Kubernetes 的集成工作，比 rkt 要复杂得多**。如果此时，再把对 runV 支持的代码也一起添加到 kubelet 当中，那么接下来 kubelet 的维护工作就可以说完全没办法正常进行了。
>
> 所以，在 2016 年，SIG-Node 决定开始动手解决上述问题。而**解决办法就是把 kubelet 对容器的操作，统一地抽象成一个接口。**这样，kubelet 就只需要跟这个接口打交道了。而作为具体的容器项目，**比如 Docker、 rkt、runV，它们就只需要自己提供一个该接口的实现，然后对 kubelet 暴露出 gRPC 服务即可。**

这一层统一的容器操作接口，就是 CRI 了。

而在有了 CRI 之后，Kubernetes 以及 kubelet 本身的架构，就可以用如下所示的一幅示意图来描述。

<img src="/Users/breeze/Library/Application Support/typora-user-images/image-20220209105048650.png" alt="image-20220209105048650" style="zoom:50%;" />

可以看到，当 Kubernetes 通过编排能力创建了一个 Pod 之后，调度器会为这个 Pod 选择一个具体的节点来运行。这时候，kubelet 当然就会通过前面讲解过的 SyncLoop 来判断需要执行的具体操作，比如创建一个 Pod。

那么此时，kubelet 实际上就会调用一个叫作 **GenericRuntime** 的通用组件来**发起创建 Pod 的 CRI 请求**。

**那么，这个 CRI 请求，又该由谁来响应呢？**

如果你使用的容器项目是 Docker 的话，那么负责响应这个请求的就是一个叫作 **dockershim** 的组件。它会把 CRI 请求里的内容拿出来，然后组装成 Docker API 请求发给 Docker Daemon。

需要注意的是，在 Kubernetes 目前的实现里，dockershim 依然是 kubelet 代码的一部分。当然，在将来，dockershim 肯定会被从 kubelet 里移出来，甚至直接被废弃掉。

<u>而更普遍的场景，就是你需要在每台宿主机上单独安装一个负责响应 CRI 的组件，这个组件，一般被称作 **CRI shim**。</u>顾名思义，CRI shim 的工作，就是扮演 kubelet 与容器项目之间的“垫片”（shim）。所以它的作用非常单一，那就是实现 CRI 规定的每个接口，然后把具体的 CRI 请求“翻译”成对后端容器项目的请求或者操作。

**总结**

在这个过程中，kubelet 的 **SyncLoop** 和 **CRI** 的设计，是其中最重要的两个关键点。也正是基于以上设计，SyncLoop 本身就要求这个控制循环是绝对不可以被阻塞的。所以，凡是在 kubelet 里有可能会耗费大量时间的操作，比如准备 Pod 的 Volume、拉取镜像等，SyncLoop 都会开启单独的 Goroutine 来进行操作。



### D2: CRI详解 与 容器运行时

#### CRI Shim

简要回顾有个CRI之后，k8s的架构图：

<img src="/Users/breeze/Library/Application Support/typora-user-images/image-20220209105048650.png" alt="image-20220209105048650" />

在上一篇文章中提到，CRI 机制能够发挥作用的核心，就在于**每一种容器项目现在都可以自己实现一个 CRI shim，自行对 CRI 请求进行处理**。这样，**Kubernetes 就有了一个统一的容器抽象层，使得下层容器运行时可以自由地对接进入 Kubernetes 当中**。

所以说，这里的 CRI shim，就是容器项目的维护者们自由发挥的“场地”了。而除了 dockershim 之外，其他容器运行时的 CRI shim，都是需要额外部署在宿主机上的。

**举个例子。**

CNCF 里的 **containerd** 项目，就可以提供一个典型的 CRI shim 的能力，即：<u>将 Kubernetes 发出的 CRI 请求，转换成对 containerd 的调用，然后创建出 runC 容器</u>。

而 **runC** 项目，才是负责执行我们前面讲解过的设置容器 Namespace、Cgroups 和 chroot 等基础操作的组件。所以，这几层的组合关系，可以用如下所示的示意图来描述。

<img src="/Users/breeze/Library/Application Support/typora-user-images/image-20220209112348199.png" alt="image-20220209112348199" style="zoom:50%;" />

**而作为一个 CRI shim，containerd 对 CRI 的具体实现，又是怎样的呢？**

我们先来看一下 CRI 这个接口的定义。下面这幅示意图，就展示了 CRI 里主要的待实现接口。

<img src="/Users/breeze/Library/Application Support/typora-user-images/image-20220209112512699.png" alt="image-20220209112512699" style="zoom:50%;" />

具体地说，**我们可以把 CRI 分为两组：**

- 第一组，是 **RuntimeService**。它提供的接口，主要是跟容器相关的操作。比如，创建和启动容器、删除容器、执行 exec 命令等等
- 第二组，是 **ImageService**。它提供的接口，主要是容器镜像相关的操作，比如拉取镜像、删除镜像等等。

接下来，主要讲解 **RuntimeService** 部分。

##### RuntimeService

在这一部分，**CRI 设计的一个重要原则，就是确保这个接口本身，只关注容器，不关注 Pod。**

这样做的原因，也很容易理解。

- 第一，Pod 是 Kubernetes 的编排概念，而不是容器运行时的概念。所以，我们就不能假设所有下层容器项目，都能够暴露出可以直接映射为 Pod 的 API。
- 第二，如果 CRI 里引入了关于 Pod 的概念，那么接下来只要 Pod API 对象的字段发生变化，那么 CRI 就很有可能需要变更。而在 Kubernetes 开发的前期，Pod 对象的变化还是比较频繁的，但对于 CRI 这样的标准接口来说，这个变更频率就有点麻烦了。

所以，在 CRI 的设计里，并没有一个直接创建 Pod 或者启动 Pod 的接口。

不过，相信你也已经注意到了，CRI 里还是有一组叫作 **RunPodSandbox** 的接口的。

这个 PodSandbox，对应的并不是 Kubernetes 里的 Pod API 对象，而只是抽取了 Pod 里的一部分与容器运行时相关的字段，比如 HostName、DnsConfig、CgroupParent 等。所以说，PodSandbox 这个接口描述的，其实是 Kubernetes 将 Pod 这个概念映射到容器运行时层面所需要的字段，或者说是一个 Pod 对象子集。

而作为具体的容器项目，你就需要自己决定如何使用这些字段来实现一个 Kubernetes 期望的 Pod 模型。这里的原理，可以用如下所示的示意图来表示清楚。

![image-20220209133642073](/Users/breeze/Library/Application Support/typora-user-images/image-20220209133642073.png)

比如，<u>当我们执行 kubectl run 创建了一个名叫 foo 的、包括了 A、B 两个容器的 Pod 之后。这个 Pod 的信息最后来到 kubelet，kubelet 就会按照图中所示的顺序来调用 CRI 接口。</u>

在具体的 CRI shim 中，这些接口的实现是可以完全不同的。比如，如果是 Docker 项目，dockershim 就会<u>创建出一个名叫 foo 的 Infra 容器（pause 容器），用来“hold”住整个 Pod 的 Network Namespace。</u>

而如果是基于虚拟化技术的容器，比如 Kata Containers 项目，它的 CRI 实现就会直接创建出一个轻量级虚拟机来充当 Pod。

此外，需要注意的是，在 RunPodSandbox 这个接口的实现中，你还需要调用 networkPlugin.SetUpPod(…) 来为这个 Sandbox 设置网络。这个 SetUpPod(…) 方法，实际上就在执行 CNI 插件里的 add(…) 方法，也就是我在前面为你讲解过的 CNI 插件为 Pod 创建网络，并且把 Infra 容器加入到网络中的操作。

接下来，kubelet 继续调用 CreateContainer 和 StartContainer 接口来创建和启动容器 A、B。对应到 dockershim 里，就是直接启动 A，B 两个 Docker 容器。所以最后，宿主机上会出现三个 Docker 容器组成这一个 Pod。

而如果是 Kata Containers 的话，CreateContainer 和 StartContainer 接口的实现，就只会在前面创建的轻量级虚拟机里创建两个 A、B 容器对应的 Mount Namespace。所以，最后在宿主机上，只会有一个叫作 foo 的轻量级虚拟机在运行。关于像 Kata Containers 或者 gVisor 这种所谓的安全容器项目，我会在下一篇文章中为你详细介绍。

**除了上述对容器生命周期的实现之外，CRI shim 还有一个重要的工作，就是如何实现 exec、logs 等接口**。这些接口跟前面的操作有一个很大的不同，就是**这些 gRPC 接口调用期间，kubelet 需要跟容器项目维护一个长连接来传输数据**。这种 API，我们就称之为 **Streaming API**。

CRI shim 里对 Streaming API 的实现，依赖于一套独立的 Streaming Server 机制。这一部分原理，可以用如下所示的示意图来描述。

![image-20220209135518858](/Users/breeze/Library/Application Support/typora-user-images/image-20220209135518858.png)

可以看到，当我们**对一个容器执行 kubectl exec 命令的时候**，这个请求首先交给 API Server，然后 API Server 就会调用 kubelet 的 Exec API。

这时，kubelet 就会调用 CRI 的 Exec 接口，而负责响应这个接口的，自然就是具体的 CRI shim。

<u>但在这一步，CRI shim 并不会直接去调用后端的容器项目（比如 Docker ）来进行处理，而只会返回一个 URL 给 kubelet</u>。

这个 URL，就是该 CRI shim 对应的 Streaming Server 的地址和端口。

而 kubelet 在拿到这个 URL 之后，就会把它以 Redirect 的方式返回给 API Server。所以这时候，API Server 就会通过重定向来向 Streaming Server 发起真正的 /exec 请求，与它建立长连接。

当然，这个 Streaming Server 本身，是需要通过使用 SIG-Node 为你维护的 Streaming API 库来实现的。并且，Streaming Server 会在 CRI shim 启动时就一起启动。此外，Stream Server 这一部分具体怎么实现，完全可以由 CRI shim 的维护者自行决定。比如，对于 Docker 项目来说，dockershim 就是直接调用 Docker 的 Exec API 来作为实现的。

以上，就是 CRI 的设计以及具体的工作原理了。



**总结**

在本篇文章中，详细解读了 CRI 的设计和具体工作原理，并梳理了实现 CRI 接口的核心流程。

从这些讲解中不难看出，CRI 这个接口的设计，实际上还是比较宽松的。这就意味着，作为容器项目的维护者，我在实现 CRI 的具体接口时，往往拥有着很高的自由度，这个自由度不仅包括了容器的生命周期管理，也包括了如何将 Pod 映射成为我自己的实现，还包括了如何调用 CNI 插件来为 Pod 设置网络的过程。

所以说，**当你对容器这一层有特殊的需求时，我一定优先建议你考虑实现一个自己的 CRI shim ，而不是修改 kubelet 甚至容器项目的代码。**这样通过插件的方式定制 Kubernetes 的做法，也是整个 Kubernetes 社区最鼓励和推崇的一个最佳实践。这也正是为什么像 Kata Containers、gVisor 甚至虚拟机这样的“非典型”容器，都可以无缝接入到 Kubernetes 项目里的重要原因。



### D3: Kata Containers 与 gVisor

待学习...





## P7: k8s容器监控与日志



### D1: Prometheus、Metrics Server与Kubernetes监控体系



#### Prometheus

首先需要明确指出的是，Kubernetes 项目的监控体系曾经非常繁杂，在社区中也有很多方案。但这套体系发展到今天，已经完全演变成了以 **Prometheus** 项目为核心的一套统一的方案。

实际上，**Prometheus** 项目是当年 CNCF 基金会起家时的“第二把交椅”。而这个项目发展到今天，已经全面接管了 Kubernetes 项目的整套监控体系。

比较有意思的是，Prometheus 项目与 Kubernetes 项目一样，也来自于 Google 的 Borg 体系，它的原型系统，叫作 BorgMon，是一个几乎与 Borg 同时诞生的内部监控系统。而 Prometheus 项目的发起原因也跟 Kubernetes 很类似，都是希望通过对用户更友好的方式，将 Google 内部系统的设计理念，传递给用户和开发者。

作为一个监控系统，Prometheus 项目的作用和工作方式，其实可以用如下所示的一张官方示意图来解释。

![image-20220209150401508](/Users/breeze/Library/Application Support/typora-user-images/image-20220209150401508.png)

可以看到，Prometheus 项目工作的核心，是使用 **Pull** （抓取）的方式去搜集被监控对象的 **Metrics** 数据（监控指标数据），然后，再把这些数据保存在一个 **TSDB** （时间序列数据库，比如 OpenTSDB、InfluxDB 等）当中，以便后续可以按照时间进行检索。

有了这套核心监控机制， Prometheus 剩下的组件就是用来配合这套机制的运行。

- 比如 Pushgateway，可以允许被监控对象以 Push 的方式向 Prometheus 推送 Metrics 数据。

- 而 Alertmanager，则可以根据 Metrics 信息灵活地设置报警。

当然， Prometheus 最受用户欢迎的功能，还是通过 **Grafana** 对外暴露出的、可以灵活配置的监控数据可视化界面。



有了 Prometheus 之后，我们就可以按照 **Metrics 数据的来源**，来对 Kubernetes 的监控体系做一个汇总了。

#### Metrics 数据的来源

**第一种 Metrics，是宿主机的监控数据**。这部分数据的提供，需要借助一个由 Prometheus 维护的**Node Exporter** 工具。一般来说，Node Exporter 会以 DaemonSet 的方式运行在宿主机上。其实，所谓的 Exporter，就是代替被监控对象来对 Prometheus 暴露出可以被“抓取”的 Metrics 信息的一个辅助进程。

而 Node Exporter 可以暴露给 Prometheus 采集的 Metrics 数据， 也不单单是节点的负载（Load）、CPU 、内存、磁盘以及网络这样的常规信息，它的 Metrics 指标可以说是“包罗万象”，你可以查看这个列表来感受一下。

**第二种 Metrics，是来自于 Kubernetes 的 API Server、kubelet 等组件的 /metrics API**。除了常规的 CPU、内存的信息外，这部分信息还主要包括了各个组件的核心监控指标。

比如，对于 API Server 来说，它就会在 /metrics API 里，暴露出各个 Controller 的工作队列（Work Queue）的长度、请求的 QPS 和延迟数据等等。这些信息，是检查 Kubernetes 本身工作情况的主要依据。

**第三种 Metrics，是 Kubernetes 相关的监控数据。**这部分数据，一般叫作 Kubernetes 核心监控数据（core metrics）。这其中包括了 Pod、Node、容器、Service 等主要 Kubernetes 核心概念的 Metrics。

其中，容器相关的 Metrics 主要来自于 kubelet 内置的 **cAdvisor** 服务。在 kubelet 启动后，cAdvisor 服务也随之启动，而它能够提供的信息，可以细化到每一个容器的 CPU 、文件系统、内存、网络等资源的使用情况。

需要注意的是，这里提到的 Kubernetes 核心监控数据，**其实使用的是 Kubernetes 的一个非常重要的扩展能力，叫作 Metrics Server**。

#### Metric Server

**Metrics Server** 在 Kubernetes 社区的定位，其实是用来取代 Heapster 这个项目的。在 Kubernetes 项目发展的初期，Heapster 是用户获取 Kubernetes 监控数据（比如 Pod 和 Node的资源使用情况） 的主要渠道。而后面提出来的 Metrics Server，则把这些信息，**通过标准的 Kubernetes API 暴露了出来**。这样，Metrics 信息就跟 Heapster 完成了解耦，允许 Heapster 项目慢慢退出舞台。

而有了 Metrics Server 之后，用户就可以通过标准的 Kubernetes API 来访问到这些监控数据了。比如，下面这个 URL：

```
http://127.0.0.1:8001/apis/metrics.k8s.io/v1beta1/namespaces/<namespace-name>/pods/<pod-name>
```

当你访问这个 Metrics API 时，它就会为你返回一个 Pod 的监控数据，而这些数据，其实是从 **kubelet 的 Summary API** （即 :/stats/summary）采集而来的。

**Summary API** 返回的信息，既包括了 **cAdVisor** 的监控数据，也包括了 kubelet 本身汇总的信息。

需要指出的是， <u>Metrics Server 并不是 kube-apiserver 的一部分</u>，而是通过 **Aggregator** 这种插件机制，在独立部署的情况下同 kube-apiserver 一起统一对外服务的。

这里，Aggregator APIServer 的工作原理，可以用如下所示的一幅示意图来表示清楚：

![image-20220209153152523](/Users/breeze/Library/Application Support/typora-user-images/image-20220209153152523.png)

可以看到，当 Kubernetes 的 API Server 开启了 **Aggregator** 模式之后，你再访问 apis/metrics.k8s.io/v1beta1 的时候，实际上访问到的是一个叫作 **kube-aggregator 的代理**。

而 kube-apiserver，正是这个代理的一个后端；而 Metrics Server，则是另一个后端。

而且，在这个机制下，你还可以添加更多的后端给这个 kube-aggregator。<u>所以 kube-aggregator 其实就是一个根据 URL 选择具体的 API 后端的代理服务器</u>。

通过这种方式，我们就可以很方便地扩展 Kubernetes 的 API 了。而 Aggregator 模式的开启也非常简单：如果你是使用 kubeadm 或者官方的 [kube-up.sh 脚本](https://github.com/kubernetes/kubernetes/blob/master/cluster/kube-up.sh)部署 Kubernetes 集群的话，Aggregator 模式就是默认开启的；如果是手动 DIY 搭建的话，你就需要在 kube-apiserver 的启动参数里加上如下所示的配置：

```shell
--requestheader-client-ca-file=<path to aggregator CA cert>
--requestheader-allowed-names=front-proxy-client
--requestheader-extra-headers-prefix=X-Remote-Extra-
--requestheader-group-headers=X-Remote-Group
--requestheader-username-headers=X-Remote-User
--proxy-client-cert-file=<path to aggregator proxy cert>
--proxy-client-key-file=<path to aggregator proxy key>
```

而这些配置的作用，主要就是为 Aggregator 这一层设置对应的 Key 和 Cert 文件。而这些文件的生成，就需要你自己手动完成了，具体流程请参考这篇[官方文档](https://github.com/kubernetes-incubator/apiserver-builder/blob/master/docs/concepts/auth.md)。

Aggregator 功能开启之后，你只需要将 Metrics Server 的 YAML 文件部署起来，如下所示：

```shell
$ git clone https://github.com/kubernetes-incubator/metrics-server
$ cd metrics-server
$ kubectl create -f deploy/1.8+/
```

接下来，你就会看到 metrics.k8s.io 这个 API 出现在了你的 Kubernetes API 列表当中。

在理解了 Prometheus 关心的三种监控数据源，以及 Kubernetes 的核心 Metrics 之后，作为用户，你其实要做的就是将 Prometheus Operator 在 Kubernetes 集群里部署起来。然后，按照本篇文章一开始介绍的架构，把上述 Metrics 源配置起来，让 Prometheus 自己去进行采集即可。



**总结**

在具体的监控指标规划上，我建议你遵循业界通用的 **USE 原则**和 **RED 原则**。

其中，**USE 原则**指的是，按照如下三个维度来规划**资源**监控指标：

- 利用率（Utilization），资源被有效利用起来提供服务的平均时间占比；
- 饱和度（Saturation），资源拥挤的程度，比如工作队列的长度；
- 错误率（Errors），错误的数量。

而 **RED 原则**指的是，按照如下三个维度来规划**服务**监控指标：

- 每秒请求数量（Rate）；
- 每秒错误数量（Errors）；
- 服务响应时间（Duration）。

不难发现， USE 原则主要关注的是“**资源**”，比如节点和容器的资源使用情况，而 RED 原则主要关注的是“**服务**”。

比如 kube-apiserver 或者某个应用的工作情况。这两种指标，在我今天为你讲解的 Kubernetes + Prometheus 组成的监控体系中，都是可以完全覆盖到的。



### D2: Custom Metrics: 自定义监控指标





### D3: 容器日志收集与管理

首先需要明确的是，Kubernetes 里面对容器日志的处理方式，都叫作 **cluster-level-logging**，即：<u>这个日志处理系统，与容器、Pod 以及 Node 的生命周期都是完全无关的。</u>

这种设计当然是为了保证，<u>无论是容器挂了、Pod 被删除，甚至节点宕机的时候，应用的日志依然可以被正常获取到。</u>

而对于一个容器来说，当应用把日志输出到 **stdout** 和 **stderr** 之后，容器项目在默认情况下就会把这些日志输出到宿主机上的一个 **JSON 文件**里。这样，你通过 **kubectl logs** 命令就可以看到这些容器的日志了。

上述机制，就是我们今天要讲解的容器日志收集的基础假设。而如果你的应用是把文件输出到其他地方，比如直接输出到了容器里的某个文件里，或者输出到了远程存储里，那就属于特殊情况了。当然，我在文章里也会对这些特殊情况的处理方法进行讲述。

而 Kubernetes 本身，实际上是不会为你做容器日志收集工作的，所以为了实现上述 cluster-level-logging，你需要在部署集群的时候，提前对具体的日志方案进行规划。而 Kubernetes 项目本身，主要为你推荐了**三种日志方案**。

- **第一种，在 Node 上部署 logging agent，将日志文件转发到后端存储里保存起来**。这个方案的架构图如下所示。

  <img src="/Users/breeze/Library/Application Support/typora-user-images/image-20220209162136151.png" alt="image-20220209162136151" style="zoom:50%;" />

不难看到，这里的核心就在于 **logging agent** ，它一般都会以 DaemonSet 的方式运行在节点上，然后将宿主机上的容器日志目录挂载进去，最后由 logging-agent 把日志转发出去。

**举个例子**

我们可以通过 **Fluentd** 项目作为宿主机上的 **logging-agent**，然后把日志转发到远端的 **ElasticSearch** 里保存起来供将来进行检索。另外，在很多 Kubernetes 的部署里，会自动为你启用 **logrotate**，在日志文件超过 10MB 的时候自动对日志文件进行 rotate 操作。

可以看到，在 Node 上部署 logging agent 最大的**优点**，在于**一个节点只需要部署一个 agent，并且不会对应用和 Pod 有任何侵入性**。所以，这个方案，在社区里是最常用的一种。

但是也不难看到，这种方案的**不足**之处就在于，它要求应用输出的日志，都**必须是直接输出到容器的 stdout 和 stderr 里**。

所以，

- **第二种方案，就是对这种特殊情况的一个处理**，即：当容器的日志只能输出到某些文件里的时候，我们可以通过一个 sidecar 容器把这些日志文件重新输出到 sidecar 的 stdout 和 stderr 上，这样就能够继续使用第一种方案了。这个方案的具体工作原理，如下所示。

  ![image-20220209164301579](/Users/breeze/Library/Application Support/typora-user-images/image-20220209164301579.png)

比如，现在我的应用 Pod 只有一个容器，它会把日志输出到容器里的 /var/log/1.log 和 2.log 这两个文件里。这个 Pod 的 YAML 文件如下所示：

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: counter
spec:
  containers:
  - name: count
    image: busybox
    args:
    - /bin/sh
    - -c
    - >
      i=0;
      while true;
      do
        echo "$i: $(date)" >> /var/log/1.log;
        echo "$(date) INFO $i" >> /var/log/2.log;
        i=$((i+1));
        sleep 1;
      done
    volumeMounts:
    - name: varlog
      mountPath: /var/log
  volumes:
  - name: varlog
    emptyDir: {}
```

在这种情况下，你用 kubectl logs 命令是看不到应用的任何日志的。而且我们前面讲解的、最常用的方案一，也是没办法使用的。

那么这个时候，我们就可以为这个 Pod 添加两个 **sidecar** 容器，分别将上述两个日志文件里的内容重新以 stdout 和 stderr 的方式输出出来，这个 YAML 文件的写法如下所示：

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: counter
spec:
  containers:
  - name: count
    image: busybox
    args:
    - /bin/sh
    - -c
    - >
      i=0;
      while true;
      do
        echo "$i: $(date)" >> /var/log/1.log;
        echo "$(date) INFO $i" >> /var/log/2.log;
        i=$((i+1));
        sleep 1;
      done
    volumeMounts:
    - name: varlog
      mountPath: /var/log
  - name: count-log-1
    image: busybox
    args: [/bin/sh, -c, 'tail -n+1 -f /var/log/1.log']
    volumeMounts:
    - name: varlog
      mountPath: /var/log
  - name: count-log-2
    image: busybox
    args: [/bin/sh, -c, 'tail -n+1 -f /var/log/2.log']
    volumeMounts:
    - name: varlog
      mountPath: /var/log
  volumes:
  - name: varlog
    emptyDir: {}
```

这时候，你就可以通过 kubectl logs 命令查看这两个 sidecar 容器的日志，间接看到应用的日志内容了，如下所示：

```shell
$ kubectl logs counter count-log-1
0: Mon Jan 1 00:00:00 UTC 2001
1: Mon Jan 1 00:00:01 UTC 2001
2: Mon Jan 1 00:00:02 UTC 2001
...
$ kubectl logs counter count-log-2
Mon Jan 1 00:00:00 UTC 2001 INFO 0
Mon Jan 1 00:00:01 UTC 2001 INFO 1
Mon Jan 1 00:00:02 UTC 2001 INFO 2
...
```

由于 sidecar 跟主容器之间是共享 Volume 的，所以这里的 sidecar 方案的额外性能损耗并不高，也就是多占用一点 CPU 和内存罢了。

但需要注意的是，这时候，宿主机上实际上会存在两份相同的日志文件：一份是应用自己写入的；另一份则是 sidecar 的 stdout 和 stderr 对应的 JSON 文件。这**对磁盘是很大的浪费**。所以说，除非万不得已或者应用容器完全不可能被修改，否则我还是建议你直接使用方案一，或者直接使用下面的第三种方案。



- **第三种方案，就是通过一个 sidecar 容器，直接把应用的日志文件发送到远程存储里面去。**也就是相当于把方案一里的 logging agent，放在了应用 Pod 里。这种方案的架构如下所示：

  ![image-20220209165114942](/Users/breeze/Library/Application Support/typora-user-images/image-20220209165114942.png)

在这种方案里，你的应用还**可以直接把日志输出到固定的文件里而不是 stdout**，你的 logging-agent 还可以使用 fluentd，后端存储还可以是 ElasticSearch。只不过， fluentd 的输入源，变成了应用的日志文件。一般来说，我们会把 fluentd 的输入源配置保存在一个 ConfigMap 里，如下所示：

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: fluentd-config
data:
  fluentd.conf: |
    <source>
      type tail
      format none
      path /var/log/1.log
      pos_file /var/log/1.log.pos
      tag count.format1
    </source>
    
    <source>
      type tail
      format none
      path /var/log/2.log
      pos_file /var/log/2.log.pos
      tag count.format2
    </source>
    
    <match **>
      type google_cloud
    </match>
```

然后，我们在应用 Pod 的定义里，就可以声明一个 Fluentd 容器作为 sidecar，专门负责将应用生成的 1.log 和 2.log 转发到 ElasticSearch 当中。这个配置，如下所示：

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: counter
spec:
  containers:
  - name: count
    image: busybox
    args:
    - /bin/sh
    - -c
    - >
      i=0;
      while true;
      do
        echo "$i: $(date)" >> /var/log/1.log;
        echo "$(date) INFO $i" >> /var/log/2.log;
        i=$((i+1));
        sleep 1;
      done
    volumeMounts:
    - name: varlog
      mountPath: /var/log
  - name: count-agent
    image: k8s.gcr.io/fluentd-gcp:1.30
    env:
    - name: FLUENTD_ARGS
      value: -c /etc/fluentd-config/fluentd.conf
    volumeMounts:
    - name: varlog
      mountPath: /var/log
    - name: config-volume
      mountPath: /etc/fluentd-config
  volumes:
  - name: varlog
    emptyDir: {}
  - name: config-volume
    configMap:
      name: fluentd-config
```

可以看到，这个 Fluentd 容器使用的输入源，就是通过引用我们前面编写的 ConfigMap 来指定的。这里我用到了 Projected Volume 来把 ConfigMap 挂载到 Pod 里。

需要注意的是，这种方案虽然部署简单，并且对宿主机非常友好，但是这个 sidecar 容器很可能会消耗较多的资源，甚至拖垮应用容器。并且，由于日志还是没有输出到 stdout 上，所以你通过 kubectl logs 是看不到任何日志输出的。

以上，就是 Kubernetes 项目对容器应用日志进行管理最常用的三种手段了。



**总结**

综合对比以上三种方案，我**比较建议将应用日志输出到 stdout 和 stderr**，然后通过在宿主机上部署 logging-agent 的方式来集中处理日志。这种方案不仅管理简单，kubectl logs 也可以用，而且可靠性高，并且宿主机本身，很可能就自带了 rsyslogd 等非常成熟的日志收集组件来供你使用。

除此之外，还有一种方式就是**在编写应用的时候，就直接指定好日志的存储后端**，如下所示：

![image-20220209165819235](/Users/breeze/Library/Application Support/typora-user-images/image-20220209165819235.png)

在这种方案下，Kubernetes 就完全不必操心容器日志的收集了，这**对于本身已经有完善的日志处理系统的公司来说，是一个非常好的选择。**

最后需要指出的是，**无论是哪种方案，你都必须要及时将这些日志文件从宿主机上清理掉，或者给日志目录专门挂载一些容量巨大的远程盘。**否则，一旦主磁盘分区被打满，整个系统就可能会陷入崩溃状态，这是非常麻烦的。













## PS

**Istio** 这个微服务治理项目了。**Istio** 项目使用 sidecar 容器完成微服务治理的原理

我们可以通过 **Fluentd** 项目作为宿主机上的 **logging-agent**，然后把日志转发到远端的 **ElasticSearch** 里保存起来供将来进行检索
