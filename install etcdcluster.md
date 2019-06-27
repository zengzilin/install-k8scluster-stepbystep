# 在三个节点安装etcd
```
yum install etcd -y
```
# 配置etcd.conf
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
