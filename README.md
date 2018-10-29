# 欢迎使用 kubernetes ansible一键部署集群工具

<img src="https://avatars1.githubusercontent.com/u/1507452?s=200&v=4" width="100"><img src="https://github.com/kubernetes/kubernetes/raw/master/logo/logo.png" width="100">

------

  该版本可直接用于生产环境的部署,我们现在的生产环境均是采用这套配置.所有的安装直接基于官方的二进制文件,采用裸金属的方式部署(非容器),详细配置如下：

- [x] 3个master节点,N个node节点,Etcd节点3个,部署在和master节点同一宿主机
- [x] 配置推荐:master和node节点 cpu,操作系统推荐centos7任意版本,ansible版本2.7.0(默认yum安装) 至少1核,内存大于2GB.至少4个主机,一个ansible控制机器,3个master节点
- [x] 支持一键安装,同样也可以拆分部署
- [x] ansible控制机器必须关闭selinux,使用root用户登录执行脚本.
- [x] 被控制节点只需配置好IP即可,其他无需任何特殊配置
- [x] kubernetes安装默认已经开启证书认证

---
## 整体架构图:
![cmd-markdown-logo](https://github.com/feiyu563/ansible-kubernetes/blob/master/images/jgt.png)

主机配置：
```
[local] #ansible控制机器配置
127.0.0.1 name=local ansible_ssh_user=root ansible_ssh_pass=123456 
#local server 192.168.10.10

[master] #kubernetes master节点配置
192.168.10.11 name=master1 ansible_ssh_user=root ansible_ssh_pass=123456
192.168.10.12 name=master2 ansible_ssh_user=root ansible_ssh_pass=123456
192.168.10.13 name=master3 ansible_ssh_user=root ansible_ssh_pass=123456

[node] #kubernetes node节点配置
192.168.10.14 name=node1 ansible_ssh_user=root ansible_ssh_pass=123456

[all:children] #kubernetes主机组配置
master
node

[all:vars] #所需的环境变量
#hub=docker.io
bootstraptoken=e9c768c6bfffc6a0c193eebe4685befa
kubernetesversion=v1.11.3 #kubernetes版本,会自动去下载,或者自己提前下载好更佳
updatekernel=true #是否更新系统内核,推荐更新
```
### 目录解析

```
addons/ #calico部署
approve/ #自动接入node节点
cni/ #网络插件部署
docker/ #部署docker
downfiles/ 自动下载kubernetes二进制文件
etcd-ssl/ #生成etcd证书
init/ #初始化部署
install-addons.yaml
installall.yaml #一键自动安装
install-master.yaml
install-node.yaml
install-ssl.yaml
kubernetes-master/ #部署kubernetes master节点
kubernetes-node/ #部署node节点
kubernetes-ssl/ #生成kubernetes证书
update-kernel/ #升级系统内核

```

------

## 部署流程

准备好ansible的控制机器(centos7系统即可)和被控机器(centos7即可),确保各机器之间内网可以互通:

### 1. ansible控制机器执行以下操作:

- [x] 关闭selinux,并重启系统:
```
sed -i "s/^SELINUX=enforcing/SELINUX=disabled/g" /etc/sysconfig/selinux && setenforce 0 && sed -i "s/^SELINUX=enforcing/SELINUX=disabled/g" /etc/selinux/config   && sed -i "s/^SELINUX=permissive/SELINUX=disabled/g" /etc/sysconfig/selinux  && sed -i "s/^SELINUX=permissive/SELINUX=disabled/g" /etc/selinux/config   && setenforce  0 && reboot
```
- [x] 安装ansible:
```
yum install -y epel-release
yum install -y sshpass ansible
```
- [x] 下载我的所有配置文件到ansible的配置目录 /etc/ansible下,替换原文件
- [x] 执行一键部署kubernetes集群
```
ansible-playbook /etc/ansible/roles/installall.yaml
```
- [x] 等待执行完成后查看node节点是否都一键上线
```
kubectl get nodes
```
