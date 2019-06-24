# 安装cfssl工具
```
wget https://pkg.cfssl.org/R1.2/cfssl_linux-amd64
wget https://pkg.cfssl.org/R1.2/cfssljson_linux-amd64
wget https://pkg.cfssl.org/R1.2/cfssl-certinfo_linux-amd64
```
```
[root@k8s-master ~]# chmod +x *-amd64
[root@k8s-master ~]# ls
anaconda-ks.cfg  cfssl-certinfo_linux-amd64  cfssljson_linux-amd64  cfssl_linux-amd64
[root@k8s-master ~]# mv cfssl_linux-amd64 /usr/local/bin/cfssl
[root@k8s-master ~]# mv cfssljson_linux-amd64 /usr/local/bin/cfssljson
[root@k8s-master ~]# mv cfssl-certinfo_linux-amd64 /usr/local/bin/cfssl-certinfo
```
# 创建证书存放目录
```
[root@k8s-master ssl]# pwd
/root/ssl
```
# 创建配置文件
CA 配置文件用于配置根证书的使用场景 (profile) 和具体参数 (usage，过期时间、服务端认证、客户端认证、加密等)，后续在签名其它证书时需要指定特定场景。
```

```
```
[root@k8s-master ssl]# cfssl print-defaults config > config.example

cat > ca-config.json <<EOF
{
  "signing": {
    "default": {
      "expiry": "87600h"
    },
    "profiles": {
      "kubernetes": {
        "usages": [
            "signing",
            "key encipherment",
            "server auth",
            "client auth"
        ],
        "expiry": "87600h"
      }
    }
  }
}
EOF
```
# 创建证书签名请求文件
```
root@k8s-master ssl]# cfssl print-defaults csr > csr.example #参考例子
cat > ca-csr.json <<EOF
{
  "CN": "kubernetes",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "CN",
      "ST": "BeiJing",
      "L": "BeiJing",
      "O": "k8s",
      "OU": "4Paradigm"
    }
  ]
}
EOF
```
# 生成CA证书和私钥
```
[root@k8s-master ssl]# cfssl gencert -initca ca-csr.json | cfssljson -bare ca
```
# 生成server证书(先生成CA证书和私钥再执行此步骤)
```
[root@k8s-master ssl]# vim  
cat > server-csr.json <<EOF
    "hosts": [
        "127.0.0.1",
        "192.168.0.211",
        "192.168.0.212",
        "192.168.0.213",
        "10.10.10.1",
        "kubernetes",
        "kubernetes.default",
        "kubernetes.default.svc",
        "kubernetes.default.svc.cluster",
        "kubernetes.default.svc.cluster.local"
    ],
    "key": {
        "algo": "rsa",
        "size": 2048
    },
    "names": [
        {
            "C": "CN",
            "L": "Beijing",
            "ST": "Bejing",
            "O": "k8s",
            "OU": "System"
        }
    ]
}
EOF
```
```
cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=kubernetes server-csr.json | cfssljson -bare server
```
# 生成admin证书
```
cat > ca-csr.json <<EOF
{
  "CN": "admin",
  "host": [],
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "CN",
      "ST": "BeiJing",
      "L": "BeiJing",
      "O": "system:masters",
      "OU": "System"
    }
  ]
}
EOF
```
```
[root@k8s-master ssl]# cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=kubernetes admin-csr.json | cfssljson -bare admin
```
# 生成kube-proxy证书
```
cat > kube-proxy-csr.json <<EOF
{
  "CN": "system:kube-proxy",
  "host": [],
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "CN",
      "ST": "BeiJing",
      "L": "BeiJing",
      "O": "k8s",
      "OU": "System"
    }
  ]
}
EOF
```
```
[root@k8s-master ssl]#  cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=kubernetes kube-proxy-csr.json | cfssljson -bare kube-proxy
```
