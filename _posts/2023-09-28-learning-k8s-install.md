---
title: 学习k8s-安装
layout: post
date: '2023-09-28 16:23:00 +0800'
published: true
tags: 
- k8s
categories:
- K8s
---

参考k8s文档

<https://kubernetes.io/zh-cn/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/>

## 安装软件包

### apt

1. 更新 apt 包索引并安装使用 Kubernetes apt 仓库所需要的包：

    ``` shell
    sudo apt-get update
    # apt-transport-https 可能是一个虚拟包（dummy package）；如果是的话，你可以跳过安装这个包
    sudo apt-get install -y apt-transport-https ca-certificates curl
    ```

2. 下载用于 Kubernetes 软件包仓库的公共签名密钥。所有仓库都使用相同的签名密钥，因此你可以忽略URL中的版本：

    ``` shell
    curl -fsSL <https://pkgs.k8s.io/core:/stable:/v1.28/deb/Release.key> | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
    ```

3. 添加 Kubernetes apt 仓库：

    ``` shell
    #此操作会覆盖 /etc/apt/sources.list.d/kubernetes.list 中现存的所有配置。
    echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] <https://pkgs.k8s.io/core:/stable:/v1.28/deb/> /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
    ```

4. 更新 apt 包索引，安装 kubelet、kubeadm 和 kubectl，并锁定其版本：

    ``` shell
    sudo apt-get update
    sudo apt-get install -y kubelet kubeadm kubectl
    sudo apt-mark hold kubelet kubeadm kubectl
    ```

### yum

1. 将 SELinux 设置为 permissive 模式:

    ``` shell
    # 将 SELinux 设置为 permissive 模式（相当于将其禁用）
    sudo setenforce 0
    sudo sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config
    ```

    **注意:**
    - 通过运行命令 setenforce 0 和 sed ... 将 SELinux 设置为 permissive 模式相当于将其禁用。 这是允许容器访问主机文件系统所必需的，例如，某些容器网络插件需要这一能力。 你必须这么做，直到 kubelet 改进其对 SELinux 的支持。
    - 如果你知道如何配置 SELinux 则可以将其保持启用状态，但可能需要设定部分 kubeadm 不支持的配置。

2. 添加 Kubernetes 的 yum 仓库。在仓库定义中的 exclude 参数确保了与 Kubernetes 相关的软件包在运行 yum update 时不会升级，因为升级 Kubernetes 需要遵循特定的过程。

    ``` shell
    # 此操作会覆盖 /etc/yum.repos.d/kubernetes.repo 中现存的所有配置
    cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
    [kubernetes]
    name=Kubernetes
    baseurl=https://pkgs.k8s.io/core:/stable:/v1.28/rpm/
    enabled=1
    gpgcheck=1
    gpgkey=https://pkgs.k8s.io/core:/stable:/v1.28/rpm/repodata/repomd.xml.key
    exclude=kubelet kubeadm kubectl cri-tools kubernetes-cni
    EOF
    ```

3. 安装 kubelet、kubeadm 和 kubectl，并启用 kubelet 以确保它在启动时自动启动:

    ``` shell
    sudo yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes
    sudo systemctl enable --now kubelet 
    ```

## 创建集群

1. 主节点

    ``` shell
    kubeadm init --control-plane-endpoint=47.113.231.122
    ```

    install cri-docker

updating...............
