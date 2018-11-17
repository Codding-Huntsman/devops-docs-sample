---
title: "Multi-node 模式"
---

## Multi-Node 模式

`Multi-Node` 即多节点集群部署，部署前建议您选择集群中任意一个节点作为一台任务执行机 (taskbox)，为准备部署的集群中其他节点执行部署的任务，且 taskbox 应能够与待部署的其他节点进行 **ssh 通信**。

## 前提条件

已购买 KubeSphere 高级版，并已下载了高级版的 Installer 至目标安装机器。

### 第一步: 准备主机

您可以参考以下节点规格 准备 **`至少 2 台`** 符合要求的主机节点开始 `multi-node` 模式的部署。

| 操作系统 | 最小配置 | 推荐配置 |
| --- | --- | --- |
| ubuntu 16.04/18.04 LTS 64bit | CPU：8 核 <br/> 内存：12 G <br/> 磁盘：40 G | CPU：16 核 <br/> 内存：32 G <br/> 磁盘：100 G |
| CentOS 7.4/7.5 64bit | CPU：8 核 <br/> 内存：12 G <br/> 磁盘：40 G | CPU：16 核 <br/> 内存：32 G <br/> 磁盘：100 G |

以下用一个示例介绍 multi-node 模式部署多节点，此示例准备了 3 台主机，以主机名为 master 的节点作为任务执行机 taskbox，各节点主机名可由用户自定义。

> 说明：高级版支持 master 和 etcd 节点高可用配置，本示例仅演示 etcd 高可用配置，若需要配置请参阅 [Master 节点高可用配置](../master-ha)。

假设主机信息如下所示：

| 主机IP | 主机名 | 集群角色 |
| --- | --- | --- |
|192.168.0.1|master|master，etcd，node|
|192.168.0.2|node1|node，etcd|
|192.168.0.3|node2|node, etcd |

**集群架构：** 单 master 多 etcd 多 node

![集群架构图](/pic04.svg)

### 第二步: 准备安装配置文件

**1.** 在获取安装包后，执行以下命令。

```bash
$ tar -zxvf KubeSphere-Installer-Advanced-v1.0.0.tar.gz
```

**2.** 进入 “`KubeSphere-Installer-Advanced-v1.0.0`” 目录

```bash
$ cd KubeSphere-Installer-Advanced-v1.0.0
```

**3.** 编辑主机配置文件 `conf/hosts.ini`，为了对待部署目标机器及部署流程进行集中化管理配置，集群中各个节点在主机配置文件 `hosts.ini` 中应参考如下配置。以下示例在 CentOS 7.5 上使用 `root` 用户安装，每台机器信息占一行，不能分行。若以 ubuntu 用户进行安装，可参考主机配置文件的注释 `non-root` 示例部分编辑。

> 注意：
> - etcd 作为一个高可用键值存储系统，etcd 节点至少需要 1 个，部署多个 etcd 能够使集群更可靠，etcd 节点个数需要设置为奇数个，在 "conf/hosts.ini" 的 `[etcd]` 部分填入主机名即可，示例将在 3 个节点部署 etcd。
> - 若安装时需要配置 master 节点高可用，"conf/hosts.ini" 请参考 [master 节点高可用 - 修改主机文件配置](../master-ha/#修改主机配置文件)。

**root 配置示例：**

```ini
[all]
master ansible_connection=local  ip=192.168.0.1
node1  ansible_host=192.168.0.2  ip=192.168.0.2  ansible_ssh_pass=PASSWORD
node2  ansible_host=192.168.0.3  ip=192.168.0.3  ansible_ssh_pass=PASSWORD

[kube-master]
master

[kube-node]
master
node1
node2

[etcd]
master
node1
node2

[k8s-cluster:children]
kube-node
kube-master
```

> 说明：
>
> - [all] 中需要修改集群中各个节点的内网 IP 和主机 root 用户密码。主机名为 "master" 的节点作为 taskbox 无需填写密码，[all] 中其它参数比如 node1 和 node2 需要分别替换 `ansible_host` 和 `ip` 为当前 node1 和 node2 的内网 IP，`ansible_ssh_pass` 即替换为各自主机的 root 用户登录密码。
> -  "master" 节点作为 taskbox，用来执行整个集群的部署任务，同时 "master" 节点在 KubeSphere 集群中也作为 master 和 etcd ，应将主机名 "master" 填入 [kube-master] 和 [etcd] 部分。
> - 主机名为 "master"，"node1"，"node2" 的节点， 作为 KubeSphere 集群的 node 节点且将部署 etcd，主机名填入 [kube-node] 和 [etcd] 部分。<br>
>
> 参数解释：<br>
> - `ansible_connection`: 与主机的连接类型，此处设置为 `local` 即本地连接。
> - `ansible_host`: 集群中将要连接的主机名。
> - `ip`: 集群中将要连接的主机 IP。
> - `ansible_ssh_pass`: 待连接主机 root 用户的密码。


**5.** Multi-Node 模式安装 KubeSphere 可选择配置部署 NFS Server 到当前集群来提供持久化存储服务，方便初次安装但没有准备存储服务端的场景下进行部署测试。若在正式环境使用需准备相应的存储服务端，并配置 KubeSphere 支持的持久化存储服务。网络、存储等相关内容需在 `conf/vars.yml` 配置文件中指定或修改。本文档以配置 NFS Server 为例，仅需在 `vars.yml` 简单配置即可安装 NFS 作为默认存储类型，参数释义详见 [存储配置说明](../storage-configuration)。

> 说明：
> - 根据配置文件按需修改相关配置项，未做修改将以默认参数执行。
> - 网络：默认插件 `calico`。
> - 支持存储类型：[青云块存储](https://docs.qingcloud.com/product/storage/volume/)、[企业级分布式存储 NeonSAN](https://docs.qingcloud.com/product/storage/volume/super_high_performance_shared_volume/)、[NFS](https://kubernetes.io/docs/concepts/storage/volumes/#nfs)、[GlusterFS](https://www.gluster.org/)、[Ceph RBD](https://ceph.com/)、[Local Volume (仅支持 all-in-one)](https://kubernetes.io/docs/concepts/storage/volumes/#local)，存储配置相关的详细信息请参考 [存储配置说明](../storage-configuration)。
> - Multi-node 安装时需要配置持久化存储，因为它不支持 Local Volume，因此把 Local Volume 的配置修改为 false，然后配置持久化存储如 QingCloud-CSI (青云块存储插件)、NeonSAN CSI (NeonSAN 存储插件)、NFS、GlusterFS、CephRBD 等。如下所示配置 NFS，将 `nfs_server_enable` 和 `nfs_server_is_default_class` 设置为 true。
> - 由于 Kubernetes 集群的 Cluster IP 子网网段默认是 10.233.0.0/18，Pod 的子网网段默认是 10.233.64.0/18，因此部署 KubeSphere 的节点 IP 地址范围不应与以上两个网段有重复，若遇到地址范围冲突可在配置文件 `conf/vars.yaml` 修改 `kube_service_addresses` 或 `kube_pods_subnet` 的参数。

**存储配置示例：**

```yaml
# Local volume provisioner deployment(Only all-in-one)
local_volume_provisioner_enabled: false
local_volume_provisioner_storage_class: local
local_volume_is_default_class: false

# NFS-Server provisioner deployment
nfs_server_enable: true
nfs_server_is_default_class: true

```

### 第三步: 安装 KubeSphere

KubeSphere 多节点部署会自动化地进行环境和文件监测、平台依赖软件的安装、Kubernetes 和 etcd 集群的自动化部署，以及存储的自动化配置。Installer 默认安装的 Kubernetes 版本是 v1.12.2，安装成功后可通过 KubeSphere 控制台右上角点击关于查看安装的版本。KubeSphere 安装包将会自动安装一些依赖软件，如 Ansible (v2.4+)，Python-netaddr (v0.7.18+)，Jinja (v2.9+)。

参考以下步骤开始 multi-node 部署：

**1.** 进入 `scripts` 目录

```bash
$ cd scripts
```

**2.** 执行 `menu.sh` 脚本：

```bash
$ ./menu.sh
```

**3.** 输入数字 `2` 选择第二种 Multi-node 模式开始部署，安装程序会提示您是否已经配置过存储，若未配置请输入 "no"，返回目录继续配置存储并参考 [存储配置说明](../storage-configuration)。

```bash
################################################
         KubeSphere Installer Menu
################################################
*   1) All-in-one
*   2) Multi-node
*   3) Add-node(s)
*   4) Uninstall
*   5) Quit
################################################
https://kubesphere.io/               2018-11-16
################################################
Please input an option: 2

```

**4.** 测试 KubeSphere 集群部署是否成功：

**(1)** 待安装脚本执行完后，当看到如下 `"Successful"` 界面，则说明 KubeSphere 安装成功。若需要在外网访问，可能需要绑定公网 EIP 并配置端口转发，若公网 EIP 有防火墙，请在防火墙添加规则放行对应的端口，外部才能够访问。

```bash
successsful!
#####################################################
###              Welcome to KubeSphere!           ###
#####################################################

Console: http://192.168.0.1:32130
Account: admin
Password: passw0rd

NOTE：Please modify the default password after login.
#####################################################
```
**(2)** 安装成功后，浏览器访问对应的 url，即可进入 KubeSphere 登录界面，可使用默认的用户名和密码登录 KubeSphere 控制台体验，参阅 [快速入门](../../quick-start/quick-start-guide) 帮助您快速上手 KubeSphere。

![KubeSphere 控制台](/kubesphere-console.png)