# 设置环境变量
将可执行文件目录添加到 PATH 环境变量中：
下载kubernetes客户端
```
wget https://dl.k8s.io/v1.14.2/kubernetes-client-linux-amd64.tar.gz
tar -xzvf kubernetes-client-linux-amd64.tar.gz
```
```
echo 'PATH=/opt/kubernetes/bin:$PATH' >>/root/.bashrc
source /root/.bashrc
```
# 生成kubelet bootstrapping kubeconfig
## 生成token
```
export BOOTSTRAP_TOKEN=$(head -c 16 /dev/urandom | od -An -t x | tr -d ' ')
[root@k8s-master ssl]# cat > token.csv << EOF
>${BOOTSTRAP_TOKEN},kubelet-bootstrap,10001, "system:kubelet-bootstrap"
>EOF
```
## 设置集群参数
```
[root@k8s-master ssl]# export KUBE_APISERVER="https://192.168.0.211:6443"
[root@k8s-master ssl]# kubectl config set-cluster kubernetes --certificate-authority=./ca.pem --embed-certs=true --server=${KUBE_APISERVER} --kubeconfig=bootstrap.kubeconfig
```
## 设置客户端认证参数
```
[root@k8s-master ssl]# kubectl config set-credentials kubelet-bootstrap \
> --token=${BOOTSTRAP_TOKEN} \
> --kubeconfig=bootstrap.kubeconfig 
```
## 设置上下文参数
```
[root@k8s-master ssl]# kubectl config set-context default \
> --cluster=kubernetes \
> --user=kubernetes \
> --kubeconfig=bootstrap.kubeconfig 
```
## 设置默认上下文
```
[root@k8s-master ssl]# kubectl config use-context default --kubeconfig=bootstrap.kubeconfig
```
# 创建kube-proxy kubeconfig
```
[root@k8s-master ssl]# kubectl config set-cluster kubernetes 
> --certificate-authority=./ca.pem \
> --embed-certs=true \
> --server=${KUBE_APISERVER} \
> --kubeconfig=kube-proxy.kubeconfig
```
```
```
