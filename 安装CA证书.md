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
[root@k8s-master ~]# mv cfssl-certinfo_linux-amd64 /usr/bin/cfssl-certinfo

```
