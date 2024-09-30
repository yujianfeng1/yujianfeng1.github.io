### 实验环境
MacBook Pro 16-inch, 2019

处理器: 2.4 GHz 8-Core Intel Core i9

内存: 32 GB 2667 MHz DDR4

macos:  Sonoma version 14.2.1 (23C71)

使用UTM虚拟机工具创建3台ubuntu20.04-arm64，虚拟机配置均为2核4G, 采用1个master， 2个node的构架

###  部署架构规划
| 角色 | 主机名 | 组件 | IP |
|-------|-------|-------|-------|
| master | master | etcd、apiserver、controller-manager、scheduler、kubelet、proxy、calico、runc |192.168.64.18
|node01|node01|pod、kubelet、proxy、calico、runc|192.168.64.19
|node01|node01|pod、kubelet、proxy、calico、runc|192.168.64.20

### 软件版本
*docker server ：24.0.7

*containerd ：1.7.12

*kubeadm : v1.30.0

*kubelet : v1.30.0

*kubectl : v1.30.0

### 集群服务初始化 （3*， 所有机器都执行）
#### 切换为管理员
```shell
sudo su
```
#### 添加主机名
```shell
cat >> /etc/hosts <<EOF
192.168.64.18 master
192.168.64.19 node01
192.168.64.20 node02
EOF

```
#### 时间同步
```shell
apt update && apt install ntpdate
ntpdate cn.ntp.org.cn
```
#### 关闭防火墙
```shell
systemctl disable ufw && systemctl stop ufw
```
#### 关闭swap分区
```shell
swapoff -a  # 临时关闭
sed -ri 's/.swap*./#&/g' /etc/fstab   # 修改配置文件 sed修改配置文件永久关闭
```
#### 启用iptables桥接流量
```shell
# 添加配置文件
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

# 加载模块
modprobe overlay
modprobe br_netfilter

# 设置所需的Sysctl参数，参数在重新启动时保持不变
cat  <<EOF | tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward = 1
vm.swappiness = 0
EOF

# 不重启就应用sysctl参数
sysctl --system

# 保守建议
reboot
```
#### 安装容器运行时
k8s支持的容器运行时有很多如docker、containerd、cri-o等等，docker 省事一些，这里选择安装docker
```shell
# 安装docker
apt-get update
apt install -y docker.io

# 设置docker开机自启
systemctl start docker
ststemctl enable docker

# 三个服务都应是running状态
systemctl status containerd.service
systemctl status docker.service
systemctl status docker.socket
```

#### 配置容器运行时的systemd驱动
```shell

vim /etc/docker/daemon.json 

```
然后将下面设置加入daemon.js
```shell
{
  "exec-opts": [
    "native.cgroupdriver=systemd"
  ]
}

```
重启docker
```shell 
systemctl restart docker
```
#### 设置Kubernetes APT存储库的GPG密钥
```shell 
KUBERNETES_VERSION=1.30

sudo mkdir -p /etc/apt/keyrings

curl -fsSL https://pkgs.k8s.io/core:/stable:/v$KUBERNETES_VERSION/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v$KUBERNETES_VERSION/deb/ /" | sudo tee /etc/apt/sources.list.d/kubernetes.list


```
 更新 apt 仓库
 ```shell
 apt-get update -y
 ```
 #### 安装Kubernetes组件
 ```shell 
 
 # 安装一些辅助包
apt-get install -y apt-transport-https ca-certificates curl gpg
 
# 安装指定版本
apt-get install -y kubelet=1.30.0-1.1 kubectl=1.30.0-1.1 kubeadm=1.30.0-1.1

# 确认服务状态（此时kubelet暂未启动）
systemctl status kubelet
 ```
 ### Kubernetes 初始化（只在master操作）
 ```shell
kubeadm init \
  --control-plane-endpoint="192.168.64.18" \
  --service-cidr=10.96.0.0/12 \
  --pod-network-cidr=10.244.0.0/16 \
  --ignore-preflight-errors=all
  
# –-apiserver-advertise-address # 集群通告地址，单master时为控制面使用的的服务器IP
# –-service-cidr #集群内部虚拟网络，Pod统一访问入口，可以不用更改，直接用上面的参数
# –-pod-network-cidr #Pod网络，与下面部署的CNI网络组件yaml中保持一致，可以不用更改，直接用上面的参数

 ```
 初始化成功显示如下
 ```shell
 Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

You can now join any number of the control-plane node running the following command on each as root:

  kubeadm join 192.168.64.18:6443 --token m5pstt.v2gebrasw9yfzxb6 \
	--discovery-token-ca-cert-hash sha256:205a991b66a2ec879bd36f66851725fd84201d7306bb088ceda8835ad99450b8 \
	--control-plane --certificate-key 34a274a91255cf25cda3af0f6ecbfc2876e97196491338aab72ca1b3bd09e1d1

Please note that the certificate-key gives access to cluster sensitive data, keep it secret!
As a safeguard, uploaded-certs will be deleted in two hours; If necessary, you can use
"kubeadm init phase upload-certs --upload-certs" to reload certs afterward.

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 192.168.64.18:6443 --token m5pstt.v2gebrasw9yfzxb6 \
	--discovery-token-ca-cert-hash sha256:205a991b66a2ec879bd36f66851725fd84201d7306bb088ceda8835ad99450b8 
 ```
 从输出中使用以下命令在master中创建kubeconfig，以便您可以使用kubectl与集群API交互。
 ```shell
 mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
kubectl命令来验证kubeconfig，以列出kube-system命名空间中的所有pod。
```shell
root@master:/home/master# kubectl get pod -n kube-system
NAME                                       READY   STATUS    RESTARTS       AGE
coredns-7db6d8ff4d-552p6                   1/1     Running   0              4h59m
coredns-7db6d8ff4d-hk5cz                   1/1     Running   0              4h59m
etcd-master                                1/1     Running   1 (141m ago)   4h59m
kube-apiserver-master                      1/1     Running   1 (141m ago)   4h59m
kube-controller-manager-master             1/1     Running   2 (141m ago)   4h59m
kube-proxy-clz8d                           1/1     Running   0              4h50m
kube-proxy-fnf7q                           1/1     Running   1 (126m ago)   4h50m
kube-proxy-t2rsp                           1/1     Running   1 (141m ago)   4h59m
kube-scheduler-master                      1/1     Running   2 (141m ago)   4h59m
metrics-server-6455f4d6f7-jf2x9            1/1     Running   0              125m

```

### 添加node节点
```shell
  kubeadm join 192.168.64.18:6443 --token m5pstt.v2gebrasw9yfzxb6 \
	--discovery-token-ca-cert-hash sha256:205a991b66a2ec879bd36f66851725fd84201d7306bb088ceda8835ad99450b8 \
	--control-plane --certificate-key 34a274a91255cf25cda3af0f6ecbfc2876e97196491338aab72ca1b3bd09e1d1
```

### 安装集群网络
#### 使用Calico网络插件进行设置
```shell
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml

```
几分钟后，如果您查看kube-system命名空间中的pods，您将看到calico pods和正在运行的CoreDNS pods。
```shell
kubectl get po -n kube-system

```
显示如下 calico-xxx

```shell 
root@master:/home/master# kubectl get po -n kube-system
NAME                                       READY   STATUS    RESTARTS       AGE
calico-kube-controllers-5b9b456c66-d46vb   1/1     Running   0              4h50m
calico-node-bnwpm                          1/1     Running   1 (133m ago)   4h50m
calico-node-q4wf8                          1/1     Running   0              4h50m
calico-node-rhj2w                          1/1     Running   0              4h50m
coredns-7db6d8ff4d-552p6                   1/1     Running   0              5h6m
coredns-7db6d8ff4d-hk5cz                   1/1     Running   0              5h6m
etcd-master                                1/1     Running   1 (148m ago)   5h7m
kube-apiserver-master                      1/1     Running   1 (148m ago)   5h7m
kube-controller-manager-master             1/1     Running   2 (148m ago)   5h7m
kube-proxy-clz8d                           1/1     Running   0              4h58m
kube-proxy-fnf7q                           1/1     Running   1 (133m ago)   4h58m
kube-proxy-t2rsp                           1/1     Running   1 (148m ago)   5h6m
kube-scheduler-master                      1/1     Running   2 (148m ago)   5h7m
metrics-server-6455f4d6f7-jf2x9            1/1     Running   0              133m

```

### 部署一个示例Nginx应用程序
现在我们有了使集群和应用程序工作的所有组件，让我们部署一个示例Nginx应用程序，看看我们是否可以通过NodePort访问它 
 
创建一个Nginx部署。直接在命令行中执行以下命令。它将pod部署在默认命名空间中。

```shell 
cat <<EOF | kubectl apply -f -
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 2 
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80      
EOF
```

在NodePort 32000上公开Nginx部署
```shell
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector: 
    app: nginx
  type: NodePort  
  ports:
    - port: 80
      targetPort: 80
      nodePort: 32000
EOF
```
检查Pod 状态
```shell 
kubectl get pods
```
结果显示如下
```shell
root@master:/home/master# kubectl get pods
NAME                               READY   STATUS    RESTARTS   AGE
nginx-deployment-576c6b7b6-5vgz9   1/1     Running   0          4h54m
nginx-deployment-576c6b7b6-965br   1/1     Running   0          4h54m
```
部署完成后，你应该能够访问分配的NodePort上的Nginx主页。
```shell
root@master:/home/master# kubectl get services
NAME            TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
kubernetes      ClusterIP   10.96.0.1       <none>        443/TCP        5h11m
nginx-service   NodePort    10.105.26.165   <none>        80:32000/TCP   4h53m

```
参考资料：
- [How To Setup Kubernetes Cluster Using Kubeadm](https://devopscube.com/setup-kubernetes-cluster-kubeadm/)

- [在ubuntu22.04-arm64系统上基于kubeadm部署kubernetes测试集群](https://juejin.cn/post/7416903220591460390)
