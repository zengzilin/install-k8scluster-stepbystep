# 在三个节点安装etcd
```
yum install etcd -y
```
# 配置etcd.conf(有省略，节点ip为服务器具体ip)
```
[root@k8s-master ssl]# cat /etc/etcd/etcd.conf
#[Member]
#ETCD_CORS=""
ETCD_DATA_DIR="/var/lib/etcd/default.etcd"
#ETCD_WAL_DIR=""
ETCD_LISTEN_PEER_URLS="http://192.168.0.211:2380"
ETCD_LISTEN_CLIENT_URLS="http://192.168.0.211:2379"
...
#[Clustering]
ETCD_INITIAL_ADVERTISE_PEER_URLS="http://192.168.0.211:2380,http://127.0.0.1:2379"
ETCD_ADVERTISE_CLIENT_URLS="http://192.168.0.211:2379"
#ETCD_DISCOVERY=""
#ETCD_DISCOVERY_FALLBACK="proxy"
#ETCD_DISCOVERY_PROXY=""
#ETCD_DISCOVERY_SRV=""
ETCD_INITIAL_CLUSTER="etcd01=http://192.168.0.211:2380,etcd02=http://192.168.0.212:2380,etcd03=http://193.168.0.213:2380"
ETCD_INITIAL_CLUSTER_TOKEN="etcd-cluster"
ETCD_INITIAL_CLUSTER_STATE="new"
...
```
```
[root@k8s-node01 system]# cat /etc/etcd/etcd.conf
#[Member]
#ETCD_CORS=""
ETCD_DATA_DIR="/var/lib/etcd/default.etcd"
#ETCD_WAL_DIR=""
ETCD_LISTEN_PEER_URLS="http://192.168.0.212:2380"
ETCD_LISTEN_CLIENT_URLS="http://192.168.0.212:2379"
#ETCD_MAX_SNAPSHOTS="5"
#ETCD_MAX_WALS="5"
ETCD_NAME="etcd02"
...
#
#[Clustering]
ETCD_INITIAL_ADVERTISE_PEER_URLS="http://192.168.0.212:2380"
ETCD_ADVERTISE_CLIENT_URLS="http://192.168.0.212:2379"
#ETCD_DISCOVERY=""
#ETCD_DISCOVERY_FALLBACK="proxy"
#ETCD_DISCOVERY_PROXY=""
#ETCD_DISCOVERY_SRV=""
ETCD_INITIAL_CLUSTER="etcd01=http://192.168.0.211:2380,etcd02=http://192.168.0.212:2380,etcd03=http://193.168.0.213:2380"
ETCD_INITIAL_CLUSTER_TOKEN="etcd-cluster"
ETCD_INITIAL_CLUSTER_STATE="new"
#ETCD_STRICT_RECONFIG_CHECK="true"
#ETCD_ENABLE_V2="true"
#
...
```
```
[root@k8s-node02 system]# cat /etc/etcd/etcd.conf 
#[Member]
#ETCD_CORS=""
ETCD_DATA_DIR="/var/lib/etcd/default.etcd"
#ETCD_WAL_DIR=""
ETCD_LISTEN_PEER_URLS="http://192.168.0.213:2380"
ETCD_LISTEN_CLIENT_URLS="http://192.168.0.213:2379"
#ETCD_MAX_SNAPSHOTS="5"
#ETCD_MAX_WALS="5"
ETCD_NAME="etcd03"
...
#
#[Clustering]
ETCD_INITIAL_ADVERTISE_PEER_URLS="http://192.168.0.213:2380"
ETCD_ADVERTISE_CLIENT_URLS="http://192.168.0.211:2379"
#ETCD_DISCOVERY=""
#ETCD_DISCOVERY_FALLBACK="proxy"
#ETCD_DISCOVERY_PROXY=""
#ETCD_DISCOVERY_SRV=""
ETCD_INITIAL_CLUSTER="etcd01=http://192.168.0.211:2380,etcd02=http://192.168.0.212:2380,etcd03=http://193.168.0.213:2380"
#ETCD_INITIAL_CLUSTER_TOKEN="etcd-cluster"
#ETCD_INITIAL_CLUSTER_STATE="new"
#ETCD_STRICT_RECONFIG_CHECK="true"
#ETCD_ENABLE_V2="true"
...
```
# 配置etcd 启动文件（添加证书配置）(三台节点的配置都一样)

```
[root@k8s-node01 system]# cat etcd.service
[Unit]
Description=Etcd Server
After=network.target
After=network-online.target
Wants=network-online.target

[Service]
Type=notify
WorkingDirectory=/var/lib/etcd/
EnvironmentFile=-/etc/etcd/etcd.conf
User=etcd
# set GOMAXPROCS to number of processors
ExecStart=/bin/bash -c "GOMAXPROCS=$(nproc) /usr/bin/etcd --name=\"${ETCD_NAME}\" --data-dir=\"${ETCD_DATA_DIR}\" --listen-client-urls=\"${ETCD_LISTEN_CLIENT_URLS}\""
--cert-file=/opt/kubernetes/ssl/server.pem \
--key-file=/opt/kubernetes/ssl/server-key.pem \
--peer-cert-file=/opt/kubernetes/ssl/server.pem \
--peer-key-file=/opt/kubernetes/ssl/server-key.pem \
--trusted-ca-file=/opt/kubernetes/ssl/ca.pem \
--peer-trusted-ca-file=/opt/kubernetes/ssl/ca.pem \
Restart=on-failure
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
```
# 开启服务
```
systemctl start etcd
systemctl enable etcd
```
# 检查节点的健康状态
```
/usr/bin/etcdctl --ca-file=ca.pem --cert-file=server.pem --key-file=server-key.pem --endpoints="http://192.168.0.211:2379,http://192.168.0.212:2379,http://192.168.0.213:2379" cluster-health
```
```
[root@k8s-master ssl]# /usr/bin/etcdctl --ca-file=ca.pem --cert-file=server.pem --key-file=server-key.pem \
> --endpoints="http://192.168.0.211:2379,http://192.168.0.212:2379,http://192.168.0.213:2379" \
> cluster-health
member 8e9e05c52164694d is healthy: got healthy result from http://192.168.0.211:2379
cluster is healthy

```
