# 1.环境准备 3台 centos 服务器
master节点地址：192.168.0.211
node1节点地址：192.168.0.212
node2节点地址：192.168.0.213
# 2.其他准备工作
- 2.1三个节点都安装 Docker version 18.09.6
- 2.2关闭 selinux：
修改/etc/selinux/config文件中的SELINUX= xxx
```
sed -i 's/^SELINUX=.*/SELINUX=disabled/' /etc/selinux/config
```
不重启系统生效，使用命令:
```
setenforce 0
```
关闭防火墙：
```
systemctl stop firewalled
systemctl disable firewalled
```
# 3.设置 k8s01-master 实现无密码登录所有node
```
ssh-keygen -t rsa
ssh-copy-id root@k8s-master
ssh-copy-id root@k8s-node01
ssh-copy-id root@k8s-node02
```
