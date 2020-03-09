Title: kubeadm 部署 kubernetes 集群
Date: 2017-10-20 14:22:06
Tags:


本文使用代理的方式来部署集群。

服务器信息
==========

-   master: 192.168.25.148
-   node: 192.168.25.149

master + node
=============

前期准备

### 配置 hostname

``` {.bash}
hostnamectl --static set-hostname master
hostname master
```

编辑 `/etc/hosts`, 添加一下内容

``` {.bash}
192.168.25.148 master
192.168.25.149 node
```

### 配置防火墙

关闭防火墙。

``` {.bash}
systemctl disable firewalld
systemctl stop firewalld
iptables -F
```

### 配置 selinux

关闭 SELinux

``` {.bash}
setenforce 0
```

编辑 `/etc/selinux/conf`

``` {.bash}
# 将以下内容
SELINUX=enforcing
# 替换为以下内容
SELINUX=disabled
```

### 关闭 swap

``` {.bash}
swapoff -a
```

修改 `/etc/fstab` ，注释掉 swap 的自动挂载。

### 配置系统

创建 `/etc/sysctl.d/k8s.conf` ，添加如下内容

``` {.linux-config}
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
vm.swappiness = 0
```

执行修改

``` {.bash}
modprobe br_netfilter
sysctl -p /etc/sysctl.d/k8s.conf
```

`sysctl -p /etc/sysctl.d/k8s.conf`

安装 Docker

### 安装依赖和工具

``` {.bash}
yum install -y yum-utils \
     device-mapper-persistent-data \
     lvm2
```

### 添加官方源

``` {.bash}
yum-config-manager \
     --add-repo \
     https://download.docker.com/linux/centos/docker-ce.repo
```

### 安装 Docker CE

Kubernetes 1.8 已经针对 Docker 的 1.11.2, 1.12.6, 1.13.1 和 17.03.2
等版本做了验证。 因为我们这里在各节点安装 docker 的 17.03.2 版本。

``` {.bash}
yum install -y --setopt=obsoletes=0 \
    docker-ce-17.03.2.ce-1.el7.centos \
    docker-ce-selinux-17.03.2.ce-1.el7.centos
```

### 配置防火墙

Docker 从 1.13 版本开始调整了默认的防火墙规则，禁用了 iptables filter
表中 FOWARD 链，这样会引起 Kubernetes 集群中跨 Node 的 Pod
无法通信，在各个 Docker 节点执行下面的命令：

``` {.bash}
iptables -P FORWARD ACCEPT
```

可在 docker 的 systemd unit 文件中以 ExecStartPost 加入上面的命令：

``` {.linux-config}
ExecStartPost=/usr/sbin/iptables -P FORWARD ACCEPT
```

### 配置 Docker 代理

``` {.bash}
$ mkdir -p /etc/systemd/system/docker.service.d/
$ vi /etc/systemd/system/docker.service.d/http-proxy.conf
[Service]
Environment="HTTP_PROXY=http://192.168.25.147:8118/" "HTTPS_PROXY=http://192.168.25.147:8118/" "NO_PROXY=localhost,127.0.0.1,10.0.0.0/8,192.168.25.148,192.168.25.149"
```

### 启动 Docker

``` {.bash}
systemctl enable docker
systemctl start docker
```

配置翻墙

略

安装 kubelet、kubectl 和 kubeadm

这两个包所有机器都需要安装。

### 配置 kubernetes 源

``` {.bash}
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg
        https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOF
```

### YUM 配置代理及缓存

``` {.bash}
$ vi /etc/yum.conf
...
proxy=socks5://127.0.0.1:1080
...
```

### 安装包

``` {.bash}
yum install -y kubelet kubeadm kubectl
```

### 更改 cgroup 驱动方式

docker 和 kubelet 的 cgroup 驱动方式可能不同，需要进行修复
编辑此文件，更改下面一行 `/etc/docker/daemon.json`

``` {.json}
{
  "exec-opts": ["native.cgroupdriver=systemd"]
}
```

重启 docker

``` {.bash}
systemctl restart docker
```

### 启动服务

``` {.bash}
systemctl enable kubelet
systemctl start kubelet
```

master
======

初始化集群

### 配置代理

``` {.bash}
export NO_PROXY="localhost,127.0.0.1,10.0.0.0/8,192.168.25.148"
export https_proxy=http://127.0.0.1:8118/
export http_proxy=http://127.0.0.1:8118/
```

### 初始化

**注意，输出的最后一行请记下，一会添加 node 的时候要用**

``` {.bash}
$ kubeadm init --kubernetes-version=v1.8.1 --pod-network-cidr=10.244.0.0/16 --token-ttl 0
[kubeadm] WARNING: kubeadm is in beta, please do not use it for production clusters.
[init] Using Kubernetes version: v1.8.1
[init] Using Authorization modes: [Node RBAC]
[kubeadm] WARNING: starting in 1.8, tokens expire after 24 hours by default (if you require a non-expiring token use --token-ttl 0)
[certificates] Generated CA certificate and key.
[certificates] Generated API server certificate and key.
[certificates] API Server serving cert is signed for DNS names [k8s kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local] and IPs [10.96.0.1 192.168.25.148]
[certificates] Generated API server kubelet client certificate and key.
[certificates] Generated service account token signing key and public key.
[certificates] Generated front-proxy CA certificate and key.
[certificates] Generated front-proxy client certificate and key.
[certificates] Valid certificates and keys now exist in "/etc/kubernetes/pki"
[kubeconfig] Wrote KubeConfig file to disk: "/etc/kubernetes/scheduler.conf"
[kubeconfig] Wrote KubeConfig file to disk: "/etc/kubernetes/admin.conf"
[kubeconfig] Wrote KubeConfig file to disk: "/etc/kubernetes/kubelet.conf"
[kubeconfig] Wrote KubeConfig file to disk: "/etc/kubernetes/controller-manager.conf"
[apiclient] Created API client, waiting for the control plane to become ready
<-> 这里会停的比较久，要去下载镜像，然后还得启动容器
[apiclient] All control plane components are healthy after 293.004469 seconds
[token] Using token: 2af779.b803df0b1effb3d9
[apiconfig] Created RBAC rules
[addons] Applied essential addon: kube-proxy
[addons] Applied essential addon: kube-dns

Your Kubernetes master has initialized successfully!

To start using your cluster, you need to run (as a regular user):

mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
http://kubernetes.io/docs/admin/addons/

You can now join any number of machines by running the following on each node
as root:

kubeadm join --token 2af779.b803df0b1effb3d9 192.168.25.148:6443
```

监控安装情况命令有：docker ps, docker images, journalctl -xeu kubelet
(/var/log/messages) 。
如果有镜像下载和容器新增，说明安装过程在进行中。否则得检查下你的代理是否正常工作了！

### 配置 kubectl

按照初始化时候的输出说明，以普通用户执行

``` {.bash}
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

安装 flannel

``` {.bash}
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

可选：master 参与运算

默认情况下 master 服务器是不参与运算的，pods 不会在 master
上运行，如果需要 pods 在 master 上运行，可执行一下命令

``` {.bash}
$ kubectl taint nodes --all node-role.kubernetes.io/master-
node "master" untainted
```

安装 dashboard

官方推荐的安装 dashboard 的方法如下

``` {.bash}
kubectl create -f https://raw.githubusercontent.com/kubernetes/dashboard/master/src/deploy/recommended/kubernetes-dashboard.yaml
```

kubernetes-dashboard.yaml 文件中的 ServiceAccount kubernetes-dashboard
只有相对较小的权限，因此我们创建一个 kubernetes-dashboard-admin 的
ServiceAccount 并授予集群 admin 的权限，创建
kubernetes-dashboard-admin.rbac.yaml：

``` {.yaml}
apiVersion: v1
kind: ServiceAccount
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard-admin
  namespace: kube-system

apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: kubernetes-dashboard-admin
  labels:
    k8s-app: kubernetes-dashboard
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: kubernetes-dashboard-admin
  namespace: kube-system
```

``` {.bash}
kubectl create -f kubernetes-dashboard-admin.rbac.yaml
```

查看 kubernete-dashboard-admin 的 token:

``` {.bash}
kubectl -n kube-system get secret | grep kubernetes-dashboard-admin
kubernetes-dashboard-admin-token-pfss5   kubernetes.io/service-account-token   3         14s

 kubectl describe -n kube-system secret/kubernetes-dashboard-admin-token-pfss5
Name:         kubernetes-dashboard-admin-token-pfss5
Namespace:    kube-system
Labels:       <none>
Annotations:  kubernetes.io/service-account.name=kubernetes-dashboard-admin
              kubernetes.io/service-account.uid=1029250a-ad76-11e7-9a1d-08002778b8a1

Type:  kubernetes.io/service-account-token

Data
====
ca.crt:     1025 bytes
namespace:  11 bytes
token:      eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJrdWJlcm5ldGVzLWRhc2hib2FyZC1hZG1pbi10b2tlbi1wZnNzNSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50Lm5hbWUiOiJrdWJlcm5ldGVzLWRhc2hib2FyZC1hZG1pbiIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50LnVpZCI6IjEwMjkyNTBhLWFkNzYtMTFlNy05YTFkLTA4MDAyNzc4YjhhMSIsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDprdWJlLXN5c3RlbTprdWJlcm5ldGVzLWRhc2hib2FyZC1hZG1pbiJ9.Bs6h65aFCFkEKBO_h4muoIK3XdTcfik-pNM351VogBJD_pk5grM1PEWdsCXpR45r8zUOTpGM-h8kDwgOXwy2i8a5RjbUTzD3OQbPJXqa1wBk0ABkmqTuw-3PWMRg_Du8zuFEPdKDFQyWxiYhUi_v638G-R5RdZD_xeJAXmKyPkB3VsqWVegoIVTaNboYkw6cgvMa-4b7IjoN9T1fFlWCTZI8BFXbM8ICOoYMsOIJr3tVFf7d6oVNGYqaCk42QL_2TfB6xMKLYER9XDh753-_FDVE5ENtY5YagD3T_s44o0Ewara4P9C3hYRKdJNLxv7qDbwPl3bVFH3HXbsSxxF3TQ
```

查看安装状态

``` {.bash}
kubectl get pods --all-namespaces
```

安装 Heapster

pods、镜像、容器列表

### pods 列表

``` {.bash}
$ kubectl get pods --all-namespaces -o wide
NAMESPACE     NAME                                    READY     STATUS    RESTARTS   AGE       IP               NODE
kube-system   etcd-master                             1/1       Running   0          23h       192.168.25.148   master
kube-system   kube-apiserver-master                   1/1       Running   0          23h       192.168.25.148   master
kube-system   kube-controller-manager-master          1/1       Running   0          23h       192.168.25.148   master
kube-system   kube-dns-2425271678-t88zz               3/3       Running   0          23h       10.244.0.2       master
kube-system   kube-flannel-ds-1kc48                   2/2       Running   0          23h       192.168.25.148   master
kube-system   kube-proxy-kc2bf                        1/1       Running   0          23h       192.168.25.148   master
kube-system   kube-scheduler-master                   1/1       Running   0          23h       192.168.25.148   master
kube-system   kubernetes-dashboard-3313488171-x9rj7   1/1       Running   0          1h        10.244.2.7       master
```

### 镜像列表

``` {.bash}
$ docker images
REPOSITORY                                               TAG                 IMAGE ID            CREATED             SIZE
gcr.io/google_containers/kube-controller-manager-amd64   v1.7.6              028bd65dc783        2 days ago          138MB
gcr.io/google_containers/kube-scheduler-amd64            v1.7.6              c3101592d24c        2 days ago          77.2MB
gcr.io/google_containers/kube-apiserver-amd64            v1.7.6              f15a956e781d        2 days ago          186MB
gcr.io/google_containers/kube-proxy-amd64                v1.7.6              af674cbf7039        2 days ago          115MB
gcr.io/google_containers/kubernetes-dashboard-amd64      v1.6.3              691a82db1ecd        7 weeks ago         139MB
quay.io/coreos/flannel                                   v0.8.0-amd64        9db3bab8c19e        2 months ago        50.7MB
gcr.io/google_containers/k8s-dns-sidecar-amd64           1.14.4              38bac66034a6        2 months ago        41.8MB
gcr.io/google_containers/k8s-dns-kube-dns-amd64          1.14.4              a8e00546bcf3        2 months ago        49.4MB
gcr.io/google_containers/k8s-dns-dnsmasq-nanny-amd64     1.14.4              f7f45b9cb733        2 months ago        41.4MB
gcr.io/google_containers/etcd-amd64                      3.0.17              243830dae7dd        6 months ago        169MB
gcr.io/google_containers/kube-controller-manager-amd64   v1.5.1              cd5684031720        9 months ago        102MB
gcr.io/google_containers/pause-amd64                     3.0                 99e59f495ffa        16 months ago       747kB
```

### 容器列表

``` {.bash}
$ docker ps
CONTAINER ID        IMAGE                                                    COMMAND                  CREATED             STATUS              PORTS               NAMES
a4d8b0bf22d3        gcr.io/google_containers/pause-amd64:3.0                 "/pause"                 2 hours ago         Up 2 hours                              k8s_POD_webserver-1536865423-8hl0k_default_641b74c8-9adb-11e7-9155-0800277a98f8_0
b68479595f3e        gcr.io/google_containers/k8s-dns-sidecar-amd64           "/sidecar --v=2 --..."   24 hours ago        Up 24 hours                             k8s_sidecar_kube-dns-2425271678-t88zz_kube-system_b024d020-9a25-11e7-9155-0800277a98f8_0
36229b640403        gcr.io/google_containers/k8s-dns-dnsmasq-nanny-amd64     "/dnsmasq-nanny -v..."   24 hours ago        Up 24 hours                             k8s_dnsmasq_kube-dns-2425271678-t88zz_kube-system_b024d020-9a25-11e7-9155-0800277a98f8_0
3477ef11db9b        gcr.io/google_containers/k8s-dns-kube-dns-amd64          "/kube-dns --domai..."   24 hours ago        Up 24 hours                             k8s_kubedns_kube-dns-2425271678-t88zz_kube-system_b024d020-9a25-11e7-9155-0800277a98f8_0
0f12257309a0        gcr.io/google_containers/pause-amd64:3.0                 "/pause"                 24 hours ago        Up 24 hours                             k8s_POD_kube-dns-2425271678-t88zz_kube-system_b024d020-9a25-11e7-9155-0800277a98f8_0
0b25355d63ae        quay.io/coreos/flannel                                   "/opt/bin/flanneld..."   24 hours ago        Up 24 hours                             k8s_kube-flannel_kube-flannel-ds-1kc48_kube-system_000e10bb-9a26-11e7-9155-0800277a98f8_0
86afd9070a0b        quay.io/coreos/flannel                                   "/bin/sh -c 'set -..."   24 hours ago        Up 24 hours                             k8s_install-cni_kube-flannel-ds-1kc48_kube-system_000e10bb-9a26-11e7-9155-0800277a98f8_0
24c225bb39b3        gcr.io/google_containers/pause-amd64:3.0                 "/pause"                 24 hours ago        Up 24 hours                             k8s_POD_kube-flannel-ds-1kc48_kube-system_000e10bb-9a26-11e7-9155-0800277a98f8_0
2891cc49e1c0        gcr.io/google_containers/kube-proxy-amd64                "/usr/local/bin/ku..."   24 hours ago        Up 24 hours                             k8s_kube-proxy_kube-proxy-kc2bf_kube-system_b020a533-9a25-11e7-9155-0800277a98f8_0
cbfdb21bc55d        gcr.io/google_containers/kube-scheduler-amd64            "kube-scheduler --..."   24 hours ago        Up 24 hours                             k8s_kube-scheduler_kube-scheduler-k8s_kube-system_2b08b8f69c973df2fbf99ddef86b56ef_0
65072be8dc4f        gcr.io/google_containers/pause-amd64:3.0                 "/pause"                 24 hours ago        Up 24 hours                             k8s_POD_kube-proxy-kc2bf_kube-system_b020a533-9a25-11e7-9155-0800277a98f8_0
82cd164def30        gcr.io/google_containers/kube-controller-manager-amd64   "kube-controller-m..."   24 hours ago        Up 24 hours                             k8s_kube-controller-manager_kube-controller-manager-k8s_kube-system_c0244259dda50419552580ddd0c049f3_0
d840219ca5bb        gcr.io/google_containers/kube-apiserver-amd64            "kube-apiserver --..."   24 hours ago        Up 24 hours                             k8s_kube-apiserver_kube-apiserver-k8s_kube-system_f3d9d19161b9f95170a67c2834ba1c8b_0
3f0509a245b8        gcr.io/google_containers/etcd-amd64                      "etcd --listen-cli..."   24 hours ago        Up 24 hours                             k8s_etcd_etcd-k8s_kube-system_9ef6d25e21bb4befeabe4d0e4f72d1ca_0
e3858cbeff18        gcr.io/google_containers/pause-amd64:3.0                 "/pause"                 24 hours ago        Up 24 hours                             k8s_POD_kube-scheduler-k8s_kube-system_2b08b8f69c973df2fbf99ddef86b56ef_0
ddd174443dc7        gcr.io/google_containers/pause-amd64:3.0                 "/pause"                 24 hours ago        Up 24 hours                             k8s_POD_kube-controller-manager-k8s_kube-system_c0244259dda50419552580ddd0c049f3_0
430e89f47f1d        gcr.io/google_containers/pause-amd64:3.0                 "/pause"                 24 hours ago        Up 24 hours                             k8s_POD_kube-apiserver-k8s_kube-system_f3d9d19161b9f95170a67c2834ba1c8b_0
b3134e793b2c        gcr.io/google_containers/pause-amd64:3.0                 "/pause"                 24 hours ago        Up 24 hours                             k8s_POD_etcd-k8s_kube-system_9ef6d25e21bb4befeabe4d0e4f72d1ca_0
```

node
====

加入集群

使用初始化集群时提示的命令加入集群

``` {.bash}
kubeadm join --token 2af779.b803df0b1effb3d9 192.168.25.148:6443
```
