
[引自CSDN kubernetes 安装](http://blog.csdn.net/swcj/article/details/54969234)
1 安装前一定要关闭firewalld服务，否则需要配firewalld的规则

2 docker启动之后会产生一个网卡，这个网卡的地址默认是172.17.0.0 网段，这是个虚拟网络，如果要是想让集群中的其他机器访问容器需要通过安装flannel来实现。所以安装完flannel和docker一定要确定一下网段和etcd中储存的是否一致，如果不一致重启docker服务
确保你的
docker0: flags=4099<UP,BROADCAST,MULTICAST> mtu 1500

inet 192.168.41.1 netmask 255.255.255.0 broadcast 0.0.0.0

ether 02:42:91:89:15:19 txqueuelen 0 (Ethernet)
和你安装完etcd设置的数据一致
etcdctl mk /coreos.com/network/config
'{"Network": "192.168.0.0/16"}'

3 请不用用容器安装linux然后再容器中安装docker这样会会有很多问题用虚拟机可以，反正我是没有成功在docker中安装linux再安装kubernetes。

4最少要用mini版本进行安装kubernetes需要依赖很多系统服务，需要centos7以上版本最好7.2以上

5 记得在拉取镜像的时候一定要加上版本号，一开始不知道在阿里的镜像库上总是拉不到镜像

6 KUBE_CONTROLLER_MANAGER_ARGS配置中默认是需要认证，的记得去掉SecurityContextDeny,ServiceAccount。

7安装中出了问题要看日志journalctl -n 10 ,运行中的日志查看/var/log/messages，docker的日志使用docker logs -f 容器id

8kubernetes-dashboard安装 其实不需要翻墙只有修改ymal文件中的镜像路径就可以了，阿里有很多的镜像

master 需要安装 kubernetes-master etcd flannel docker
node需要安装 kubernetes-node flannel docker etcd

查看防火墙状态firewalld
systemctl status firewalld.service
关闭防火墙
systemctl stop firewalld.service
systemctl disable firewalld.service

安全设置
setenforce 0 

查看iptables
systemctl status iptables.service
systemctl stop iptables.service

设置主机名
hostnamectl –static set-hostname centos-master

关闭Selinux
/etc/selinux/config
SELINUX=disabled

安装master
yum install docker
yum install etcd
yum install flannel
yum install kubernetes-master

配置 etcd
所有的etcd都统一配置
/etc/etcd/etcd.conf
ETCD_NAME=default



ETCD_DATA_DIR="/var/lib/etcd/default.etcd"



ETCD_LISTEN_CLIENT_URLS="http://0.0.0.0:2379"



ETCD_ADVERTISE_CLIENT_URLS="http://192.168.163.148:2379" masterIP端口



运行

systemctl enable etcd
systemctl start etcd

etcdctl mk /coreos.com/network/config
'{"Network": "192.168.0.0/16"}'
etcdctl get /coreos.com/network/config

node节点需要运行etcdctl ls /coreos.com查看是否有返回值



配置apiserver

/etc/kubernetes/apiserver



KUBE_ETCD_SERVERS="--etcd_servers=http://192.168.163.148:2379"



KUBE_SERVICE_ADDRESSES="--service-cluster-ip-range=192.168.0.0/16"



配置controller-manager

/etc/kubernetes/controller-manager



KUBE_CONTROLLER_MANAGER_ARGS="--node-monitor-grace-period=10s --pod-eviction-timeout=10s"



配置config

/etc/kubernetes/config

KUBE_MASTER="--master=http://192.168.163.148:8080"



启动

systemctl enable 

systemctl start kube-apiserver

systemctl start kube-scheduler

systemctl start kube-controller-manager



因为没有关闭防火墙导致 kube-apiserve无法启动通过 systemctl status kube-apiserver.service查看 kube-apiserver状态和journalctl -xe查看日志







安装客户端kubernetes-node



yum install kubernetes-node







安装docker

yum install docker

systemctl start docker



安装flannel

yum install flannel





配置flanneld

/etc/sysconfig/flanneld

添加

FLANNEL_ETCD_ENDPOINTS="http://127.0.0.1:2379"

FLANNEL_ETCD="http://127.0.0.1:2379"
FLANNEL_ETCD_KEY="/coreos.com/network"





安装配置kubernetes node

yum install kubernetes node
/etc/kubernetes/kublet

node节点ip
KUBELET_HOSTNAME="--hostname_override=192.168.163.150"

server节点ip
KUBELET_API_SERVER="--api_servers=http://192.168.163.148:8080"

配置
KUBE_PROXY_OPTS="\



--master=http://127.0.0.1:8080 \



--proxy-mode=iptables \



--logtostderr=true"

启动
systemctl start flanneld.service
systemctl start kubeletsystemctl start kube-proxy


安装kubernetes-dashboard

首先要保证网络正确
mast要能ping通node上容器的ip，node上的容器能够访问apiserver-host。
查看master是否能获取节点kubectl  get node

通过
https://rawgit.com/kubernetes/dashboard/master/src/deploy/kubernetes-dashboard.yaml连接获取 kubernetes-dashboard.yaml文件 修改，

将google的替换本地的镜像
image: docker.io/ist0ne/kubernetes-dashboard-amd64
不要去google拉取镜像
imagePullPolicy: IfNotPresent
配置apiserver的ip和端口
- --apiserver-host=http://192.168.137.147:8080
拉取
registry.access.redhat.com/rhel7/pod-infrastructure




修改完成后上传至master
运行 kubectl create -f  kubernetes-dashboard.yaml

Error from server: error when creating "nginx.yaml": Pod "nginx" is forbidden:
no API token found for service account default/default, retry after the token is automatically created and added to the service account解决办法是编辑/etc/kubernetes/apiserver
 去除 KUBE_ADMISSION_CONTROL中的SecurityContextDeny,ServiceAccount，并重启kube-apiserver.service服务：#vim /etc/kubernetes/apiserverKUBE_ADMISSION_CONTROL="--admission_control=NamespaceLifecycle,NamespaceExists,LimitRanger,ResourceQuota"