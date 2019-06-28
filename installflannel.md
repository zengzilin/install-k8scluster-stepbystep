# 下载二进制包
```
wget https://github.com/coreos/flannel/releases/download/v0.11.0/flannel-v0.11.0-linux-amd64.tar.gz
```
# 编辑flanneld配置文件
```
[root@k8s-master cfg]# cat flanneld 
# Flanneld configuration options  

# etcd url location.  Point this to the server where etcd runs
# FLANNEL_ETCD_ENDPOINTS="http://127.0.0.1:2379"

# etcd config key.  This is the configuration key that flannel queries
# For address range assignment
FLANNEL_ETCD_PREFIX="/atomic.io/network"

# Any additional options that you want to pass
FLANNEL_OPTIONS="--etcd-endpoints=http://192.168.0.211:2379,http://192.168.0.212:2379,http://192.168.0.213:2379 -etcd-cafile=/opt/kubernetes/ssl/ca.pem -etcd-certfile=/opt/kubernetes/ssl/server.pem -etcd-keyfile=/opt/kubernetes/ssl/server-key.pem"
[root@k8s-master cfg]# 
```
# 配置 flanneld.service启动文件
```
cd /lib/systemd/system
[root@k8s-master system]# cat flanneld.service 
[Unit]
Description=Flanneld overlay address etcd agent
After=network.target
After=network-online.target
Wants=network-online.target
After=etcd.service
Before=docker.service

[Service]
Type=notify
EnvironmentFile=/opt/kubernetes/cfg/flanneld
# EnvironmentFile=-/etc/sysconfig/docker-network
ExecStart=/opt/kubernetes/bin/flanneld --ip-masq $FLANNEL_OPTIONS
ExecStartPost=/opt/kubernetes/bin/mk-docker-opts.sh -k DOCKER_NETWORK_OPTIONS -d /run/flannel/subnet.env
Restart=on-failure

[Install]
WantedBy=multi-user.target
# WantedBy=docker.service

```
# 重启systemd.service内核（否则 flanneld.service启动会报错）
（Failed to start flanneld.service: Unit not found.）
```
[root@k8s-master system]# systemctl daemon-reload
```
# 配置flannel子网
```
[root@k8s-master ssl]# /usr/bin/etcdctl --ca-file=ca.pem --cert-file=server.pem --key-file=server-key.pem --endpoints="http://192.168.0.211:2379,http://192.168.0.212:2379,http://192.168.0.213:2379" set /coreos/network/config '{ "Network": "172.17.0.0/16", "Backend":{"Type": "vxlan"}}'
```
