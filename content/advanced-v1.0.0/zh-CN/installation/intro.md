---
title: "安装说明"
---

[KubeSphere](https://kubesphere.io) 是在目前主流容器调度平台 [Kubernetes](https://kubernetes.io) 之上构建的 **企业级分布式多租户容器管理平台**，为用户提供简单易用的操作界面以及向导式操作方式，KubeSphere 提供了在生产环境集群部署的全栈化容器部署与管理平台，以及细粒度的资源监控和 CI/CD 流水线等。

## 安装 KubeSphere

KubeSphere 安装支持 [all-in-one](../all-in-one) 和 [multi-node](../multi-node) 两种模式，即支持单节点和多节点安装两种安装方式。 KubeSphere Installer 采用 [Ansible](https://www.ansible.com/) 对安装目标机器及安装流程进行集中化管理配置。采用预配置模板，可以在安装前通过对相关配置文件进行自定义实现对安装过程的预配置，以适应不同的 IT 环境，帮助您快速安装 KubeSphere。

**说明:**

> - KubeSphere 集群中由于安装服务的不同，分为管理节点和工作节点两个角色。
> - 当进行 all-in-one 模式进行单节点安装时，这个节点既是管理节点，也是工作节点。
> - 当进行 multi-node 模式安装多节点集群时，可在配置文件中设置集群角色。
> - 由于安装过程中需要更新操作系统和从镜像仓库拉取镜像，因此必须能够访问外网。
> - 如果是新安装的系统，在 Software Selection 界面需要把 OpenSSH Server 选上。

### All-in-One 模式

`All-in-One` 模式即单节点安装，支持一键安装，仅建议您用来测试或熟悉安装流程和了解 KubeSphere 高级版的功能特性，详见 [All-in-One 模式](../all-in-one)。在正式使用环境建议使用 Multi-node 模式。

### Multi-Node 模式

`Multi-Node` 即多节点集群安装，高级版支持 master 节点和 etcd 的高可用，支持在正式环境安装和使用，详见 [Multi-node 模式](../multi-node)。

#### 存储配置说明

Multi-Node 模式安装 KubeSphere 可选择配置部署 NFS Server 来提供持久化存储服务，方便初次安装但没有准备存储服务端的场景下进行部署测试。若在正式环境使用需配置 KubeSphere 支持的持久化存储服务，并准备相应的存储服务端。本文档说明安装过程中如何在 Installer 中配置 [青云块存储](https://docs.qingcloud.com/product/storage/volume/)、[企业级分布式存储 NeonSAN](https://docs.qingcloud.com/product/storage/volume/super_high_performance_shared_volume/)、[NFS](https://kubernetes.io/docs/concepts/storage/volumes/#nfs)、[GlusterFS](https://www.gluster.org/)、[Ceph RBD](https://ceph.com/) 这类持久化存储的安装参数，详见 [存储配置说明](../storage-configuration)。

#### Master 节点高可用配置

Multi-node 模式安装 KubeSphere 可以帮助用户顺利地部署环境，但是要应用到实际的生产环境就不得不考虑 master 节点的高可用问题了。本文档以配置负载均衡器 (Load Banlancer) 为例，引导您在安装过程中如何配置高可用的 master 节点，详见 [Master 节点高可用配置](../master-ha)。

## 集群节点扩容

安装 KubeSphere 后，在正式环境使用时可能会遇到服务器容量不足的情况，这时就需要添加新的节点 (node)，然后将应用系统进行水平扩展来完成对系统的扩容，配置详见 [集群节点扩容](../add-nodes)。

## 卸载

卸载将从机器中删除 KubeSphere，该卸载操作不可逆，详见 [卸载说明](../uninstall)。

## 组件版本信息

|  组件 |  版本 |
|---|---|
|KubeSphere| AE-v1.0.0|
|KubeSphere Console| Advanced Edition |
|Kubernetes| v1.12.2|
|OpenPitrix| v0.3.4|